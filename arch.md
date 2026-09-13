# PagerDuty → Keep 置き換えアーキテクチャ 検討サマリー（v3）

作成日: 2026-09-14
対象: PagerDuty のルーティング層を Keep（keephq/keep）へ置き換え、EKS 障害時・Keep 停止時・東京リージョン障害時にも critical アラートをオンコール担当へ届ける構成の検討
v2 からの変更: 外部レビューで指摘された信頼性保証の境界の欠陥（Keep の 202 の扱い、冪等キー、RPO の表現、ハートビートの観測範囲、NAT 構成）を修正。v3 は「耐久性のあるアラートパイプラインの中に Keep がプラグインされる」構造とする。

本書は会話で確認された事実と決定事項のみを記載し、未確定の点は「未確認事項」に分離している。

---

## 1. 背景（確認済みの事実）

- 100 以上のリリースを Kubernetes（EKS）上で運用している
- PagerDuty を 20 ユーザーで契約し、年間 200 万円かかっている
- アラート量は月約 5,000 件（発生間隔は不明）
- オンコール管理・エスカレーション・電話発報は別ツールの使用を検討している
- 通知経路は Keep → SQS → 内製ツール（EKS 外）→ 内製コミュニケーションツール
- 内製ツールは SQS から pull する。役割は「サービスのルーティング（コミュニケーションツールのどのルームにアラートを連携するか）」
- 要件: アラートはオンコール担当にできるだけダウンタイムなしで転送する
- 要件: 東京リージョン障害時に大阪リージョンで待機系として動ける準備をする

## 2. なぜやるのか

1. PagerDuty の年間 200 万円を削減する
2. Keep を本番 EKS 上に置くと EKS 障害時にアラート経路そのものが止まるため、EKS に依存しない配置で取りこぼしを防ぐ

## 3. So that（達成したい状態）

- critical アラートが、EKS 障害・Keep 停止・Redis 消失・東京リージョン障害のいずれでもオンコール担当に届き続ける
- 非 critical アラートは失われず、Keep 復旧後に自動で再処理される
- Keep が一度も保存しなかったアラートも、Keep の外側の記録から復元できる
- 年間コストを 200 万円から約 108〜126 万円（DR・レビュー反映分込み）に下げる

---

## 4. 調査で判明した前提

| 項目 | 内容 |
|---|---|
| Keep の位置づけ | Keep 自身が「IRM の前段の知的レイヤー」と定義。オンコールスケジュール・エスカレーション・電話発報はネイティブに持たない |
| Keep の事業状況 | 2025 年 5 月に Elastic が買収。OSS リポジトリは 2026 年 7 月時点で更新継続 |
| Keep の構成要素 | Keep API（FastAPI）、UI（Next.js）、Soketi（任意）、DB（PostgreSQL 等）。ECS Fargate 向け公式手順あり |
| **Keep の 202 応答** | `/alerts/event/{provider}` は Redis 有効時、ARQ へ enqueue した時点で 202 を返す。**202 は DB 永続化完了を意味しない。** Redis 無効時もバックグラウンドタスク処理のため同様。`ARQ_EXPIRES` 既定 3,600 秒超で未処理ジョブは破棄 |
| Keep の pull 機能 | 標準 pull は `KEEP_PULL_INTERVAL` 既定 7 日。公式が「ワークフロー自動化が使えないため push を強く推奨」と明記 |
| Keep の耐久性 | ワークフロー失敗の自動再試行はない。`KEEP_USE_LIMITER` 既定 OFF |
| Keep の識別子 | fingerprint はアラートの同一性を表す。event_id を持てる設計 |
| SQS FIFO の重複排除 | 重複排除期間は 5 分。5 分を超えた再送は同一 MessageDeduplicationId でも配送され得る。AWS も下流の冪等性を要求 |
| Lambda と SQS | イベントソースマッピングは同一リージョン内のみ |
| RDS クロスリージョンリードレプリカ | 非同期レプリケーション。昇格は DB 再起動を伴い数分以上かかることがある |
| Regional NAT Gateway | 2025 年 11 月発表。単一 NAT がワークロードのある AZ へ自動拡張・縮小。GovCloud・中国を除く全リージョンで GA |
| AWS Incident Manager | 2025 年 11 月 7 日で新規顧客受付停止 |
| Grafana OnCall OSS | 2026 年 3 月 24 日にアーカイブ |
| PagerDuty 定価 | Professional 21 ドル/ユーザー/月（年契約）、Business 41 ドル/ユーザー/月 |

