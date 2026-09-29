# C-MAC AWSサンドボックス共通運用ランブック作成仕様書

## 1. このファイルの目的

このファイルは、Claude Codeに「C-MAC AWSサンドボックス共通運用ランブック」一式を生成させるための入力仕様書である。

Claude Codeは本仕様書を唯一の要件定義として読み取り、既存の断片的なMarkdownを前提にせず、最終的なフォルダー構成、Markdown文書、Terraform、PowerShell、Bash、設定例を一括生成すること。

## 2. 成果物の目的

C-MACのAWSサンドボックス環境で、学習または技術検証の用途に応じたAWS環境を、安全かつ短時間で構築、確認、停止、削除、再構築できる共通運用ランブックを作成する。

本成果物はAWSの学習教材ではない。AWS用語の長い解説は不要であり、実際に使える設計判断、構築手順、IaC、確認方法、削除手順、トラブルシューティングを重視する。

## 3. 利用者と用途

- 利用者は当面1名のみ。
- 個人の学習および非業務の技術検証に利用する。
- 業務ソースコード、社内機密情報、顧客情報、個人情報、本番データは扱わない。
- 個人契約のClaude Codeを利用する。
- 個人Public GitHubアカウントを利用できる。
- Public GitHubには一般公開可能な汎用コードとダミー情報のみ保存する。
- C-MAC固有情報はPublic GitHubへ保存しない。

## 4. 利用者の技術スタック

- ReactフレームワークのWeb開発: ほぼ未経験
- Claude Code開発: 初級
- Python: 初級
- DB設計: 初級
- AWS: 初級
- Snowflake: 中級
- Docker: Webアプリ開発の標準要素として利用
- GitおよびGitHubを利用
- VS Codeを利用
- APIはFastAPIを優先
- IaCはTerraformを採用

## 5. C-MACサンドボックスの前提

### 5.1 環境

- Sharedアカウントからサンドボックス個別アカウントへスイッチロールする。
- サンドボックス用ロール名は `CP-SandD01-D-Role-ChildAdmin`。
- 利用リージョンは `ap-northeast-1` のみ。
- 非商用の学習・検証に限る。
- オンプレミスネットワークへ接続できない。

### 5.2 定期削除

- 毎週日曜日22:00から翌09:00までサインインできない。
- ほぼすべてのAWSリソースが削除される。
- S3も週末削除の対象である。
- AWS内のリソースやデータを永続的な正本として扱わない。
- 週末前に原則としてTerraform管理対象を `terraform destroy` する。
- 週末後は空の環境から再構築する。

### 5.3 制約

- VPC、Subnet、Network ACLは利用者が作成できない。
- 一部のAWSサービスは利用できない。
- IAMユーザー、IAMグループ、アクセスキーは利用者が作成できない。
- サンドボックス環境ではアクセスキーを使用しない。
- IAMロール作成時は `ChildAccountRoleBoundary` を適用する。
- NACLの穴あけは実施されない。
- SSM用VPC Endpointは既に存在する。
- 管理用に予約された `CP-SandD*-D-*` に該当する名称を作成しない。

### 5.4 クォータ

EC2で利用可能なインスタンスタイプ:

- t3.nano
- t3.micro
- t3.small
- t3.medium
- t2.nano
- t2.micro
- t2.small
- t2.medium

EBS:

- gp2
- st1
- sc1
- standard
- 最大100GB

RDS:

- db.t3.micro
- db.t3.small
- db.t3.medium
- 最大100GB

## 6. 接続方式

### 6.1 Windows EC2

- SMART Gateway経由のRDPを利用する。
- Public Subnetへ配置する。
- Public IPv4を有効にする。
- SMART Gatewayの許可IPだけからRDPを許可するSecurity Groupを利用する。
- Windows Server 2022 Japanese Full Baseを利用する。
- AMI IDは固定せず、構築時に最新の公式AMIを検索する。
- インスタンスタイプは `t3.medium`。
- Windows EC2はGUI中心のリモート開発端末とする。
- Windows Server上のDocker Desktopは採用しない。

### 6.2 Linux EC2

- 原則としてSSM Session Managerを利用する。
- SSHキーとSSHインバウンドは使用しない。
- SSMだけで管理できる構成ではインバウンドルールを設けない。
- Docker EngineとDocker ComposeはLinux EC2で実行する。

