# PagerDuty → Keep 置き換えアーキテクチャ 検討サマリー（v4）

作成日: 2026-09-17
対象: PagerDuty のルーティング層を Keep（keephq/keep）へ置き換え、EKS 障害時・Keep 停止時・東京リージョン障害時にも critical アラートをオンコール担当へ届ける構成の検討と、段階的な実装戦略
v3 からの変更: 内製ツールが未作成であることを反映（Lambda ベースで新規構築）、受信口のパブリックホストゾーン化、東京 MVP → 大阪 DR の段階戦略、MVP の範囲と完了条件、後から変えない設計前提を追加

本書は会話で確認された事実と決定事項のみを記載し、未確定の点は「未確認事項」に分離している。

---

## 1. 背景（確認済みの事実）

- 100 以上のリリースを Kubernetes（EKS）上で運用している
- PagerDuty を 20 ユーザーで契約し、年間 200 万円かかっている（1 ユーザーあたり約 55 ドル/月で定価 Business を上回る）
- アラート量は月約 5,000 件（発生間隔は不明）
- オンコール管理・エスカレーション・電話発報は別ツールの使用を検討している
- 内製ツールは**まだ存在せず、これから作成する**。役割は「サービスのルーティング（コミュニケーションツールのどのルームにアラートを連携するか）」
- 要件: アラートはオンコール担当にできるだけダウンタイムなしで転送する
- 要件: 東京リージョン障害時に大阪リージョンで待機系として動ける準備をする
- 戦略: **まず東京リージョンのみで MVP を作り、次に大阪リージョンを作成する**

## 2. なぜやるのか

1. PagerDuty の年間 200 万円を削減する
2. Keep を本番 EKS 上に置くと EKS 障害時にアラート経路そのものが止まるため、EKS に依存しない配置で取りこぼしを防ぐ

## 3. So that（達成したい状態）

- critical アラートが、EKS 障害・Keep 停止・Redis 消失・東京リージョン障害のいずれでもオンコール担当に届き続ける
- 非 critical アラートは失われず、Keep 復旧後に自動で再処理される
- Keep が一度も保存しなかったアラートも、Keep の外側の記録から復元できる
- 年間コストを 200 万円から約 108〜126 万円（DR 込み）に下げる

---

## 4. 調査で判明した前提

| 項目 | 内容 |
|---|---|
| Keep の位置づけ | Keep 自身が「IRM の前段の知的レイヤー」と定義。オンコール・エスカレーション・電話発報はネイティブに持たない |
| Keep の事業状況 | 2025 年 5 月に Elastic が買収。OSS は 2026 年 7 月時点で更新継続 |
| Keep の 202 応答 | `/alerts/event/{provider}` は処理を enqueue した時点で 202 を返す。**202 は DB 永続化完了を意味しない。** `ARQ_EXPIRES` 既定 3,600 秒超で未処理ジョブは破棄 |
| Keep の pull 機能 | 標準 pull は既定 7 日間隔。公式が「ワークフロー自動化が使えないため push を強く推奨」 |
| Keep の耐久性 | ワークフロー失敗の自動再試行はない |
| SQS FIFO の重複排除 | 重複排除期間は 5 分。下流の冪等性が必要 |
| Lambda と SQS | イベントソースマッピングは同一リージョン内のみ |
| Route 53 プライベートホストゾーン | 関連付けた VPC 内（と Resolver インバウンド経由のオンプレ）からのみ解決可能。**社外 SaaS からは解決できない** |
| Route 53 ヘルスチェック | インターネット上のヘルスチェッカーから対象を叩くため、パブリック到達可能なエンドポイントが前提 |
| RDS クロスリージョンリードレプリカ | 非同期。昇格は DB 再起動を伴い数分以上かかることがある |
| Regional NAT Gateway | 2025 年 11 月発表。単一 NAT が AZ に自動拡張。GovCloud・中国を除く全リージョンで GA |
| AWS Incident Manager | 2025 年 11 月 7 日で新規顧客受付停止 |
| Grafana OnCall OSS | 2026 年 3 月 24 日にアーカイブ |
| PagerDuty 定価 | Professional 21 ドル、Business 41 ドル（ユーザー/月、年契約） |

