このディレクトリにある `claude-code-input-spec.md` を、最初から最後まで省略せずに読んでください。

あなたのタスクは、この仕様書を唯一の要件定義として、`C-MAC AWSサンドボックス共通運用ランブック`一式を、このディレクトリ内に完成させることです。

## 最重要指示

- 私への逐次確認をせず、仕様書の確定事項に従って最後まで作業してください。
- 既存の断片的なMarkdownがある場合でも、それを無条件に正とせず、仕様書との整合性を優先してください。
- 仕様書にないC-MAC固有値を推測しないでください。
- 未確定値は必ず `<PLACEHOLDER_NAME>` 形式で記載してください。
- AWSアカウントID、VPC ID、Subnet ID、Security Group ID、IAM ARN、社内URL、SMART Gateway情報、実在するS3バケット名などをPublic向けコードへ記載しないでください。
- パスワード、秘密鍵、トークン、Terraform State、Terraform PlanをGit管理対象へ含めないでください。
- 空のディレクトリや内容のないREADMEを大量に作らないでください。
- Placeholderだけのファイルを完成品として扱わないでください。
- 実行できる箇所は、実際に構文検査と整合性検査を行ってください。
- AWSへ実際のリソースを作成するコマンドは、私の承認なしに実行しないでください。
- Terraformの `apply` と `destroy` は実行しないでください。
- ローカルファイルの作成、書式検査、静的検査、テストは実行してください。

## 作業手順

### 1. 仕様分析

最初に `claude-code-input-spec.md` を読み、以下を内部的に整理してください。

- 確定要件
- 未確定事項
- セキュリティ制約
- C-MAC固有制約
- 必要な成果物
- ファイル間の依存関係
- 実装順序

分析結果を長く説明する必要はありません。作業計画を簡潔に表示した後、そのまま実装してください。

### 2. フォルダーと文書の生成

仕様書で要求された最終フォルダー構成を作成してください。

ただし、空フォルダーを埋めるためだけのファイルは作らず、今回の成果物として意味がある文書とコードを作成してください。

少なくとも以下を完成させてください。

- ルート `README.md`
- `CHANGELOG.md`
- `.gitignore`
- C-MAC制約
- 共通設計
- 作業開始手順
- 作業終了手順
- 週末前の破棄手順
- 週末後の再構築手順
- Terraform Bootstrap
- Windows開発用EC2
- Linux Docker開発用EC2
- Docker ComposeによるReact + FastAPI + Snowflake
- 定期ETL・バッチの基本パターン
- Web公開構成の基本パターン
- 共通コンポーネント手順
- トラブルシューティング
- 構成パターン用テンプレート
- 社内資料と外部資料の参照管理用文書

### 3. Terraform Bootstrap

以下を具体化してください。

- Bootstrap用S3の手動作成手順
- Runner用IAM Trust Policy
- Runner基盤用IAM Policy
- S3 Prefix限定Policy
- Permissions Boundaryの適用箇所
- Runner用Security Group
- Amazon Linux 2023、t3.microのRunner
- Public Subnet、Public IPv4、インバウンドなし
- SSM Session Manager接続
- Terraform固定バージョンの導入
- Checksum検証
- AWS Providerの固定
- S3 Backend
- Versioning
- Encryption
- `use_lockfile = true`
- 非公開IaC ZIPのS3搬送
- 疎通確認
- Runner自身の手動削除
- 週末後のBootstrap再構築

広すぎるIAM権限が必要になる場合は、無条件でワイルドカードを使用せず、理由、想定リスク、将来の縮小方法を文書へ記載してください。

### 4. Windows開発用EC2

以下をTerraformと手順書で実装してください。

- Windows Server 2022 Japanese
- AMI IDをハードコードせず、公式AMIを検索するData Source
- t3.medium
- Public Subnet
- Public IPv4
- SMART Gateway許可IPだけからRDPを許可
- Security Group
- RSA Key Pairの扱い
- Windowsパスワード取得に必要な手順
- SMART Gateway接続
- VS Code
- Git
- Claude Code
- Node.js
- Python
- Snowflake接続クライアント
- 個人Public GitHubへの接続確認
- PowerShellによる可能な範囲の初期化
- 動作確認
- 週末前の退避
- Terraform Destroy

Windows Server上へDocker Desktopを導入しないでください。

### 5. Linux Docker開発用EC2

以下をTerraformと手順書で実装してください。

- C-MACで利用可能なLinux AMI
- SSM Session Manager
- SSHインバウンドなし
- Docker Engine
- Docker Compose Plugin
- Git
- Python
- Node.js
- Claude Code
- Snowflake接続
- ブートストラップスクリプト
- Dockerの非root利用
- Dockerソケットを公開しない構成
- 起動、停止、ログ、削除
- 週末後の再構築

Private Subnetから外部へ接続できることを推測しないでください。

外向き通信が必要な構成では、Public Subnet案とPrivate Subnet + NATまたはProxy案を分け、初期採用案、切替条件、残存リスクを記載してください。

### 6. React + FastAPI + Snowflake

最小限でも実行可能なサンプルを作成してください。

構成:

- Reactフロントエンド
- FastAPIバックエンド
- Docker Compose
- Snowflake接続
- ReactからSnowflakeへ直接接続しない
- FastAPIからSnowflakeへ接続
- `.env.example`
- `.env`はGit管理しない
- ヘルスチェック
- 構造化JSONログ
- 相関ID
- 入力バリデーション
- 制御された例外処理
- タイムアウト
- Snowflake未接続時にもヘルスチェックとエラー内容を判別できる
- ユニットテスト
- 起動、確認、停止、削除手順

React、Python、Dockerの依存バージョンを固定してください。

Snowflakeの実際の認証情報やアカウント識別子は使用せず、ダミー値を使ってください。

### 7. 文書品質

- ルートREADMEを利用者の入口にしてください。
- ルートREADMEから各構成パターンへ相対リンクで移動できるようにしてください。
- 共通事項を各文書へ重複させすぎないでください。
- コマンドをコピーして実行できる形式にしてください。
- PowerShell、Bash、HCL、JSON、YAML、dotenvを適切なコードフェンスで囲んでください。
- 各構成には成功条件と非スコープを記載してください。
- 採用コンポーネントだけでなく、不採用理由と切替条件を記載してください。
- 構築、確認、停止、個別削除、全体削除、週末前、週末後を分けてください。
- AWSコンソール操作では、UIの文言だけでなく確認すべき設定値を記載してください。
- 破壊的操作には明確な警告を付けてください。
- 内部情報をスクリーンショットやログへ残さない注意事項を記載してください。

### 8. 静的検査

利用可能な範囲で以下を実行してください。

- Markdownリンクの存在確認
- Markdownファイルの空ファイル確認
- Terraform `fmt -check -recursive`
- Terraform `validate`
- JSON構文検査
- YAML構文検査
- Docker Composeの構文検査
- Pythonテスト
- Pythonの構文検査
- フロントエンドのビルドまたは型検査
- 機密情報らしい値が含まれていないかの検索
- Placeholderの一覧化

ネットワークアクセスや外部パッケージが必要で実行できない検査は、失敗として隠さず、未実施理由と利用者が実行するコマンドを記載してください。

### 9. 最終報告

作業終了時に、以下だけを簡潔に報告してください。

1. 作成・更新したファイル一覧
2. 最終フォルダー構成
3. 実施した検査と結果
4. 未実施の検査と理由
5. 残っているPlaceholder一覧
6. 私が次に入力・確認すべき値
7. AWS上で最初に実施する手順
8. 既知のリスク

作業途中で止まらず、実装可能な範囲をすべて完成させてください。