### 6.3 Terraform Runner

- AWS CloudShellは利用できない。
- 社内PCからAWS CLIおよびTerraformを直接実行できない。
- Terraform Runner EC2のみ、AWSコンソールから手動でBootstrapする。
- OSはAmazon Linux 2023を第一候補とする。
- インスタンスタイプは `t3.micro`。
- Public Subnetへ配置し、Public IPv4を有効にする。
- インバウンドルールは設定しない。
- SSM Session Managerだけで接続する。
- キーペアを使用しない。
- Terraform実行時だけ起動し、週末前に最後に削除する。
- Private SubnetにNATまたは承認済みProxyが利用可能になった場合のみ、Private Subnetへの移行を検討する。

## 7. TerraformとState

- Terraformは固定バージョンを使用する。
- AWS Providerも固定バージョンを使用する。
- 実際の作成時点で公式情報を確認し、安定したバージョンを固定する。
- `.terraform.lock.hcl`をGit管理する。
- 実行中のStateは非公開S3 Backendへ保存する。
- S3 Backendでは暗号化、バージョニング、`use_lockfile = true`を利用する。
- S3は週末削除対象なので永続的な正本にしない。
- 週末前に全ワークロードをdestroyし、空のStateへ更新する。
- StateのBox Drive保存可否が確定していないため、許可確認前はBox DriveへStateを保存しない。
- Terraform Runner、Runner用IAM、Runner用Security Group、Bootstrap S3はTerraform管理外のBootstrapリソースとする。
- BootstrapリソースはAWSコンソールで手動作成・手動削除する。

## 8. コードと文書の管理

### 8.1 ランブック

- VS Codeで編集する。
- ローカルGitを正本とする。
- リモートGitは使用しない。
- Box DriveへGit Bundleと必要に応じてZIPでバックアップする。
- MarkdownはUTF-8、改行はLFを推奨する。

### 8.2 Public GitHubへ保存可能

- 環境非依存のTerraform Module
- 実在しないダミー値を使ったExample
- 一般公開可能なReact、FastAPI、Pythonの検証コード
- 一般公開可能なDockerfileとCompose定義
- 一般公開可能なBootstrap Script
- `.terraform.lock.hcl`
- `.gitignore`

### 8.3 Public GitHubへ保存禁止

- AWSアカウントID
- VPC ID
- Subnet ID
- Security Group ID
- IAM ARN
- Permissions Boundary ARN
- C-MAC固有のリソース名
- SMART Gateway情報
- 社内URL
- Backendバケット名
- `terraform.tfvars`
- `backend.hcl`
- Terraform State
- Terraform Plan
- パスワード
- 秘密鍵
- トークン
- 社内情報

### 8.4 非公開IaC

- ローカルGitとBox Driveで管理する。
- 実行時にZIP化し、AWSコンソールから非公開S3へアップロードする。
- RunnerはS3からZIPを取得する。
- Public GitHubへpushしない。

## 9. Docker方針

- Docker EngineとDocker ComposeをWebアプリ検証の標準要素とする。
- React、FastAPI、必要に応じたローカル検証用DBをコンテナ化する。
- Snowflake自体はコンテナ化しない。
- SnowflakeにはFastAPIなどサーバーサイドから接続する。
- DockerfileとCompose定義へ秘密情報を記載しない。
- コンテナイメージへ秘密情報を埋め込まない。
- Dockerソケットを外部公開しない。
- コンテナは可能な限り非rootで実行する。
- ベースイメージはバージョンまたはダイジェストを固定する。
- 永続データをコンテナまたはEC2内だけに保存しない。

## 10. 初期構成パターン

### Pattern 00: Terraform Bootstrap

- Bootstrap S3
- Runner用IAM
- Runner用Security Group
- Terraform Runner EC2
- SSM接続
- Terraform CLI導入
- S3 Backend
- 非公開IaCの搬送
- 週末前の破棄

### Pattern 01: Windows開発用EC2

- Windows Server 2022 Japanese
- t3.medium
- Public Subnet
- Public IPv4
- SMART Gateway + RDP
- VS Code
- Git
- Claude Code
- Node.js
- Python
- Snowflakeクライアント
- 個人Public GitHubへの接続
- Dockerは対象外

