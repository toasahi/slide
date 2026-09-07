# AWS DevOps Agent × Mastra A2A 連携 実装計画

北極星: **誰でもインシデントを解決できる状態**。調査は DevOps Agent に任せ、Mastra 側は「人の言葉 → 調査依頼 → 結果を行動に変換」を担う。

---

## 0. 裁定（設計判断のまとめ）

| 論点 | 裁定 | 理由 |
|---|---|---|
| トランスポート | `@a2a-js/sdk` の **REST（HTTP+JSON）トランスポートのみ** を使う。Mastra 組み込みの `A2AAgent` は使わない | DevOps Agent の Agent Card は `protocolBinding: "HTTP+JSON"` 一択。Mastra 側は JSON-RPC 前提のため直結不可 |
| Mastra への組み込み方 | **`createTool` で 3 つのツール + 長時間調査用の Workflow** として組み込む。サブエージェント化はしない | スキル選択不可・Message/Task の二態・5〜8 分の非同期を Mastra の抽象に押し込むより、ツール境界で吸収した方が制御しやすい |
| Message / Task の吸収 | クライアントが `DevOpsResult` という **統一型に正規化** して返す。Mastra 側は A2A の型を一切見ない | DevOps Agent が chat（Message 即答）か investigate（Task 非同期）かを決めるため、呼び出し側は両方を受けられる必要がある |
| 長時間 Task の待ち方 | MVP は **GetTask ポーリング**（15〜30 秒間隔、上限 12 分）。`SubscribeToTask`(SSE) は Phase 4 で追加 | pushNotifications: false のため Webhook は使えない。SSE の 5〜8 分保持は再接続処理が必要で MVP には重い |
| 認証 | MVP は **Bearer トークン**（scope: `operate`、client type: `agent`）。SigV4 は Phase 4 | 最小構成で疎通を優先。SigV4 はリクエスト署名インターセプタで後付け可能 |
| スキルの誘導 | クライアントでは指定できないので **メッセージ本文の言い回しで誘導** し、返ってきた型（Message か Task か）で事後判定する | Agent Card の examples が「Investigate …」「Root cause …」= investigate、質問形 = chat と読める。Phase 0 で実測して確定 |
| 会話の継続 | `contextId` を **インシデント単位で永続化** し、追加質問や再調査で使い回す | 調査結果への追い質問（「じゃあどう直す？」）が北極星の要 |

---

## 1. 前提と確認済み事実

- DevOps Agent の A2A は **A2A v1.0 / HTTP+JSON バインディング**。エンドポイントは `https://connect.aidevops.{region}.api.aws/a2a/*`、Agent Card は `/.well-known/agent-card.json`
- 対応オペレーション: SendMessage / SendStreamingMessage / GetTask / ListTasks / CancelTask / SubscribeToTask
- 必須ヘッダ: `A2A-Version: 1.0`（無いと 400）。初回応答 5〜30 秒、調査 5〜8 分 → クライアントタイムアウトは **120 秒以上**
- 認証: Bearer（Agent Space 単位、最大 60 日で失効）または SigV4。トークンは Agent Space の Configuration で機能を有効化してから Web App で発行
- `@a2a-js/sdk` v1.0 は JSON-RPC / REST / gRPC の 3 トランスポートをクライアント・サーバ両方でサポート。`ClientFactory` + `RestTransportFactory`、`CallInterceptor` でヘッダ注入、各呼び出しで `RequestOptions.signal` によるキャンセルが可能
- Mastra は `@mastra/core@1.33.1` 以降で `A2AAgent` を提供するが、本計画では JSON-RPC 制約のため不採用（前提どおり）

---

## 2. アーキテクチャ

