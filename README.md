# trendops-runtime

## README・設計書・Runbook自動更新Agent

---

## 1. プロダクト名案

### DocOps Agent

**コード変更に追従して、ドキュメントの陳腐化を防ぐAIエージェント**

別名案：

* Docsync Agent
* Runbook Keeper
* AutoDocOps
* Living Docs Agent
* Driftless Docs Agent

ハッカソン向けには、**DocOps Agent** が一番わかりやすいです。
DevOps文脈にも自然に乗ります。

---

# 2. 背景・課題

ソフトウェア開発では、コードやインフラは頻繁に変更される一方で、README、API仕様書、設計書、運用Runbookなどのドキュメントは更新されず、実態とズレていく。

このズレにより、以下の問題が発生する。

* 新規メンバーが正しく環境構築できない
* API仕様と実装が一致しない
* 障害対応時にRunbookが古く、対応が遅れる
* インフラ構成変更が設計書に反映されない
* レビュー時にドキュメント更新漏れを人間が確認する必要がある
* 本番運用に必要な知識がコード変更に追従しない

特にDevOpsでは、コード、CI/CD、インフラ、運用手順が一体として改善され続ける必要がある。
そのため、**ドキュメントも継続的に更新される仕組み**が求められる。

---

# 3. 目的

DocOps Agentは、GitHub上のコード変更・インフラ変更・API変更を検知し、関連ドキュメントの不整合をAIが判断して、自動で修正PRを作成する。

これにより、ドキュメントを「人間が頑張って更新するもの」から、**開発フローの中で自動的に保守されるもの**へ変える。

---

# 4. コンセプト

## つくる

Geminiを活用して、PR差分・コード・設定ファイル・既存ドキュメントを読み取り、更新が必要な箇所を判断するAIエージェントを実装する。

## まわす

GitHub WebhookやGitHub Actionsと連携し、PR作成時・マージ時・CI実行時にドキュメント更新チェックを自動実行する。

## とどける

Cloud Run上にAgent APIと可視化UIをデプロイし、ユーザーが実際に利用できる形で提供する。

---

# 5. 対象ユーザー

## メインユーザー

* ソフトウェアエンジニア
* SRE / Platform Engineer
* DevOps Engineer
* Tech Lead
* 開発チームのドキュメント管理担当者

## 想定利用シーン

* API仕様を変更したとき
* 環境変数を追加・変更したとき
* TerraformやCloud Run設定を変更したとき
* デプロイ手順や障害対応手順が変わったとき
* READMEのセットアップ手順が古くなったとき
* PRレビュー時にドキュメント更新漏れを検出したいとき

---

# 6. 解決したい課題

## 課題1：READMEが古くなる

環境変数、起動コマンド、依存関係、セットアップ手順がコード変更に追従しない。

## 課題2：API仕様書と実装がズレる

実装ではレスポンス項目が追加・削除されているのに、OpenAPIやREADME上のAPI説明が更新されていない。

## 課題3：Runbookが運用実態とズレる

障害対応手順、ログ確認方法、ロールバック方法、アラート条件が古いまま残る。

## 課題4：ドキュメント更新漏れを人間が見つける必要がある

レビュー担当者がコードだけでなくドキュメント整合性まで確認する必要があり、負荷が高い。

## 課題5：開発速度が上がるほどドキュメント品質が下がる

継続的デリバリーが進むほど、ドキュメント更新が追いつかなくなる。

---

# 7. 提供価値

DocOps Agentは以下の価値を提供する。

1. コード変更に応じて更新すべきドキュメントを自動検出する
2. README、API仕様書、Runbookの更新案を自動生成する
3. 修正内容をGitHub PRとして作成する
4. なぜ更新が必要かを説明する
5. ドキュメント更新漏れをCI/CDの一部として検出する
6. 運用ドキュメントの鮮度を保つ

---

# 8. MVPスコープ

ハッカソンでは、以下をMVPとして実装する。

## MVPで必ず作るもの

### 1. GitHub PR差分取得

GitHubのPR差分を取得し、変更されたファイルを解析する。