---

## 5. 検討した 3 案と裁定

| 案 | 構成 | 評価 |
|---|---|---|
| A | 本番と別の EKS（ops クラスタ）+ RDS Multi-AZ | EKS というプラットフォームの相関障害リスクが残る |
| B | ECS Fargate + RDS Multi-AZ | EKS から独立。ただし Keep 停止中は受信口が失われる |
| C | 案 B + フルマネージド耐久受信層 | Keep 停止時も受信を保証 |

**裁定: 案 C を採用。** 外部レビューでも「案 C の方向性は採用」「Keep を critical の配送経路から外す判断は正しい」と評価された。ただし v2 は「条件付き No-Go」であり、以下の修正を施した v3 を本番採用の前提とする。

---

## 6. v2 からの修正点（外部レビュー反映）

| # | v2 の問題 | v3 の修正 | 優先度 |
|---|---|---|---|
| 1 | Keep の 202 を永続化成功と見なし、受信 SQS を削除していた | Keep を耐久性の境界にしない。正規化 Lambda の成功条件を「AWS の耐久ストレージへ書けたこと」に変更。keep-delivery.fifo と DynamoDB Journal を導入 | P0 |
| 2 | critical 直送と Keep 投入の部分失敗で受信メッセージが再配信され、両方が再実行される | Lambda を Keep の成否から切り離す（keep-delivery.fifo 経由） | P0 |
| 3 | fingerprint+status を冪等キーにしていた（firing→resolved→firing の再発を落とす） | transition_id を導入。直送経路と Keep 経路で同一 ID を引き回す | P0 |
| 4 | 受信 SQS が FIFO（正規化前に安定した MessageGroupId を作れず、単一グループでは HOL ブロッキング） | 受信は Standard、正規化後を FIFO | P0 |
| 5 | 「critical RPO 0」と記載（東京 SQS のバックログは東京障害中に利用不能） | 表現を修正。必要なら critical の入口二重化 | P0（文書）/ P1（実装） |
| 6 | ハートビートが Keep→出口しか監視していない | 入口→出口の合成監視（大阪から）を追加 | P0 |
| 7 | NAT Gateway 1 台（ゾーン型）で AZ 障害時に外向き通信が失われる | Regional NAT Gateway へ変更 | P0 |
| 8 | 「RPO 数秒」「RTO 1〜3 分」を保証値のように記載 | 目標値として記載し、DR GameDay で実測して SLO 化 | P0（文書） |
| 9 | 大阪レプリカが Single-AZ | 大阪で継続運用する可能性を考慮し Multi-AZ 化を検討 | P1 |

---

## 7. 採用アーキテクチャ（v3）

### 7.1 設計原則

- 守るべきものは「critical アラートの配送」であり、Keep の高可用化ではない
- Keep は「失ってはいけないデータの保管場所」ではなく「再実行可能な処理レイヤー」
- 失ってはいけない記録（イベント台帳、冪等性）は Keep の外側の AWS マネージドストレージに置く
- Keep をアップグレードして壊しても、Redis を全部失っても、後から再処理できる

### 7.2 構成（東京リージョン）

```
監視ソース（本番EKS / Alertmanager、CloudWatch、SaaS監視、大阪からの外形監視）
  │ HTTPS（Route 53 フェイルオーバー DNS 経由）
  ▼
API Gateway（HTTP API）
  ▼
ingress.standard（SQS Standard、14日保持、DLQ付き）
  ▼ イベントソースマッピング
Normalize Lambda
  ├─ 正規化、event_id / transition_id 採番
  ├─ DynamoDB AlertEventJournal へ Conditional Put
  ├─ critical → alerts.fifo へ直送（source=direct）
  └─ 全件 → keep-delivery.fifo
        ▼ イベントソースマッピング
     Keep Dispatcher Lambda → Keep API（push）
        ▼
     Keep API ×2（ECS Fargate、2AZ、opsアカウント）
       ├─ RDS PostgreSQL Multi-AZ
       └─ ElastiCache Redis Multi-AZ（ARQ）
        ▼ ワークフロー（amazonsqs アクション、source=keep）
alerts.fifo（SQS FIFO、唯一の契約面、DLQ付き）
  ▼ pull（東京・大阪の両方を常時）
内製ツール（EKS外）: サービス→ルームのルーティング、transition_id による冪等処理、配送結果を Journal へ書き戻し
  ▼
内製コミュニケーションツール / オンコール

横に:
Reconciler Lambda（スケジュール実行）
  ├─ Journal → Keep: keep_status=pending が N 分以上なら Keep API で確認し、なければ keep-delivery.fifo へ再投入
  └─ Journal → 通知: notification_status=pending の critical が N 分以上なら独立 SNS 経路で通知
```