---

## 5. 3 案の比較と裁定

| 案 | 構成 | 評価 |
|---|---|---|
| A | 本番と別の EKS + RDS Multi-AZ | EKS というプラットフォームの相関障害リスクが残る |
| B | ECS Fargate + RDS Multi-AZ | EKS から独立。Keep 停止中は受信口が失われる |
| C | 案 B + フルマネージド耐久受信層 | Keep 停止時も受信を保証 |

**裁定: 案 C を採用。** 外部レビューで「Keep を critical の配送経路から外す判断は正しい」と評価されたが、v2 は条件付き No-Go とされ、以下の修正を施した v3 以降を本番採用の前提とする。

---

## 6. 設計原則と外部レビュー反映事項

### 6.1 設計原則
- 守るべきものは「critical アラートの配送」であり、Keep の高可用化ではない
- Keep は「失ってはいけないデータの保管場所」ではなく「再実行可能な処理レイヤー」
- 失ってはいけない記録（イベント台帳、冪等性）は Keep の外側の AWS マネージドストレージに置く

### 6.2 外部レビュー（v2 → v3）で修正した点

| # | 修正 | 優先度 |
|---|---|---|
| 1 | Keep の 202 を耐久性の境界にしない。Normalize Lambda の成功条件を「AWS の耐久ストレージへ書けたこと」に変更し、keep-delivery.fifo と DynamoDB Journal を導入 | P0 |
| 2 | Lambda を Keep の成否から切り離す | P0 |
| 3 | fingerprint+status を廃止し、event_id / transition_id を導入 | P0 |
| 4 | 受信 SQS を Standard に、正規化後を FIFO に | P0 |
| 5 | 「critical RPO 0」を削除。DNS フェイルオーバーのみなら RPO 0 ではない | P0（文書） |
| 6 | 入口→出口の合成監視を Keep processing heartbeat とは別に持つ | P0 |
| 7 | ゾーン型 NAT 1 台をやめ Regional NAT Gateway に | P0 |
| 8 | RTO / RPO を目標値として記載し GameDay で実測 | P0（文書） |
| 9 | 大阪レプリカの Multi-AZ 化 | P1 |

---

## 7. 採用アーキテクチャ（v3 = 最終形）

### 7.1 構成（東京リージョン）

```
監視ソース（本番EKS / Alertmanager、CloudWatch、SaaS監視、大阪からの外形監視）
  │ HTTPS（パブリックホストゾーンの DNS 名経由）
  ▼
API Gateway（HTTP API、Lambda オーソライザ、スロットリング）
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
     Keep API ×2（ECS Fargate、2AZ、opsアカウント、Regional NAT）
       ├─ RDS PostgreSQL Multi-AZ
       └─ ElastiCache Redis Multi-AZ（ARQ）
        ▼ ワークフロー（amazonsqs アクション、source=keep）
alerts.fifo（SQS FIFO、唯一の契約面、DLQ付き）
  ▼ イベントソースマッピング（各リージョンのキューに各リージョンの Lambda）
内製ツール Lambda: Journal 冪等更新 → 対応表 → コミュニケーションツール API 投稿
  ▼
内製コミュニケーションツール / オンコール

横に:
Reconciler Lambda（スケジュール実行）
  ├─ Journal → Keep: keep_status=pending が N 分以上なら Keep API で確認し、なければ keep-delivery.fifo へ再投入
  └─ Journal → 通知: notification_status=pending の critical が N 分以上なら独立 SNS 経路で通知
```

### 7.2 Normalize Lambda の成功条件（ingress メッセージを削除する条件）

