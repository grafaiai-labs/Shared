Windows 開発サーバー (Pattern 01: Windows 開発用 EC2) の Security Group 定義を変更してください。

変更要件:

■ インバウンドルール

既存のルールは維持したまま、以下のルールを追加してください。

- Type: All Traffic
- Protocol: All
- Port Range: All
- Source: 18.99.64.220/30
- IPv4

AWSコンソール上では以下のルールに相当します。

sgr-0f0e8fe9cb5991cd7
IPv4
すべてのトラフィック
すべて
すべて
18.99.64.220/30

■ アウトバウンドルール

現在定義されているアウトバウンドルールをすべて削除してください。

そのうえで、以下のルールを 1 つだけ定義してください。

- Type: All Traffic
- Protocol: All
- Port Range: All
- Destination: 0.0.0.0/0
- IPv4

AWSコンソール上では以下のルールに相当します。

sgr-08cb20c79ad5792a2
IPv4
すべてのトラフィック
すべて
すべて
0.0.0.0/0

実施内容:

1. Windows 開発サーバー用 Security Group の Terraform 定義を修正
2. 関連する terraform test / plan テストを修正
3. README や設計書に Security Group 仕様が記載されている場合は整合性が取れるよう更新
4. 不要になった 443/80 のアウトバウンドルール定義は削除
5. 変更対象ファイル一覧を提示
6. git diff 形式で変更内容を要約
7. Terraform validate と terraform test が通ることを確認

変更後の期待状態:

Inbound
- TCP 3389 ← RDP許可CIDR (既存)
- All Traffic ← 18.99.64.220/30 (追加)

Outbound
- All Traffic → 0.0.0.0/0 のみ

実装前に関連ファイルを調査し、影響箇所を漏れなく修正してください。