対象例：

* API実装ファイル
* 環境変数設定ファイル
* Dockerfile
* Cloud Run設定ファイル
* Terraformファイル
* package.json / requirements.txt
* README.md
* docs配下のMarkdownファイル

---

### 2. ドキュメント更新要否判定

Geminiを使い、PR差分と既存ドキュメントを比較して、更新が必要かどうかを判断する。

判定対象：

* READMEのセットアップ手順
* 環境変数一覧
* API仕様
* デプロイ手順
* Runbook
* トラブルシューティング手順

---

### 3. 更新案生成

Geminiを使って、対象ドキュメントの修正案を生成する。

出力形式：

* Markdown差分
* OpenAPI YAML差分
* Runbook修正案
* 変更理由の説明

---

### 4. GitHub PR作成

Agentが自動でドキュメント更新PRを作成する。

PRには以下を含める。

* 更新されたドキュメントファイル
* 変更理由
* 参照したコード差分
* Agentの判断ログ
* 人間レビューが必要な注意点

---

### 5. 判断プロセスの可視化UI

Cloud Run上に簡単なWeb UIを用意し、Agentが何を見て、どう判断し、何を更新したかを可視化する。

表示内容：

* 対象PR
* 変更ファイル
* 影響を受けるドキュメント
* 更新要否
* 生成された修正案
* 作成されたPRリンク
* Agentの実行ログ

---

# 9. MVP外スコープ

ハッカソン期間中は、以下は必須にしない。

* 完全な自然言語ドキュメント検索
* 複数リポジトリ横断分析
* Slack / Teams連携
* Confluence / Notion連携
* 高度なRAG基盤
* すべての言語・フレームワーク対応
* 完全自動マージ
* 本番環境への直接変更
* 複雑な権限管理UI
* 高度なセマンティックコード解析

ただし、余裕があれば加点要素として実装する。

---

# 10. 機能要件

## FR-001：GitHub連携

### 概要

DocOps AgentはGitHubリポジトリと連携し、PRやコミットの情報を取得できる。

### 詳細要件

* GitHub Webhookを受け取れること
* PR作成・更新イベントを検知できること
* PR差分を取得できること
* 変更ファイル一覧を取得できること
* 対象ブランチに対して新しいPRを作成できること
* PRコメントを投稿できること

### 受け入れ条件

* GitHub上でPRを作成すると、Agentが自動で起動する
* Agentが変更ファイル一覧を取得できる
* Agentがドキュメント更新PRを作成できる

---

## FR-002：変更差分解析

### 概要

AgentはPR差分を解析し、ドキュメント更新に影響する変更かどうかを判断する。

### 対象変更

* APIエンドポイントの追加・変更・削除
* レスポンス形式の変更
* リクエストパラメータの変更
* 環境変数の追加・削除・名称変更
* Dockerfileの変更
* Cloud Run設定の変更
* Terraformの変更
* デプロイコマンドの変更
* エラーハンドリングの変更
* ログ出力形式の変更

### 受け入れ条件

* APIレスポンス変更を検知できる
* 環境変数追加を検知できる
* Cloud Run設定変更を検知できる
* ドキュメント更新が不要な変更は不要と判断できる

---

## FR-003：関連ドキュメント特定

### 概要

Agentは変更内容に応じて、更新すべきドキュメントを特定する。

### 対象ドキュメント

* README.md
* docs/api.md
* docs/runbook.md
* docs/deployment.md
* openapi.yaml
* architecture.md
* troubleshooting.md

### 判定例

| 変更内容          | 更新対象                            |
| ------------- | ------------------------------- |
| 環境変数追加        | README、deployment.md、runbook.md |
| APIレスポンス変更    | openapi.yaml、docs/api.md        |
| Cloud Run設定変更 | deployment.md、runbook.md        |
| エラーログ形式変更     | troubleshooting.md、runbook.md   |
| ヘルスチェック変更     | README、runbook.md               |

### 受け入れ条件

* 変更内容に応じて関連ドキュメント候補を提示できる
* 更新対象と判断した理由を説明できる