| severity | 条件 |
|---|---|
| critical | Journal 書込成功 + alerts.fifo 書込成功 + keep-delivery.fifo 書込成功 |
| 非 critical | Journal 書込成功 + keep-delivery.fifo 書込成功 |

### 7.3 識別子

| ID | 意味 | 用途 |
|---|---|---|
| fingerprint | 何のアラートか | MessageGroupId、相関 |
| event_id | 今回発生した 1 回 | Journal の PK |
| transition_id | 今回の firing / resolved | MessageDeduplicationId、通知の冪等キー。direct と Keep の両経路で同一値 |
| source_event_time | ソースで起きた時刻 | 順序判定 |

### 7.4 DynamoDB AlertEventJournal

PK: event_id。属性: fingerprint / status / severity / transition_id / received_at / source_event_time / keep_status（pending / accepted / confirmed）/ notification_status（pending / delivered）/ region。Conditional Put で重複投入を防ぎ、DynamoDB Streams を有効化（後のグローバルテーブル化の前提）。

### 7.5 内製ツール（新規構築、Lambda ベース）

「pull」は常駐ポーリングではなく、alerts.fifo に付けた Lambda のイベントソースマッピングで実現する。SQS + Lambda の標準機能（可視性タイムアウト、再試行、DLQ、バッチ部分失敗レポート）をそのまま使い、東京・大阪それぞれの alerts.fifo に各リージョンの Lambda を付けることで「両リージョンを常時 pull」が成立する。

v1 の責務:
1. alerts.fifo からメッセージを受け取り、共通スキーマをパースする
2. transition_id で Journal を Conditional Update（notification_status: pending → delivered）し、成功した場合だけ通知する。これが「direct と keep のどちらが先でも 1 回だけ」の実体
3. サービス→ルームの対応表（DynamoDB または Git 管理の設定）を引き、コミュニケーションツール API へ投稿する。対応表は Keep に持たせない
4. 投稿失敗時は Journal を pending に戻し、メッセージを削除せず終了（SQS が再試行）
5. Keep 経由（source=keep、エンリッチ済み）が後から届いたら、通知は再送せず付加情報の更新に使う

v1 に含めないもの: オンコール判定・エスカレーション・電話発報、ack/解決の Keep への書き戻し、抑止ルール。
内製ツール専用の DB は不要（Journal を冪等ストアとして共用）。コード規模は数百行。

### 7.6 受信口の公開と防御

- 受信口の DNS 名（例: alerts.example.com）は**パブリックホストゾーン**に置く。社外 SaaS はプライベートホストゾーンを解決できず、Route 53 のフェイルオーバーとヘルスチェックもパブリック到達可能なエンドポイントが前提
- SaaS ごとに異なる秘密（ヘッダまたはパス内トークン）を Secrets Manager で管理し、Lambda オーソライザで検証
- 発信元 IP を公開している SaaS は WAF の IP 許可リストで絞る
- API Gateway のスロットリングでバースト上限を設定
- IAM 認証に対応できるソースは IAM 認証を使う
- EKS 内の Alertmanager も同じパブリック DNS 名を叩き、経路を一本化する（スプリットホライズン構成はフェーズ 1 では採らない）

### 7.7 IAM

- Keep タスクロール: alerts.fifo への `sqs:SendMessage` のみ
- 内製ツール Lambda: alerts.fifo の Receive / Delete / ChangeMessageVisibility、Journal の UpdateItem のみ
- 全キュー・テーブルで SSE-KMS

---

## 8. 障害モードと対策