### 7.3 Normalize Lambda の成功条件

| severity | ingress メッセージを削除（ACK）する条件 |
|---|---|
| critical | Journal への書込成功 + alerts.fifo への書込成功 + keep-delivery.fifo への書込成功 |
| 非 critical | Journal への書込成功 + keep-delivery.fifo への書込成功 |

Keep が 6 時間停止しても 24 時間停止しても、受信系には影響しない。Keep Dispatcher は Keep が 5xx/無応答なら keep-delivery.fifo のメッセージを残して終了し、SQS が再試行する。

### 7.4 識別子の設計

| ID | 意味 | 用途 |
|---|---|---|
| fingerprint | 何のアラートか | alerts.fifo / keep-delivery.fifo の MessageGroupId、相関 |
| event_id | 今回発生した 1 回 | Journal の PK、イベント識別 |
| transition_id | 今回の firing / resolved | MessageDeduplicationId、通知の冪等キー。直送経路と Keep 経路の両方に同一値を引き回す |
| source_event_time | ソースで起きた時刻 | 順序判定 |

fingerprint+status および lastReceived を冪等キーに使わない。

### 7.5 DynamoDB AlertEventJournal

| 属性 | 内容 |
|---|---|
| PK | event_id |
| fingerprint / status / severity | 正規化後の値 |
| transition_id | 通知冪等キー |
| received_at / source_event_time | 時刻 |
| keep_status | pending / accepted / confirmed（Keep API で存在確認済み） |
| notification_status | pending / delivered（内製ツールが書き戻す） |
| region | 受信リージョン |

- Conditional Put で重複投入を防ぐ
- DynamoDB グローバルテーブルで東京・大阪に複製
- 月 5,000 件ではコストは無視できる

### 7.6 各部品の役割

| 部品 | 役割 |
|---|---|
| API Gateway | 唯一の HTTPS 受け口。判断せず ingress.standard に書き込む |
| ingress.standard | Standard キュー。at-least-once の重複は transition_id で下流が吸収 |
| Normalize Lambda | 正規化、ID 採番、Journal 記録、critical 直送、keep-delivery 投入。バッチ部分失敗レポート有効 |
| keep-delivery.fifo | Keep 投入待ちキュー。Keep の停止時間を吸収 |
| Keep Dispatcher Lambda | keep-delivery.fifo から Keep API へ push。失敗時はメッセージを残す |
| Keep | 重複排除・相関・エンリッチ。ワークフローで alerts.fifo へ書き出す。`REDIS=true`、`KEEP_STORE_RAW_ALERTS=true`、`KEEP_USE_LIMITER` OFF |
| Redis（ARQ） | Keep 内部の作業キュー。失っても Journal から再処理可能 |
| RDS | Keep の永続記録。Multi-AZ |
| alerts.fifo | Normalize Lambda（direct）と Keep（keep）が書き、内製ツールだけが読む契約面。MessageGroupId=fingerprint、MessageDeduplicationId=transition_id |
| AlertEventJournal | Keep の外側の不変イベント記録。照合の基準 |
| Reconciler Lambda | Journal を基準に Keep 側・通知側の未達を検出し、再投入または独立通知 |
| 内製ツール | 東京・大阪の alerts.fifo を常時 pull。サービス→ルームの対応表を一元管理。transition_id で 1 回だけ通知（direct と keep のどちらが先でも 1 回）。配送結果を Journal に書き戻す。ack/解決は Keep API へ書き戻す |
| 内製コミュニケーションツール | 最終通知先 |

### 7.7 Keep ワークフロー例（構文は Keep のバージョンで要確認）

```yaml
workflow:
  id: forward-to-oncall-queue
  triggers:
    - type: alert
  actions:
    - name: enqueue
      provider:
        type: amazonsqs
        config: "{{ providers.oncall-queue }}"
        with:
          message: keep.json_dumps(alert)
          group_id: "{{ alert.fingerprint }}"
          dedup_id: "{{ alert.transition_id }}"
          severity: "{{ alert.severity }}"
          status: "{{ alert.status }}"
          service: "{{ alert.service }}"
          event_id: "{{ alert.event_id }}"
          source: keep
```

