Windows開発用EC2のRDP接続不具合を調査して修正してください。

# 背景

現在 Terraform で作成される Security Group:

yudaitanaka1-windev-ec2-01-sg

には以下のルールがあります。

RDP TCP 3389
- 220.216.68.253/32
- 163.49.24.253/32
- 163.49.23.253/32

しかし SMART Gateway 経由で RDP 接続できません。

一方で EC2 に以下の Security Group を追加すると接続できます。

CP-SandD01-D-SMARTGateway-SG

主なルール:

SSH TCP 22
- 220.216.68.253/32
- 163.49.24.253/32
- 163.49.23.253/32

RDP TCP 3389
- 220.216.68.253/32
- 163.49.24.253/32
- 163.49.23.253/32

ICMP
- 220.216.68.253/32
- 163.49.24.253/32
- 163.49.23.253/32

All Traffic
- 18.99.64.220/30

# やってほしいこと

1. Pattern01 の Security Group 実装を調査する

2. SMART Gateway 接続に必要な通信要件を推測ではなくコードと設計から分析する

3. CP-SandD01-D-SMARTGateway-SGとの差分を整理する

4. 接続失敗の原因候補を列挙する

5. 最小権限で修正案を作成する

6. Terraformコードを修正する

7. terraform test があれば更新する

8. README.md の Security Group 設計も更新する

# 重要

- SSHを無条件に追加しない
- ICMPを無条件に追加しない
- All Trafficを無条件に追加しない
- なぜ必要か説明できる場合のみ追加する
- 追加が不要なら不要と判断する
- 修正理由を ADR 形式でまとめる

# 成果物

- 原因分析
- 修正内容
- Terraform差分
- README差分
- テスト結果