```
[Slack / Web UI]
      │  「ECS が 503 出してる、見て」
      ▼
[Mastra: Incident Copilot Agent]
      │  tools
      ├─ devops_ask         … 即答系（chat）
      ├─ devops_investigate … 調査開始（investigate → taskId）
      └─ devops_task_status … 進捗/結果取得
      │
      ▼
[packages/devops-agent-a2a]  ← 今回作るクライアント
      │  @a2a-js/sdk RestTransport + Bearer/SigV4 interceptor
      │  Message | Task → DevOpsResult に正規化
      ▼
[AWS DevOps Agent  /a2a  (HTTP+JSON, A2A v1.0)]

[Mastra Workflow: incident-investigation]
   start → poll(GetTask) → summarize → (人が承認) → 対応手順提示
```

パッケージ構成（monorepo 想定）:

```
packages/
  devops-agent-a2a/        # 純粋な A2A クライアント。Mastra 非依存
    src/client.ts          # DevOpsAgentClient
    src/auth.ts            # bearerInterceptor / sigv4Interceptor
    src/normalize.ts       # Message|Task → DevOpsResult
    src/types.ts
  mastra-app/
    src/mastra/tools/devops.ts
    src/mastra/workflows/incident-investigation.ts
    src/mastra/agents/incident-copilot.ts
```

---

## 3. クライアント設計（packages/devops-agent-a2a）

### 3.1 統一型 `DevOpsResult`

Message / Task の違いをここで吸収する。Mastra 側はこの型しか扱わない。

```ts
export type DevOpsResult =
  | { kind: "answer";      contextId: string; text: string }                        // chat 即答
  | { kind: "task_started"; contextId: string; taskId: string; state: TaskState }  // investigate 受付
  | { kind: "task_result"; contextId: string; taskId: string; text: string; artifacts: Artifact[] }
  | { kind: "needs_input"; contextId: string; taskId: string; question: string }   // input-required
  | { kind: "failed";      contextId?: string; taskId?: string; reason: string };
```

### 3.2 クライアント本体（骨子）

```ts
import { ClientFactory, RestTransportFactory, type CallInterceptor } from "@a2a-js/sdk/client";
import type { AgentCard, Message, Task } from "@a2a-js/sdk";

const A2A_VERSION_HEADER = { "A2A-Version": "1.0" };

export const bearerInterceptor = (token: () => Promise<string>): CallInterceptor => ({
  async before(ctx) {
    ctx.headers = { ...ctx.headers, ...A2A_VERSION_HEADER, Authorization: `Bearer ${await token()}` };
    return ctx;
  },
});

export class DevOpsAgentClient {
  private client!: Awaited<ReturnType<ClientFactory["createFromAgentCard"]>>;

  static async create(opts: { region: string; interceptors: CallInterceptor[]; card?: AgentCard }) {
    const base = `https://connect.aidevops.${opts.region}.api.aws`;
    const factory = new ClientFactory({
      transports: [new RestTransportFactory()],   // REST のみ。JSON-RPC は登録しない
      interceptors: opts.interceptors,
    });
    const self = new DevOpsAgentClient();
    self.client = opts.card
      ? await factory.createFromAgentCard(opts.card)
      : await factory.createFromUrl(base, "/.well-known/agent-card.json");
    return self;
  }

  /** chat / investigate どちらでも受けられる入口 */
  async send(text: string, contextId?: string, signal?: AbortSignal): Promise<DevOpsResult> {
    const res = await this.client.sendMessage(
      { message: { messageId: crypto.randomUUID(), role: "user", contextId,
                   parts: [{ text }] } },
      { signal },
    );
    return normalize(res);           // Message か Task かをここで判定
  }

  async getTask(taskId: string, signal?: AbortSignal): Promise<DevOpsResult> {
    return normalize(await this.client.getTask({ id: taskId }, { signal }));
  }

  async cancel(taskId: string) { return this.client.cancelTask({ id: taskId }); }
}
```

※ `@a2a-js/sdk` v1.0 の正確な API 名（`ClientFactory` のオプション形、Interceptor のフック名、`parts` の形）は実装時に型定義で確認する。上記は構造を示す骨子。

### 3.3 正規化ロジック（normalize.ts）

```
入力が Message            → answer（parts の text を結合）
入力が Task:
  state = completed       → task_result（status.message + artifacts の text を結合）
  state = submitted/working → task_started
  state = input-required  → needs_input（status.message を質問として抽出）
  state = failed/canceled/rejected → failed