### 7.8 ネットワーク・IAM

- NAT は Regional NAT Gateway（ゾーン型 1 台は使わない）
- SQS / Secrets Manager / ECR / CloudWatch Logs / DynamoDB は VPC エンドポイント化を検討
- Keep タスクロール: alerts.fifo への `sqs:SendMessage` のみ
- 内製ツール: alerts.fifo の Receive / Delete / ChangeMessageVisibility、Journal の UpdateItem（notification_status）のみ
- 全キュー・テーブルで SSE-KMS

---

## 8. 障害モードと対策

| 想定障害 | 対策 |
|---|---|
| 本番 EKS 障害 | Keep と受信層を EKS 外・別アカウントに配置 |
| Normalize Lambda 停止 | ingress.standard に滞留。復旧後に自動処理 |
| Keep 停止（API / Redis / RDS） | critical は直送で到達。非 critical は keep-delivery.fifo に滞留し復旧後に Dispatcher が再投入 |
| Keep が 202 を返したが永続化しなかった | Reconciler が Journal の keep_status=pending を検出し、Keep API で確認後に再投入 |
| Keep → alerts.fifo 書き込み失敗 | 同上（Keep に存在するが alerts.fifo に出ていない場合は Reconciler が直接 alerts.fifo へ投入） |
| Redis 全消失 | 作業中ジョブは失われるが、Journal から再処理可能 |
| 内製ツール停止 | alerts.fifo に滞留。DLQ 滞留・最古メッセージ滞留時間を CloudWatch アラーム → 独立 SNS 経路で通知。critical は Reconciler が notification_status=pending を検出し独立通知 |
| 入口の静かな停止（Route 53 / API Gateway / ingress / Normalize） | 大阪からの合成監視（Ingress synthetic heartbeat）で検出 |
| Keep 処理の静かな停止 | Keep の interval ワークフローによる Keep processing heartbeat で検出 |
| AZ 障害 | Keep 2AZ 分散、ALB ヘルスチェック、Regional NAT、RDS / Redis Multi-AZ |
| Keep のバージョンアップ | ECS ローリングデプロイ（minimumHealthyPercent=100）。壊れても Journal から再処理可能 |

ハートビートは 2 種類を持つ:
1. **Ingress synthetic heartbeat**: 大阪の Synthetics canary → 東京 API Gateway → ingress → Normalize → canary 用 alerts.fifo まで到達を確認
2. **Keep processing heartbeat**: Keep の interval ワークフローで 5 分ごとに alerts.fifo へ投入し、末端で未着を検出

---

## 9. DR（東京 → 大阪、パイロットライト）

### 9.1 大阪に置くもの

| 区分 | 部品 | 状態 |
|---|---|---|
| 常時稼働 | API Gateway、ingress.standard、Normalize Lambda、alerts.fifo、keep-delivery.fifo、DLQ、Synthetics canary（東京の外形監視） | 東京と同じコードをデプロイ |
| 常時複製 | DynamoDB AlertEventJournal（グローバルテーブル） | 東京・大阪で双方向複製 |
| 待機 | Keep API/UI の Fargate サービス | desired count 0 |
| 待機 | RDS クロスリージョンリードレプリカ | 非同期複製。Multi-AZ 化は P1 |
| 待機 | ECR クロスリージョンレプリケーション、Secrets Manager マルチリージョンレプリケーション | |
| 事前作成 | ALB、Regional NAT Gateway | |
| 不要 | ElastiCache Redis | フェイルオーバー時に新規作成 |

### 9.2 フェイルオーバー

- **入口**: Route 53 フェイルオーバールーティング（TTL 60 秒）。EvaluateTargetHealth のみに頼らず、大阪 canary の結果を CloudWatch アラーム化した Route 53 ヘルスチェックで切り替える
- **critical の経路**: 切り替え後、大阪の API Gateway → ingress → Normalize → 大阪 alerts.fifo → 内製ツールで到達。Keep の起動を待たない
- **Keep の経路**: (1) RDS リードレプリカ昇格、(2) Keep Fargate を desired count 2 へ、(3) amazonsqs アクションの向き先を大阪 alerts.fifo へ切り替え。Step Functions で自動化
- **内製ツール**: 東京・大阪の alerts.fifo を常時両方 pull。出力側のフェイルオーバー操作は不要

### 9.3 RTO / RPO の正確な表現（目標値。保証値ではない）