| 想定障害 | 対策 |
|---|---|
| 本番 EKS 障害 | Keep と受信層を EKS 外・別アカウントに配置 |
| Normalize Lambda 停止 | ingress.standard に滞留。復旧後に自動処理 |
| Keep 停止 | critical は直送で到達。非 critical は keep-delivery.fifo に滞留し復旧後に再投入 |
| Keep が 202 を返したが永続化しなかった | Reconciler が Journal の keep_status=pending を検出し再投入 |
| Keep → alerts.fifo 書き込み失敗 | Reconciler が検出し直接 alerts.fifo へ投入 |
| Redis 全消失 | Journal から再処理可能 |
| 内製ツール停止 | alerts.fifo に滞留。DLQ 滞留・最古メッセージ滞留時間を CloudWatch アラーム → 独立 SNS 経路。critical は Reconciler が notification_status=pending を検出し独立通知 |
| 入口の静かな停止 | 大阪からの Ingress synthetic heartbeat（フェーズ 2 では東京内 canary で代用） |
| Keep 処理の静かな停止 | Keep の interval ワークフローによる Keep processing heartbeat |
| AZ 障害 | Keep 2AZ 分散、ALB ヘルスチェック、Regional NAT、RDS / Redis Multi-AZ |
| Keep のバージョンアップ | ECS ローリングデプロイ（minimumHealthyPercent=100）。壊れても Journal から再処理可能 |

---

## 9. 段階的な実装戦略

### 9.1 フェーズ 1: 東京 MVP（PagerDuty 並走）

**範囲**
- 受信: パブリック DNS 名 → API Gateway（Lambda オーソライザ、スロットリング）→ ingress.standard
- 処理: Normalize Lambda（ID 採番、Journal 記録、critical 直送、keep-delivery 投入）
- Keep: ECS Fargate ×2（Regional NAT）、RDS Multi-AZ、Redis、Dispatcher Lambda、amazonsqs ワークフロー
- 出力: alerts.fifo → 内製ツール Lambda → コミュニケーションツール
- 対象: 10〜20 サービス、PagerDuty と同じアラートが両方に届く状態

**完了条件**
1. 対象サービスのアラートが direct 経路と Keep 経路の両方でルームに届き、通知は 1 回だけである
2. firing → resolved → firing の再発が 3 回とも通知される
3. Keep のタスクを全停止しても critical がルームに届く
4. Keep を停止した状態で非 critical を流し、Keep 復旧後に自動で届く
5. 同一アラートを ingress に 2 回投入しても通知は 1 回である

**MVP に含めないもの**: Reconciler、2 種のハートビート、DLQ アラームと独立 SNS 経路、GameDay、大阪スタック、Route 53 フェイルオーバー、RDS クロスリージョンレプリカ

### 9.2 フェーズ 2: Go 条件の充足（東京のみ）
- Reconciler、2 種のハートビート、DLQ アラーム、独立 SNS 経路
- Keep 停止・Redis 停止・RDS フェイルオーバー・AZ 障害の GameDay
- 完了時点で「PagerDuty と並走したまま内製経路を本番扱い」にできる。単独運用への切り替えは不可

### 9.3 フェーズ 3: DR（大阪）
- 大阪スタック、RDS クロスリージョンレプリカ、ECR / Secrets 複製、大阪からの canary、Route 53 フェイルオーバー、Step Functions 自動化
- 東京隔離の GameDay
- **PagerDuty 解約はこのフェーズ完了後**

規模感の目安: 各フェーズ 3〜4 週間（1〜2 名）。実装者の経験とコミュニケーションツール API の仕様に依存する。

### 9.4 フェーズ 1 で決めておく「後から変えない」前提

| 項目 | 理由 |
|---|---|
| IaC をリージョンをパラメータ化して書く | 大阪展開が同じスタックの再適用で済む |
| 監視ソースはパブリックホストゾーンの DNS 名を叩く | 後でフェイルオーバーレコードに差し替えるだけ。ソース側の設定変更を 2 回やらない |
| event_id / transition_id を最初から採番する | 冪等性と DR の二重書き吸収の土台。後付けは全経路の改修 |
| Journal は DynamoDB Streams を有効にして作る | グローバルテーブル化の前提 |
| Keep のイメージは ECR にミラーする | クロスリージョン複製を後で足せる |
| Keep のワークフロー・プロバイダ設定は Git 管理し、UI で手編集しない | 大阪に同じ設定を適用する手段 |
| 秘密情報は Secrets Manager に置く | マルチリージョン複製を後で足せる |
| キュー・テーブル名にリージョンを含めない | 両リージョンで同じ論理名 |
| NAT は最初から Regional NAT Gateway | ゾーン型からの移行はルートテーブル変更を伴う |
| SaaS ごとの秘密を Secrets Manager で管理し Lambda オーソライザで検証 | 公開受信口の防御を最初から組み込む |