```

判定のポイント:
- chat は Message で返る「はず」だが、Task(completed) で返る可能性もある。**Task(completed) も即答として扱う**ことで、どちらでも Mastra 側が壊れない
- `contextId` は必ず持ち回る（Message にも Task にも入る）

### 3.4 待機ヘルパ

```ts
export async function waitForTask(c: DevOpsAgentClient, taskId: string, opts = { intervalMs: 20_000, timeoutMs: 12 * 60_000 }) {
  const deadline = Date.now() + opts.timeoutMs;
  while (Date.now() < deadline) {
    const r = await c.getTask(taskId);
    if (r.kind !== "task_started") return r;    // 終端 or needs_input
    await sleep(opts.intervalMs);
  }
  return { kind: "failed", taskId, reason: "timeout" } as const;
}
```

---

## 4. Mastra 側の設計

### 4.1 ツール（3 本）

| ツール | 役割 | 実装 |
|---|---|---|
| `devops_ask` | コスト・構成・Runbook などの即答質問 | `client.send(text, contextId)` → `answer` を期待。`task_started` が返ったら「調査に回った」旨を返す |
| `devops_investigate` | インシデント調査の開始 | メッセージを「Investigate: …」の形に整形して `send` → `taskId` を返す（待たない） |
| `devops_task_status` | 進捗・結果の取得 | `getTask` → `DevOpsResult` を返す |

ツールの `execute` 内で 5〜8 分待たない。待つのは Workflow の仕事にする（エージェントのターンを長時間塞がない）。

### 4.2 Workflow `incident-investigation`

```
step1 start      : devops_investigate → taskId, contextId を保存
step2 poll       : waitForTask（Mastra の sleep/ループで 20 秒間隔）
step3 handle     : needs_input → suspend して人に質問を返し、回答で resume → send(answer, contextId)
step4 summarize  : LLM で「原因 / 影響 / 今すぐやること / 恒久対策」に整形
step5 handoff    : Slack スレッドに投稿。対応アクションは人が承認してから実行
```

### 4.3 Incident Copilot Agent（instructions の要点）

- 利用者は AWS 非専門家という前提。専門用語は必ず一文で補足する
- 「壊れている」「遅い」「エラーが出る」系 → `devops_investigate`。「いくら」「どうなってる」「手順ある？」系 → `devops_ask`
- investigate は 5〜8 分かかることを **先に伝える**
- 結果は「原因 / 影響範囲 / 今すぐやること（コマンド or コンソール手順）/ 再発防止」の 4 段で返す
- 同じインシデントの追い質問は同じ `contextId` を使う（Slack スレッド ID ↔ contextId をストレージに紐づけ）

---

## 5. フェーズ計画

### Phase 0 — 疎通スパイク（1〜2 日）

目的: 憶測を潰す。以下を `curl` / 最小 Node スクリプトで実測し、結果を `docs/spike.md` に残す。

- [ ] Agent Space で Access Token を有効化 → `operate` / `agent` でトークン発行 → Secrets Manager 保管
- [ ] `GET /.well-known/agent-card.json` 取得（Bearer 不要か確認）
- [ ] REST の実パス確認（v1.0 HTTP+JSON バインディング: `POST /a2a/v1/message:send` などの想定。card の `url` を base にする）
- [ ] chat 系メッセージ送信 → **Message で返るか Task で返るか** を確認
- [ ] investigate 系メッセージ送信 → Task の `state` 遷移、完了までの秒数、`artifacts` の中身（text/plain か application/json か）を確認
- [ ] `input-required` が発生するケースがあるかを確認
- [ ] `message.metadata` にスキル指定のヒントが効くか試す（効けば言い回し誘導より確実）
- [ ] `@a2a-js/sdk` の `RestTransportFactory` で同じことができるか確認（Interceptor でヘッダが通るか）

### Phase 1 — クライアント実装（3〜4 日）

- [ ] `packages/devops-agent-a2a` 作成（Mastra 非依存、ESM、Node 20+）
- [ ] `DevOpsAgentClient` / `bearerInterceptor` / `normalize` / `waitForTask`
- [ ] Agent Card はリポジトリに JSON で同梱し `createFromAgentCard` を既定に（起動時の外部依存を減らす）。`createFromUrl` は fallback
- [ ] タイムアウト 120 秒、`AbortSignal` 伝播、401 時の再試行なし（トークン失効は運用で検知）
- [ ] テスト: Phase 0 で保存した実レスポンス JSON をフィクスチャにして `normalize` を単体テスト。REST 層は `msw` で模擬

### Phase 2 — Mastra 統合（3 日）

- [ ] 3 ツール実装（Zod 入出力スキーマ = `DevOpsResult`）
- [ ] `incident-investigation` Workflow（ポーリング、suspend/resume、タイムアウト時の `CancelTask`）
- [ ] `contextId` ストレージ（Mastra Storage or DynamoDB。キー: incidentId / Slack thread_ts）
- [ ] Incident Copilot Agent と instructions

### Phase 3 — 「誰でも解決できる」体験（1 週間）

- [ ] Slack 入口（`@copilot ECS が 503` → スレッドで進捗と結果）
- [ ] 結果の 4 段フォーマットと、コンソール手順へのリンク化
- [ ] 追い質問（同 contextId で `devops_ask`）
- [ ] 承認フロー: 対応アクションは Mastra の suspend で人の承認を待つ。**エージェント応答の自動実行はしない**（AWS ドキュメントも人のレビューを推奨）

### Phase 4 — 運用強化（随時）

- [ ] SigV4 インターセプタ（`@aws-sdk/signature-v4` or `aws4fetch` でリクエスト署名。複数 Agent Space 対応が必要になったら）
- [ ] トークンローテーション（60 日上限。Secrets Manager rotation + 期限 7 日前アラート）
- [ ] `SubscribeToTask`(SSE) によるリアルタイム進捗
- [ ] 観測性: Mastra のトレースに taskId / contextId を付与。調査所要時間・失敗率をメトリクス化
- [ ] 評価: 過去インシデント 10 件で「非専門家が結果だけで復旧手順を実行できたか」を採点

---

## 6. リスクと対策

| リスク | 対策 |
|---|---|
| chat 依頼が investigate に振られる（or 逆） | 返り型で事後判定するので壊れない。言い回しテンプレを Phase 0 の実測で固定 |
| 5〜8 分の待ちで Slack/HTTP がタイムアウト | ツールは待たず Workflow で非同期化。進捗を定期投稿 |
| Agent Card の形式が SDK の期待と微妙に違う（例: `specVersion` 無し） | 同梱 JSON に必要フィールドを補完した上で `createFromAgentCard` |
| REST パスの想定違い | Phase 0 で確定。SDK が合わなければ `fetch` 直叩きの薄い REST 層に差し替え（`normalize` 以降は共通） |
| トークン漏洩 | `agent` client type + IP allowlist + Secrets Manager。CloudTrail の AssumeRole セッションタグで追跡可能 |

---

## 7. 完了条件（MVP）

1. Slack で「〇〇が壊れた」と書くと、10 分以内に「原因 / 影響 / 今すぐやること / 再発防止」が返る
2. 同スレッドでの追い質問に文脈を保って答えられる
3. DevOps Agent が Message で返しても Task で返しても、Mastra 側のコード変更なしに動く
4. トークン失効・タイムアウト・調査失敗が利用者に分かる言葉で通知される