### Pattern 02: Linux Docker開発用EC2

- Linux
- SSM
- Docker Engine
- Docker Compose
- React
- FastAPI
- Python
- Git
- Claude Code
- Snowflake接続

### Pattern 03: Docker ComposeによるReact + FastAPI + Snowflake

- Reactフロントエンドコンテナ
- FastAPIバックエンドコンテナ
- Docker Composeによる一括起動
- FastAPIからSnowflakeへの接続
- ヘルスチェック
- 構造化ログ
- 入力バリデーション
- 相関ID
- エラー処理

### Pattern 04: 定期ETL・バッチ

- 外部データ取得
- 変換
- Snowflakeロード
- 冪等性
- 再試行
- タイムアウト
- 構造化ログ
- 実行結果確認

### Pattern 05: Web公開構成

- ALBまたはAPI Gateway
- Private Subnetのアプリケーション
- 必要時のWAF
- TLS
- アクセスログ
- 公開範囲の最小化

## 11. コンポーネント選定原則

- 最初にAWSサービスを選ばず、試験目的、公開範囲、処理方式、データ保持、外部接続、可用性から必要なコンポーネントを決める。
- 本番構成を無条件に再現しない。
- 不要なALB、WAF、NAT Gateway、Cache、Broker、Multi-AZを作らない。
- コンポーネント自体が試験対象の場合に限り採用する。
- 同じ目的を達成できる場合はマネージドサービスを優先する。
- DB、Batch、内部APIを直接インターネットへ公開しない。
- 踏み台はSSMで代替できない場合に限る。
- 採用したコンポーネントだけでなく、不採用理由と採用へ切り替える条件も記録する。

## 12. 各構成パターンに必須の章

1. 用途
2. 成功条件
3. 非スコープ
4. 前提条件
5. 設計判断
6. 採用コンポーネント
7. 不採用コンポーネントと切替条件
8. 制約、代替策、残存リスク
9. Mermaid構成図
10. 通信フロー
11. データフロー
12. 作成するAWSリソース
13. パラメーター
14. 命名とタグ
15. IAM設計
16. Permissions Boundary
17. Security Group設計
18. Terraform構成
19. 手動例外手順
20. OSとミドルウェア初期設定
21. Docker構成
22. アプリケーション配置
23. 正常系確認
24. 異常系確認
25. ログと監視
26. 停止手順
27. 個別削除
28. 全体削除
29. 週末前の退避
30. 週末後の再構築
31. トラブルシューティング
32. コスト
33. 未確定事項
34. 更新履歴

## 13. 命名とタグ

予約名称 `CP-SandD*-D-*` に該当しないこと。

暫定命名形式:

```text
<owner>-<purpose>-<resource>-<sequence>
```

必須タグ:

- Name
- Owner
- Purpose
- Environment = sandbox
- ManagedBy = terraform または manual-bootstrap
- ExpiresAt

## 14. セキュリティ原則

- AWSアクセスキーを作成または保存しない。
- 秘密情報をMarkdown、Git、ソースコード、Dockerfile、Compose、ログ、スクリーンショットへ含めない。
- `.env`をGit管理しない。
- `.env.example`にはダミー値のみ記載する。
- IAMは最小権限を目標とし、BoundaryとC-MAC制約を優先する。
- RDP、SSH、DBポートをインターネット全体へ公開しない。
- Snowflake認証情報をReactクライアントへ含めない。
- ログへパスワード、トークン、個人情報、機密情報を出力しない。
- `terraform plan`を確認せず`apply`しない。
- Terraform StateとPlanは機密性のあるファイルとして扱う。