---

## FR-004：不整合検出

### 概要

Agentはコード変更と既存ドキュメントを比較し、ドキュメントの不整合を検出する。

### 不整合例

* READMEに存在しない環境変数がコードに追加されている
* OpenAPIに存在しないレスポンス項目が実装に追加されている
* Runbookに古いログ確認コマンドが残っている
* デプロイ手順が現在のCloud Run設定と一致していない
* 依存関係のインストール手順が古い

### 受け入れ条件

* 不整合の有無を判定できる
* 不整合箇所をファイル名・行番号またはセクション名付きで提示できる
* 不整合の理由を自然言語で説明できる

---

## FR-005：ドキュメント更新案生成

### 概要

Agentは検出した不整合に対して、更新後のドキュメント案を生成する。

### 詳細要件

* Markdownを生成できる
* OpenAPI YAMLを更新できる
* 既存の文体・構成をなるべく維持できる
* 変更理由を説明できる
* 不明点がある場合はTODOコメントを残せる
* 過剰な書き換えを避け、必要最小限の差分にできる

### 受け入れ条件

* READMEの環境変数一覧を自動更新できる
* API仕様書のレスポンス例を自動更新できる
* Runbookの手順を自動更新できる
* 既存ドキュメント全体を破壊しない

---

## FR-006：ドキュメント更新PR作成

### 概要

Agentは生成した更新案をGitHub上でPRとして作成する。

### PR内容

* タイトル
* 概要
* 更新対象ファイル
* 更新理由
* 参照した元PR
* Agentの判断ログ
* 人間レビューが必要な点

### PRタイトル例

```text
docs: update README and runbook for new ORDER_API_KEY
```

### PR本文例

```markdown
## Summary
This PR updates documentation based on changes detected in #123.

## Detected Changes
- Added environment variable: ORDER_API_KEY
- Updated order API response schema

## Updated Docs
- README.md
- docs/api.md
- docs/runbook.md

## Agent Reasoning
The source PR introduced a new required environment variable, but README.md did not include it in the setup instructions.

## Review Notes
Please confirm whether ORDER_API_KEY is required in all environments.
```

### 受け入れ条件

* Agentがdocs更新用PRを作成できる
* PR本文に判断理由が含まれる
* 元PRへのリンクが含まれる
* 人間がレビューできる状態になっている

---

## FR-007：PRコメント通知

### 概要

Agentは元PRに対して、ドキュメント更新が必要かどうかをコメントする。

### コメント例

```markdown
DocOps Agent checked this PR.

Documentation update required:
- README.md: new environment variable `ORDER_API_KEY`
- docs/api.md: response schema changed
- docs/runbook.md: deployment checklist should be updated

Created docs update PR: #124
```

### 受け入れ条件

* 元PRにコメントを投稿できる
* 更新不要の場合も「更新不要」と通知できる
* 更新PRのリンクを提示できる

---

## FR-008：判断ログ保存

### 概要

Agentは実行ごとに判断ログを保存する。

### 保存内容

* 実行ID
* 対象リポジトリ
* 対象PR
* 変更ファイル一覧
* 参照したドキュメント
* Geminiへの入力概要
* Geminiの出力概要
* 更新要否
* 作成したPR
* エラー内容

### 受け入れ条件

* 実行履歴を後から確認できる
* UI上で判断ログを表示できる
* 失敗時に原因を確認できる

---

## FR-009：Web UI

### 概要

Agentの動作を可視化するWeb UIを提供する。

### 画面

#### ダッシュボード画面

* 実行履歴一覧
* 対象PR
* ステータス
* 更新要否
* 作成PRリンク

#### 詳細画面

* 変更ファイル
* 検出した不整合
* 更新対象ドキュメント
* 生成した差分
* Agentの判断ステップ
* エラー内容

### 受け入れ条件

* Cloud Run上でUIにアクセスできる
* Agentの処理状況を確認できる
* デモ時に「差分検出 → 判断 → 修正PR作成」の流れが見える

---

## FR-010：手動実行

### 概要