| 対象 | 表現 |
|---|---|
| critical（東京障害後に新規受信） | RTO 1〜数分を目標 |
| critical（東京で受理済みのバックログ） | 東京復旧まで利用不能となり得る。**RPO 0 ではない** |
| Keep の経路 | RTO 15〜30 分を目標。リードレプリカ昇格は数分以上かかることがあるため実測が必要 |
| Keep の DB | 通常時 ReplicaLag 数秒を目標。実際の RPO は障害発生時点の ReplicaLag に依存 |

### 9.4 critical の RPO を 0 に近づける場合（P1）

- 監視ソースが東京・大阪の両 API Gateway へ dual-send する。両方で同じ transition_id を使い、内製ツールの冪等処理で 1 回だけ通知
- ただし東京 EKS 上の Alertmanager は東京リージョンと共に消えるため、dual-send できるのは東京 EKS 外のソース（CloudWatch、SaaS 監視）に限られる
- 「東京が丸ごと消えた」ことを検知するのは大阪の Synthetics canary であり、これを大阪 ingress に critical を投入する正規のアラートソースとして扱う

### 9.5 フェイルバック

- 東京復旧後、東京の ingress / keep-delivery のバックログが自動で流れ始める。東京側の Keep Fargate は desired count 0 のまま起動させない（両リージョンの Keep が別 DB で同時稼働するとアラート状態が分裂する）
- Keep の DB を東京へ戻す場合は、大阪を新プライマリとして東京にリードレプリカを作り直し、計画停止で逆方向に昇格する
- Keep のワークフロー・プロバイダ設定は Git で管理し、両リージョンに同じものを適用する

---

## 10. コスト

### 10.1 東京リージョン月額（オンデマンド、x86、1 ドル=150 円換算）

| 項目 | 月額(USD) |
|---|---|
| Fargate: Keep API 1vCPU/2GB ×2、UI 0.5vCPU/1GB、Soketi（任意） | 126 |
| RDS PostgreSQL db.t4g.medium Multi-AZ + gp3 50GB | 137 |
| ElastiCache Redis cache.t4g.small ×2 | 60 |
| ALB | 25 |
| Regional NAT Gateway（2AZ 相当） | 90 |
| API Gateway / Lambda ×3 / SQS ×3 + DLQ | ほぼ無料枠内 |
| DynamoDB AlertEventJournal（グローバルテーブル分含む） | 3〜5 |
| SNS / CloudWatch / Secrets Manager / KMS / Route 53 | 13 |
| **小計** | **約 455〜460 ドル** |

### 10.2 大阪リージョン月額

| 項目 | 月額(USD) |
|---|---|
| RDS リードレプリカ db.t4g.medium（Single-AZ。Multi-AZ 化は +65） | 65 |
| ストレージ | 7 |
| ALB | 20 |
| Regional NAT Gateway | 45〜90 |
| Synthetics canary（5 分間隔、東京外形監視） | 10〜15 |
| Route 53 ヘルスチェック、Secrets 複製、ECR 複製、クロスリージョン転送 | 10 |
| API Gateway / SQS / Lambda（未使用時） | ほぼ 0 |
| **小計** | **約 160〜210 ドル** |

### 10.3 合計と年間比較

- 東京 + 大阪: **約 600〜700 ドル/月（約 9〜10.5 万円）、年約 108〜126 万円**
- 削減オプション: Soketi 停止（−13）、Graviton 化（−約 25、arm64 イメージ提供有無は要確認）、RDS 1 年 RI（−約 40）、VPC エンドポイント化による NAT 転送量削減

| 選択肢 | 年額 | 現状比 |
|---|---|---|
| 現状 PagerDuty（20 ユーザー） | 200 万円 | — |
| v3 構成 + DR | 約 108〜126 万円 | −74〜92 万円 |
| 参考: PagerDuty Business 定価・年契約 | 約 148 万円 | −52 万円 |
| 参考: PagerDuty Professional 定価・年契約 | 約 76 万円 | −124 万円 |

現契約は 1 ユーザーあたり約 8,300 円/月（約 55 ドル）で、定価 Business を上回っている。

含まれないもの: 内製ツール・内製コミュニケーションツールのホスティング費と DR 費、構築工数（v2 より Reconciler・Journal・canary・GameDay 分が増える）、運用工数、並走期間の PagerDuty 費、別ツールの SMS/音声 API 従量費。

---

## 11. 未確認事項

