# Informatica 2026年7月リリースに関する最終影響調査報告書

ご提示いただいたすべての詳細画面（`Enhanced connectors`、`Changed behavior`、および今回の `Important notices`）の検証を完了しました。
7月リリースのサマリーとデータ基盤開発における最影響度サマリーを以下の通り報告いたします。

---

## 1. 総合結論：データ連携ジョブ自体は安全、ただし「自動化スクリプト」に要警戒

本リリースにより、SalesforceからSnowflakeへデータを転送する「マッピング」や「タスク」のロジックそのものが破損するリスクはありません。

ただし、今回の [Important notices](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Important_notices_5.html) で共有いただいた**「セッションベース認証の廃止方針（JWTへの移行）」**は、APIを用いたジョブの運用自動化や外部連携を行っている場合に**中〜長期的に大きな影響を及ぼすリスク**があります。

---

## 2. 各セクションの検証結果サマリー

### ① 重要なお知らせ（[Important notices](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Important_notices_5.html)）
* **内容:** ユーザー認証における「セッションベース認証」のサポート終了予告と、JSON Web Token（JWT）への移行。
* **技術的詳細:** * 2025年10月時点でセッションベース認証はすでに非推奨（Deprecated）となっています。
  * **2026年7月リリースの「しばらく後」に、JWT認証がデフォルトの認証方式になります。**
  * これに伴い、REST APIを呼び出すスクリプトは、トークンの有効期限が切れる前後に「自動再認証」を処理できるようにコードを修正する必要があります。
* **影響度:** **【中〜高（運用方法による）】**
  * Informaticaの画面上での開発には影響しません。
  * ジョブの起動や監視を、外部システム（Hinemos、JP1、Airflow、自作スクリプトなど）から**Informatica REST API**を叩いて実行している場合、認証エラーでジョブが起動できなくなる未来のリスク（または2026年7月以降に管理者がJWTを有効化した際のリスク）があります。

### ② 仕様変更（[Changed behavior](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Changed_behavior_5.html)）
* **内容:** REST APIによるプロジェクト・フォルダの削除（DELETE）要求時における、アセットタイプの厳密なチェック。
* **影響度:** **【低（通常運用なら影響なし）】**
  * API経由でプロジェクトやフォルダを削除する特殊なCI/CD自動化等を行っていない限り、影響はありません。

### ③ コネクタ強化（[Enhanced connectors](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Enhanced_connectors.html)）
* **内容:** Databricks、Microsoft Azure Synapse/Fabric の機能強化。
* **影響度:** **【なし】**
  * Salesforce Connector および Snowflake Connector に関する変更は含まれていません。

---

## 3. 今すぐ確認すべき「影響チェックリスト」

リリース後のトラブルを未然に防ぐため、以下の運用・実装がないか最終確認を行ってください。

* [ ] **Q1. 外部システムからInformaticaのREST APIを呼び出していますか？**
  * ➔ **YESの場合:** 認証ヘッダーに `icSessionId` などを設定する従来のセッション方式を使っている場合、JWT方式（有効期限付きトークンのハンドリング）へのスクリプト改修を計画してください。
  * *注意:* もし「REST V2 Connector」を使ってIDMC自体のAPIを叩いている場合は、JWTを有効化すると競合するため、ドキュメントに記載されている通り「API Center」への移行検討が必要です。
* [ ] **Q2. ジョブのデプロイや削除をAPIで自動化していますか？**
  * ➔ **YESの場合:** フォルダやプロジェクトを削除するAPIリクエストが、正しいアセットIDをターゲットにしているかコードを見直してください（不整合があるとエラーになります）。

---

## 4. 総括

今回の2026年7月リリースにおいて、**「SalesforceからSnowflakeへのデータパイプラインそのものが動かなくなる」という直近の致命的な変更はありません。現在開発中のロジックやマッピング定義はそのまま進めて問題ありません。

ただし、**「APIを使った認証方式の世界的な変更（JWT化）」**というインフラ・運用面の大きな波が来ているため、インフラ担当者や運用設計者とこの情報を共有し、認証スクリプトのアップデート時期を会話しておくことを強く推奨いたします。