ユーザーは任意のPR番号を指定して、Agentを手動実行できる。

### 受け入れ条件

* UIまたはAPIからPR番号を指定して実行できる
* Webhookなしでもデモできる
* 同じPRに対して再実行できる

---

# 11. 非機能要件

## NFR-001：デプロイ環境

* Agent APIはCloud Runにデプロイする
* Web UIもCloud Runで提供する
* コンテナ化されていること
* GitHub ActionsまたはCloud BuildでCI/CDを構築する

---

## NFR-002：AI技術

* GeminiまたはVertex AI Geminiを利用する
* PR差分、既存ドキュメント、更新判断にGeminiを活用する
* 出力はJSON形式でも取得できるようにする
* プロンプトはバージョン管理する

---

## NFR-003：安全性

* Agentが本番環境を直接変更しない
* Agentが作成するのはPRまでとする
* マージは人間が行う
* GitHub Tokenの権限は最小限にする
* Secret Managerで機密情報を管理する
* Agentの判断理由を記録する

---

## NFR-004：信頼性

* Agentの処理が失敗しても元PRに影響を与えない
* 同じPRに対して再実行できる
* 生成差分が不正な場合はPRを作成せず、エラーとして記録する
* GitHub APIエラー時はリトライする

---

## NFR-005：可観測性

* Cloud Loggingに実行ログを出力する
* 実行ステータスを保存する
* エラー内容をUIで確認できる
* Agentの判断ステップを追跡できる

---

## NFR-006：保守性

* 対応ドキュメント種別を追加しやすい設計にする
* プロンプトをコードから分離する
* ルールベース判定とAI判定を分離する
* GitHub連携部分とGemini連携部分を分離する

---

## NFR-007：パフォーマンス

MVPでは以下を目標とする。

* 1 PRあたりの解析時間：1〜3分以内
* 小規模リポジトリでの差分解析：30秒以内
* UI表示：3秒以内
* PR作成：解析完了後30秒以内

---

# 12. システム構成

```text
GitHub
  ├─ Pull Request
  ├─ Webhook
  └─ Docs Update PR
        ↑
        |
Cloud Run
  ├─ DocOps Agent API
  ├─ Web UI
  └─ Worker
        |
        ├─ GitHub API
        ├─ Gemini / Vertex AI
        ├─ Firestore
        ├─ Secret Manager
        └─ Cloud Logging
```

---

# 13. 処理フロー

## 自動実行フロー

```text
1. 開発者がGitHubでPRを作成
2. GitHub WebhookがCloud RunのAgent APIを呼び出す
3. AgentがPR差分を取得
4. Agentが変更ファイルを分類
5. Agentが関連ドキュメントを取得
6. Geminiが不整合を判定
7. Geminiが修正案を生成
8. Agentがdocs更新ブランチを作成
9. Agentが修正ファイルをコミット
10. AgentがGitHub PRを作成
11. 元PRにコメントを投稿
12. UIに実行結果を表示
```

---

## 手動実行フロー

```text
1. ユーザーがUIでリポジトリ名とPR番号を入力
2. Agentが対象PRを取得
3. 差分とドキュメントを解析
4. 更新要否を判定
5. 更新PRを作成
6. 結果をUIに表示
```

---

# 14. AI Agentの判断ステップ

Agentは以下のように段階的に判断する。

```text
Step 1: PR差分を取得
Step 2: 変更ファイルを分類
Step 3: ドキュメント影響がありそうな変更を抽出
Step 4: 関連ドキュメントを検索
Step 5: コード変更とドキュメント内容を比較
Step 6: 不整合を検出
Step 7: 更新案を作成
Step 8: 更新の確信度を算出
Step 9: PRを作成するか判断
Step 10: GitHubにPRまたはコメントを作成
```

---

# 15. Agentの判断ルール

## ルールベースで検出するもの

AIだけに任せず、以下はルールベースで検出する。

### 環境変数

コード内で以下のような記述を検出する。

```js
process.env.ORDER_API_KEY
```

```python
os.environ["ORDER_API_KEY"]
```