### 9.5 フェーズ 3 まで安全に後回しにできるもの
大阪スタック、RDS クロスリージョンレプリカ、Secrets / ECR 複製、大阪からの外形監視、Route 53 フェイルオーバーレコード、Step Functions 自動化、大阪レプリカの Multi-AZ 化

---

## 10. DR 設計（フェーズ 3 で実装）

### 10.1 大阪に置くもの
常時稼働: API Gateway、ingress.standard、Normalize Lambda、alerts.fifo、keep-delivery.fifo、内製ツール Lambda、Synthetics canary。常時複製: Journal（グローバルテーブル）。待機: Keep Fargate（desired 0）、RDS リードレプリカ、ECR / Secrets 複製。事前作成: ALB、Regional NAT。不要: Redis。

### 10.2 フェイルオーバー
- 入口: Route 53 フェイルオーバー（TTL 60 秒）。大阪 canary の結果を CloudWatch アラーム化したヘルスチェックで切り替え
- critical: 切り替え後、大阪の経路で Keep を待たず到達
- Keep: RDS 昇格 → Keep desired 2 → amazonsqs 向き先を大阪へ。Step Functions で自動化
- 内製ツール: 各リージョンの Lambda が各リージョンの alerts.fifo を消費するため、出力側の操作は不要

### 10.3 RTO / RPO（目標値。GameDay で実測して SLO 化）

| 対象 | 表現 |
|---|---|
| critical（東京障害後に新規受信） | RTO 1〜数分を目標 |
| critical（東京で受理済みのバックログ） | 東京復旧まで利用不能となり得る。RPO 0 ではない |
| Keep の経路 | RTO 15〜30 分を目標 |
| Keep の DB | 通常時 ReplicaLag 数秒を目標。実 RPO は障害時点の ReplicaLag に依存 |

critical の RPO を 0 に近づける dual-ingress（東京・大阪の両 API Gateway へ送信）は P1。東京 EKS 上の Alertmanager は東京と共に消えるため、dual-send できるのは東京 EKS 外のソースに限られる。「東京が丸ごと消えた」ことは大阪の canary が検知し、大阪 ingress に critical を投入する正規のソースとして扱う。

### 10.4 フェイルバック
東京側の Keep は desired 0 のまま起動させない（別 DB での同時稼働を防ぐ）。Keep の DB を東京へ戻す場合は大阪をプライマリとして東京にレプリカを作り直し、計画停止で逆方向に昇格する。

---

## 11. コスト（1 ドル=150 円換算、東京オンデマンド x86）

| リージョン | 月額(USD) |
|---|---|
| 東京（Fargate 126、RDS 137、Redis 60、ALB 25、Regional NAT 90、Journal 3〜5、その他 13） | 約 455〜460 |
| 大阪（RDS レプリカ 72、ALB 20、Regional NAT 45〜90、canary 10〜15、複製・転送 10） | 約 160〜210 |
| **合計** | **約 600〜700（約 9〜10.5 万円）、年約 108〜126 万円** |

フェーズ 1〜2 は東京のみのため月約 455〜460 ドル（年約 82 万円）。

| 選択肢 | 年額 | 現状比 |
|---|---|---|
| 現状 PagerDuty | 200 万円 | — |
| v3 構成 + DR | 約 108〜126 万円 | −74〜92 万円 |
| 参考: PagerDuty Business 定価 | 約 148 万円 | −52 万円 |
| 参考: PagerDuty Professional 定価 | 約 76 万円 | −124 万円 |

