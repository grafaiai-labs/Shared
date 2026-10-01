Windows開発用EC2のSecurity Group実装を改修してください。

背景:
現在作成される Security Group (yudaitanaka1-windev-ec2-01-sg) には
RDP許可ルールのみ作成されています。

実環境でRDP接続できなかったため調査したところ、
CP-SandD01-D-SMARTGateway-SG を追加すると接続できました。

調査結果から、SMART Gateway の送信元IPは以下の3つであることが判明しました。

163.49.23.253/32
163.49.24.253/32
220.216.68.253/32

現在のREADMEでは
rdp_allowed_cidrs による RDP許可のみを前提にしていますが、
実際のSMART Gateway環境に合わせて IaC を修正したいです。

要件:

1.
README、設計書、サンプルtfvarsを更新する。

2.
SMART Gateway CIDR を単一CIDRではなく複数CIDR配列として扱う。

3.
デフォルト例として以下を設定する。

163.49.23.253/32
163.49.24.253/32
220.216.68.253/32

4.
Security Group作成時に以下を作成する。

RDP TCP 3389
- 163.49.23.253/32
- 163.49.24.253/32
- 220.216.68.253/32

5.
以下も追加する。

SSH TCP 22
- 163.49.23.253/32
- 163.49.24.253/32
- 220.216.68.253/32

ICMP ALL
- 163.49.23.253/32
- 163.49.24.253/32
- 220.216.68.253/32

6.
既存のfor_each実装があれば活用し、
CIDR追加時にTerraformコード変更が不要な構造にする。

7.
terraform test を更新する。

8.
READMEの成功条件、Security Group設計、
terraform.tfvars.example を更新する。

9.
CHANGELOG.mdへ変更履歴を追加する。

10.
変更内容を以下形式で報告する。

- 修正ファイル一覧
- 変更理由
- Terraform差分概要
- テスト結果
- 想定されるterraform plan差分