検出した環境変数がREADMEに存在しなければ、更新候補にする。

---

### APIエンドポイント

以下のような変更を検出する。

```text
GET /orders
POST /orders
PATCH /orders/{id}
```

新規追加やレスポンス変更があれば、API仕様書の更新候補にする。

---

### インフラ設定

以下の変更を検出する。

* Dockerfile
* cloudbuild.yaml
* service.yaml
* terraform/*.tf
* .github/workflows/*.yml

変更があれば、deployment.mdやrunbook.mdの更新候補にする。

---

## Geminiで判断するもの

以下はGeminiに判断させる。

* この変更がドキュメント更新を必要とするか
* どのドキュメントが影響を受けるか
* 既存ドキュメントのどの記述が古いか
* どのように書き換えるべきか
* 人間に確認すべき点は何か

---

# 16. 入出力仕様

## 入力

### GitHub PR情報

```json
{
  "repository": "example/app",
  "pull_request_number": 123,
  "base_branch": "main",
  "head_branch": "feature/add-order-api",
  "changed_files": [
    "src/routes/orders.ts",
    "src/config/env.ts",
    "README.md"
  ]
}
```

---

## Agentの中間出力

```json
{
  "requires_doc_update": true,
  "confidence": 0.87,
  "detected_changes": [
    {
      "type": "environment_variable",
      "name": "ORDER_API_KEY",
      "action": "added",
      "source_file": "src/config/env.ts"
    },
    {
      "type": "api_response_change",
      "endpoint": "GET /orders",
      "action": "modified",
      "source_file": "src/routes/orders.ts"
    }
  ],
  "affected_documents": [
    {
      "path": "README.md",
      "reason": "New required environment variable is not documented."
    },
    {
      "path": "docs/api.md",
      "reason": "Order API response schema changed."
    }
  ]
}
```

---

## 出力PR

```json
{
  "title": "docs: update docs for order API changes",
  "body": "Documentation updated based on PR #123.",
  "files": [
    "README.md",
    "docs/api.md"
  ],
  "source_pr": 123
}
```

---

# 17. 画面要件

## 画面1：実行履歴一覧

### 表示項目

* 実行ID
* リポジトリ名
* PR番号
* ステータス
* 更新要否
* 作成PR
* 実行日時

### ステータス

* Pending
* Running
* No Update Needed
* Update PR Created
* Failed

---

## 画面2：実行詳細

### 表示項目

* 対象PR情報
* 変更ファイル一覧
* Agentの判断ステップ
* 検出された変更
* 不整合の内容
* 更新対象ドキュメント
* 生成された差分
* 作成されたPRリンク

---

## 画面3：手動実行画面

### 入力項目

* GitHub owner
* repository
* pull request number

### ボタン

* Analyze
* Create docs PR
* Re-run

---

# 18. データモデル

## executions

```json
{
  "id": "exec_001",
  "repository": "example/app",
  "pull_request_number": 123,
  "status": "update_pr_created",
  "requires_doc_update": true,
  "created_pr_number": 124,
  "created_at": "2026-06-13T10:00:00Z",
  "updated_at": "2026-06-13T10:02:00Z"
}
```

---

## detected_changes

```json
{
  "execution_id": "exec_001",
  "type": "environment_variable",
  "name": "ORDER_API_KEY",
  "action": "added",
  "source_file": "src/config/env.ts",
  "reason": "New environment variable added in application config."
}
```

---

## affected_documents

```json
{
  "execution_id": "exec_001",
  "path": "README.md",
  "reason": "Environment variable list is outdated.",
  "update_required": true
}
```

---

## agent_logs

```json
{
  "execution_id": "exec_001",
  "step": "detect_doc_drift",
  "message": "README.md does not mention ORDER_API_KEY.",
  "timestamp": "2026-06-13T10:01:00Z"
}
```

---

# 19. API要件

## POST /webhook/github

GitHub Webhookを受け取る。

### Request

GitHub Webhook payload

### Response

```json
{
  "status": "accepted",
  "execution_id": "exec_001"
}
```

---

## POST /analyze

手動でPR解析を実行する。

### Request

```json
{
  "owner": "example",
  "repo": "app",
  "pull_request_number": 123
}
```

### Response

```json
{
  "execution_id": "exec_001",
  "status": "running"
}
```

---

## GET /executions

実行履歴を取得する。

### Response

```json
[
  {
    "execution_id": "exec_001",
    "repository": "example/app",
    "pull_request_number": 123,
    "status": "update_pr_created"
  }
]
```

---

## GET /executions/{id}

実行詳細を取得する。

### Response

```json
{
  "execution_id": "exec_001",
  "status": "update_pr_created",
  "detected_changes": [],
  "affected_documents": [],
  "agent_logs": []
}
```

---

## POST /executions/{id}/create-pr

解析結果をもとにdocs更新PRを作成する。

### Response

```json
{
  "status": "created",
  "pull_request_url": "https://github.com/example/app/pull/124"
}
```

---

# 20. 技術要件

## 必須技術

* Google Cloud
* Gemini / Vertex AI Gemini
* Cloud Run
* GitHub API
* GitHub ActionsまたはCloud Build
* Docker

## 推奨構成

### Backend

* Python FastAPI
  または
* Node.js / TypeScript Express

### Frontend

* Next.js
* React
* Tailwind CSS

### Storage

* Firestore

### Secrets

* Secret Manager

### Logs

* Cloud Logging

### CI/CD

* GitHub Actions
* Cloud Build

---

# 21. セキュリティ要件

## GitHub Token

* Secret Managerに保存する
* 必要最小限の権限にする
* public repoの場合も書き込み権限は限定する

## Webhook

* GitHub Webhook Secretで署名検証を行う
* 未検証のWebhookは拒否する

## Agentの実行権限

* 本番ブランチへ直接pushしない
* 必ずdocs更新用ブランチを作る
* 必ずPR経由にする
* 自動マージはMVPでは行わない

## Geminiへの入力

* シークレット値をそのまま送らない
* `.env` や秘密情報をマスクする
* PR差分のうち必要な情報だけを渡す

---

# 22. デモ要件

## デモで見せるべき流れ

```text
1. APIレスポンス仕様を変更するPRを作成
2. AgentがPR差分を検知
3. README / docs/api.md / runbook.md の不整合を検出
4. UIでAgentの判断ログを表示
5. Agentがdocs更新PRを作成
6. 元PRにコメントが投稿される
7. docs更新PRの差分をGitHubで確認
```

---

# 23. デモシナリオ案

## シナリオ：注文APIのレスポンス変更

### 変更前

`GET /orders/{id}` のレスポンス：

```json
{
  "id": "order_001",
  "status": "paid",
  "total": 1200
}
```

### 変更後

```json
{
  "id": "order_001",
  "status": "paid",
  "total": 1200,
  "estimated_delivery_date": "2026-08-20"
}
```

### 発生するドキュメント不整合

* `docs/api.md` に `estimated_delivery_date` がない
* `openapi.yaml` に `estimated_delivery_date` がない
* `README.md` のサンプルレスポンスが古い

### Agentの判断

```text
GET /orders/{id} のレスポンスに estimated_delivery_date が追加されています。
しかし docs/api.md と openapi.yaml にはこの項目が記載されていません。
API利用者に影響があるため、ドキュメント更新が必要です。
```

### Agentが作るPR

```text
docs: update order API response documentation
```

更新ファイル：

* `docs/api.md`
* `openapi.yaml`
* `README.md`

---

# 24. もう一つのデモシナリオ案

## シナリオ：環境変数追加

### コード変更

```ts
const timeout = process.env.ORDER_API_TIMEOUT || "3000";
```

### 既存README

```markdown
## Environment Variables

| Name | Description |
|---|---|
| ORDER_API_KEY | API key for order service |
```

### Agentの判断

```text
ORDER_API_TIMEOUT がコードに追加されていますが、README.md の環境変数一覧に存在しません。
ローカル開発・Cloud Runデプロイ時に必要な設定である可能性があるため、README.md と docs/deployment.md の更新が必要です。
```

### Agentが作る差分

```markdown
| ORDER_API_TIMEOUT | Timeout value for order API requests in milliseconds. Default: 3000 |
```

### デモ映えポイント

このシナリオは非常にわかりやすいです。
短時間で作るなら、**環境変数追加の検知**を中心にすると成功しやすいです。

---

# 25. 受け入れ基準

MVP完成の判定基準は以下。

## Must

* GitHub PR差分を取得できる
* READMEまたはdocs配下のファイルを取得できる
* Geminiで更新要否を判定できる
* Markdownの更新案を生成できる
* GitHubにdocs更新PRを作成できる
* Cloud Run上でAgent APIが動く
* UIでAgentの処理結果を確認できる

## Should

* 元PRにコメントできる
* Agentの判断ログを表示できる
* OpenAPI YAMLを更新できる
* 実行履歴をFirestoreに保存できる
* GitHub ActionsでCI/CDできる

## Could

* Slack通知できる
* Runbookの自動更新に対応できる
* Terraform変更から設計書更新を提案できる
* 類似PRから過去のドキュメント更新例を参照できる
* 更新PRの品質を自己レビューできる

---

# 26. ハッカソン向け優先順位

## Day 1で作る

* サンプルリポジトリ
* GitHub PR差分取得
* README読み込み
* Geminiによる更新要否判定
* Markdown更新案生成

## Day 2で作る

* GitHub PR作成
* Cloud Runデプロイ
* 簡易UI
* Agent実行ログ保存
* デモ用シナリオ整備

## 余裕があれば作る

* OpenAPI更新
* Runbook更新
* Slack通知
* Agent判断ログのリッチ表示
* before / after比較UI

---

# 27. ピッチ用の一言

> コードは毎日変わるのに、ドキュメントは置き去りになる。DocOps Agentは、PR差分を読み、古くなったREADME・API仕様書・Runbookを検出し、修正PRまで自動で作るAIエージェントです。

---

# 28. 審査で強調するポイント

このアイデアは派手なチャットボットではなく、**開発プロセスそのものを改善するAI Agent**である点を強調すると良いです。

特に以下を見せると刺さります。

* GitHub PRをトリガーにAgentが自律的に動く
* コード差分からドキュメント影響を判断する
* Geminiが更新理由を説明する
* Cloud Runで本番運用可能な形にしている
* docs更新PRとして人間がレビューできる
* DevOpsの継続改善サイクルに組み込める

---

# 29. 最小構成の実装対象

ハッカソンで確実に完成させるなら、まずはこれに絞るのがおすすめです。

## 対象

* README.md
* docs/api.md
* docs/runbook.md

## 検出対象

* 環境変数追加
* APIレスポンス変更
* デプロイ手順変更

## Agentのアクション

* PR差分取得
* 更新要否判定
* Markdown更新
* docs更新PR作成
* UIで判断ログ表示

---

# 30. 最終要件サマリー

## 作るもの

**PR差分を読み取り、古くなったドキュメントを検出し、自動で修正PRを作るAIエージェント**

## 主な機能

* GitHub PR監視
* PR差分解析
* README / API仕様 / Runbookの不整合検出
* Geminiによる更新案生成
* GitHub docs更新PR作成
* Agent判断ログの可視化
* Cloud Runデプロイ

## 技術構成

* Gemini / Vertex AI
* Cloud Run
* GitHub API
* GitHub Actions
* Firestore
* Secret Manager
* Cloud Logging

## MVPのゴール

```text
APIや環境変数の変更PRを作る
↓
DocOps Agentが差分を解析
↓
古いREADMEやAPI仕様を検出
↓
Geminiが修正案を生成
↓
Agentがdocs更新PRを作成
↓
UIで判断プロセスを確認できる
```

## 成功条件

* デモで一連の流れが5分以内に見せられる
* Agentが自律的に判断していることが伝わる
* DevOpsの継続改善に組み込めることが伝わる
* 実務で使いたいと思える完成度になっている