## 15. 必須`.gitignore`

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfvars.json
*.auto.tfvars
*.auto.tfvars.json
backend.hcl
override.tf
override.tf.json
*_override.tf
*_override.tf.json
*.tfplan
crash.log
crash.*.log
.env
.env.*
!.env.example
*.pem
*.key
*.p12
*.pfx
*.zip
node_modules/
__pycache__/
.pytest_cache/
.venv/
dist/
build/
```

## 16. 要求する最終フォルダー構成

Claude Codeは、以下を最終形として必要なファイルをすべて生成すること。

```text
aws-sandbox-runbook/
├── README.md
├── CHANGELOG.md
├── .gitignore
├── 01_constraints/
├── 02_common-design/
├── 03_common-runbook/
├── 04_patterns/
│   ├── 00_terraform-bootstrap/
│   │   ├── README.md
│   │   ├── iam/
│   │   └── scripts/
│   ├── 01_windows-dev-ec2/
│   │   ├── README.md
│   │   ├── terraform/
│   │   └── scripts/
│   ├── 02_linux-docker-dev/
│   │   ├── README.md
│   │   ├── terraform/
│   │   └── scripts/
│   ├── 03_docker-react-fastapi-snowflake/
│   │   ├── README.md
│   │   ├── compose.yaml
│   │   ├── .env.example
│   │   ├── frontend/
│   │   └── api/
│   ├── 04_scheduled-etl/
│   └── 05_public-web/
├── 05_components/
├── 06_troubleshooting/
├── 07_templates/
├── 08_iac/
│   ├── README.md
│   ├── bootstrap/
│   ├── patterns/
│   └── modules/
├── 09_scripts/
├── 10_assets/
└── 99_references/
```

空ファイルや内容のないREADMEを大量に作らないこと。未確定のため実装できない箇所は、理由、影響、確認方法、差し替え箇所を明記すること。

## 17. 生成品質

- すべてのMarkdownリンクを相対パスで整合させる。
- 同じ内容を複数ファイルへ重複させすぎない。
- 共通事項は共通文書へ集約し、用途別文書から参照する。
- コマンドはコピペ可能にする。
- PowerShell、Bash、HCL、JSON、YAML、dotenvを適切なコードフェンスで記載する。
- プレースホルダーは `<PLACEHOLDER_NAME>` の形式で統一する。
- 実在するC-MAC固有値をPublic向けExampleへ書かない。
- 依存バージョンは固定する。
- Terraform `fmt`、`validate`を通す。
- Pythonには型ヒント、入力バリデーション、構造化ログ、例外処理、タイムアウトを含める。
- FastAPIにはPydanticモデル、ヘルスチェック、相関IDを含める。
- Reactには環境変数の境界を設け、秘密情報を含めない。
- Dockerは非root、Healthcheck、固定ベースイメージ、必要最小限のレイヤーを考慮する。
- TerraformのIAM権限は広範なワイルドカードを避け、避けられない場合は理由と縮小方法を記載する。
- S3 State BackendはPublic Access Block、暗号化、Versioning、Lockfileを前提にする。
- AWSコンソールUIは変化するため、画面文言だけでなく確認すべき設定値を記載する。

## 18. 未確定事項

以下はClaude Codeが勝手に推測して確定してはならない。

- 実際のAWSアカウントID
- VPC ID
- Public Subnet ID
- Private Subnet ID
- SMART Gateway許可IP
- Permissions Boundary ARN
- S3バケット名
- Snowflakeアカウント識別子
- Snowflake認証方式
- GitHubリポジトリURL
- Claude Codeの認証情報
- C-MACで利用可能なAWSサービスの最新版
- StateをBox Driveへ保存してよいか
- Public SubnetからGitHub、HashiCorp、Terraform Registryへ接続できるか

これらは `<PLACEHOLDER_NAME>` とし、確認手順と設定箇所を記載すること。

## 19. 作業完了条件

Claude Codeは、以下を満たすまで作業を完了としてはならない。

- 最終フォルダー構成を出力した。
- ルートREADMEから各文書へ移動できる。
- Terraform Bootstrapの手順が開始から削除までつながっている。
- Windows開発用EC2のTerraformと手順が揃っている。
- Linux Docker開発用EC2のTerraformと手順が揃っている。
- React + FastAPI + Snowflakeの最小サンプルが起動可能な形で揃っている。
- 週末前destroyと週末後rebuildのRunbookが揃っている。
- IAM、SG、秘密情報、Public GitHubの境界が明記されている。
- すべての未確定値がプレースホルダー化されている。
- Markdownリンク切れを検査した。
- Terraform FormatとValidateの検査手順を実行した。
- 生成ファイル一覧、未解決事項、次に利用者が入力すべき値を最終報告した。