### 責務分担
1. オンコール判定・エスカレーション・電話発報をどこで行うか
2. 「オンコール担当にダウンタイムなしで転送」は、ルームへの投稿で満たされるか、電話・プッシュ通知まで含むか
3. 「サービス → ルーム」の対応表は内製ツールが一元管理する前提でよいか
4. 「critical」の定義（Normalize Lambda が各ソースからどう判定するか）
5. 内製ツールが Journal の notification_status を書き戻す契約を受け入れられるか（代替: Reconciler が内製ツールに問い合わせる）

### Keep
6. Keep API `/alerts/event/{provider}` の認証とレスポンスコードの仕様
7. Keep が transition_id / event_id をアラート属性として保持し、ワークフローから参照できるか
8. Keep の重複排除ルールが同一 fingerprint の再発（firing→resolved→firing）を抑止しないか
9. Keep Dispatcher Lambda → Keep API の経路（Keep の ALB を内部向けにする場合の VPC 配置）
10. amazonsqs アクションの向き先をリージョン間で切り替える手順
11. Keep の arm64 イメージ提供有無
12. Keep の OSS が「community-maintained」フェーズであるとする外部レビューの記述の出典

### DR・周辺
13. 内製ツールと内製コミュニケーションツールが東京リージョン障害の影響を受けないか
14. 東京 EKS 外の監視ソースで大阪エンドポイントへの dual-send が可能か

### 契約
15. 現在の PagerDuty 契約の内訳
16. PagerDuty の途中解約条件と並走期間の二重コスト

---

## 12. 本番 Go 条件（外部レビューの 6 条件を採用）

1. Keep の 202 Accepted を耐久性の境界にしない。keep-delivery.fifo + AlertEventJournal + Reconciler を入れる
2. fingerprint+status を廃止し、event_id / transition_id を導入する。direct と Keep の両経路で同一 ID を維持する
3. 「critical RPO 0」を削除する。DNS フェイルオーバーのみなら RPO 0 ではない。必要なら critical の dual-ingress を実装する
4. 大阪から入口→出口の合成監視を行う。Keep processing heartbeat とは別に持つ
5. ゾーン型 NAT 1 台を使わない。Regional NAT Gateway にする
6. DR GameDay を PagerDuty 解約条件にする。Keep 停止、Redis 停止、RDS フェイルオーバー、AZ 障害、東京リージョン隔離を実際に起こし、RTO / RPO を測定して SLO 化する

---

## 13. 最終裁定

1. **アーキテクチャは案 C（v3）を採用する。** 上記 6 つの Go 条件を満たした時点で本番 Go とする。満たすまでは条件付き No-Go
2. **Keep は再実行可能な処理レイヤーとして扱う。** 失ってはいけない記録（Journal、冪等性）は Keep の外側の AWS マネージドストレージに置く。Keep の高可用化に投資するより、Keep の外側の耐久性を優先する
3. **critical は Keep を通さない直送を常時行う。** Normalize Lambda の成功条件は「AWS の耐久ストレージへ書けたこと」であり、Keep の成否に依存しない
4. **冪等性は transition_id で担保する。** SQS FIFO の 5 分窓に依存せず、内製ツールが durable な冪等ストア（Journal）で 1 回だけ通知する
5. **DR はパイロットライトとする。** RTO / RPO は目標値として扱い、GameDay で実測する。critical の RPO を 0 に近づける dual-ingress は P1
6. **責務分担は「Keep はノイズ削減・相関・エンリッチ、内製ツールはルーティングと配送、AWS マネージド層は耐久性と冪等性」とする**
7. **コスト面の判断。** v3 は v2 より月 60〜130 ドル増えるが、年約 74〜92 万円の削減は成立する。現契約の内訳を先に確認し、交渉後も差が残るなら移行を進める
8. **移行手順。** PagerDuty を解約せず並走させ、10〜20 サービスで v3 の全経路（direct、Keep 経由、Reconciler、2 種のハートビート）を検証し、GameDay で RTO / RPO を実測してから全面切り替えする

---

## 付録: コスト単価の前提

- Fargate（東京、x86）: 0.05056 ドル/vCPU 時、0.00553 ドル/GB 時
- 稼働時間: 730 時間/月
- 為替: 1 ドル=150 円
- Regional NAT Gateway の課金は AZ 展開数に依存するため 2AZ 相当で概算
- 単価は 2026 年 9 月時点の把握値。確定時は AWS Pricing Calculator で再確認すること