含まれないもの: コミュニケーションツールのホスティング費、構築工数、運用工数、並走期間の PagerDuty 費、別ツールの SMS / 音声 API 従量費。

---

## 12. 未確認事項

### 責務分担
1. オンコール判定・エスカレーション・電話発報をどこで行うか
2. 「ダウンタイムなしで転送」はルームへの投稿で満たされるか、電話・プッシュ通知まで含むか
3. 「critical」の定義（Normalize Lambda が各ソースからどう判定するか）
4. コミュニケーションツール API の冪等性（同じ投稿を 2 回受けたときに重複表示されないキーを持てるか）

### Keep
5. Keep API `/alerts/event/{provider}` の認証とレスポンスコード
6. Keep が transition_id / event_id をアラート属性として保持し、ワークフローから参照できるか
7. Keep の重複排除ルールが同一 fingerprint の再発を抑止しないか
8. Dispatcher Lambda → Keep API の経路（ALB を内部向けにする場合の VPC 配置）
9. amazonsqs アクションの向き先をリージョン間で切り替える手順
10. Keep の arm64 イメージ提供有無
11. 外部レビューの「Keep OSS は community-maintained フェーズ」という記述の出典

### DR・周辺
12. 内製コミュニケーションツールが東京リージョン障害の影響を受けないか
13. 東京 EKS 外の監視ソースで大阪エンドポイントへの dual-send が可能か

### 契約
14. 現在の PagerDuty 契約の内訳（交渉のみで下がる余地）
15. PagerDuty の途中解約条件と並走期間の二重コスト

---

## 13. 本番 Go 条件（外部レビューの 6 条件）

1. Keep の 202 を耐久性の境界にしない（keep-delivery.fifo + Journal + Reconciler）
2. fingerprint+status を廃止し event_id / transition_id を導入。両経路で同一 ID
3. 「critical RPO 0」を削除。必要なら dual-ingress
4. 大阪から入口→出口の合成監視を行う
5. ゾーン型 NAT 1 台を使わず Regional NAT Gateway
6. DR GameDay を PagerDuty 解約条件にする（Keep 停止、Redis 停止、RDS フェイルオーバー、AZ 障害、東京リージョン隔離を実際に起こし RTO / RPO を測定）

---

## 14. 最終裁定

1. **アーキテクチャは案 C（v3）を採用し、東京 MVP → フェーズ 2（Go 条件充足）→ 大阪 DR の 3 段階で実装する。** 6 つの Go 条件を満たすまで PagerDuty は解約しない
2. **Keep は再実行可能な処理レイヤーとして扱う。** 失ってはいけない記録は Keep の外側に置き、Keep の高可用化より外側の耐久性を優先する
3. **critical は Keep を通さない直送を常時行う。** Normalize Lambda の成功条件は Keep の成否に依存しない
4. **冪等性は transition_id と Journal で担保する。** SQS FIFO の 5 分窓に依存しない
5. **内製ツールは alerts.fifo に付けた Lambda として新規構築する。** 責務はルーティングと配送に限定し、冪等ストアは Journal を共用する
6. **受信口はパブリックホストゾーンに置き、Lambda オーソライザ・WAF・スロットリングで防御する**
7. **DR はパイロットライトとし、RTO / RPO は目標値として GameDay で実測する**
8. **フェーズ 1 で「後から変えない」10 項目を守る。** これにより大阪展開は純粋な追加作業になる
9. **コスト面の判断。** 年約 74〜92 万円の削減は成立する。現契約の内訳を先に確認し、交渉後も差が残るなら移行を進める

---

## 付録: コスト単価の前提

- Fargate（東京、x86）: 0.05056 ドル/vCPU 時、0.00553 ドル/GB 時
- 稼働時間: 730 時間/月、為替: 1 ドル=150 円
- Regional NAT Gateway は AZ 展開数に依存するため 2AZ 相当で概算
- 単価は 2026 年 9 月時点の把握値。確定時は AWS Pricing Calculator で再確認すること
