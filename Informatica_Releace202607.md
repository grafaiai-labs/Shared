# Informatica 2026年7月リリースに関する包括的影響調査報告書（第三版）

ご提示いただいた最新の画面（`Data Integration Connectors` ＞ [New and updated connector packages](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/New_and_updated_connector_packages.html)）の検証を完了しました。
7月リリースの全体像と、貴チームの開発（Salesforce ➔ Snowflake へのデータ連携ジョブ）における影響度サマリーを以下の通り更新・改修します。

---

## 1. 総合結論：データ連携ジョブ自体は安全、ただし「自動化スクリプト」と「Salesforceコネクタのアップデート」に要警戒

本リリースにより、SalesforceからSnowflakeへデータを転送する「マッピング」や「タスク」のロジックそのものが破損するリスクはありません。

ただし、`Important notices` にある**「セッションベース認証の廃止方針（JWTへの移行）」**に加え、今回新たに**「Salesforce Connector」自体がアップデート対象に含まれていること**が判明しました。直接的なエラーリスクは低いですが、リリース後の動作検証（リグレッションテスト）の優先度を上げる必要があります。

---

## 2. 2026年7月リリースの全体サマリーと影響度検証

現在判明している仕様変更・機能強化の全体像と、貴チームへの影響判定です。

### ① コネクタパッケージの更新（[New and updated connector packages](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/New_and_updated_connector_packages.html)） 【NEW】
* **変更内容:** 2026年7月リリースにおいて、多数のコネクタパッケージが新機能の追加、バグ修正、または内部修正のためにアップデートされます。
* **対象となる主要コネクタ:** * **Salesforce Connector**、Salesforce Analytics Connector、Salesforce Data 360 Connector
  * REST V2 Connector
  * （※Snowflake Connector は今回のリストに含まれていません）
* **貴チームへの影響度:** **【低〜中（リリース後の検証を推奨）】**
  * 接続自体ができなくなる仕様変更は報告されていませんが、貴チームが利用している **Salesforce Connector に内部修正やバグフィックスが入ります。** リリース後、既存のジョブが正常に動作するかテスト環境での実行確認を行うことを強く推奨します。

### ② 管理者機能の仕様変更（[Administrator > Changed behavior](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Changed_behavior.html)）
* **変更内容:** REST APIを用いたユーザー作成（v3 users resource）時のMFA必須属性追加、Bitbucket構成フィールドのラベル名変更、REST APIによるプロジェクト・フォルダ削除時のバリデーション厳格化。
* **貴チームへの影響度:** **【低（通常運用なら影響なし）】**
  * ユーザー作成やアセット削除などをAPIで自動化していない限り、現在のデータ連携ジョブへの影響はありません。

### ③ 重要なお知らせ（[Important notices](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Important_notices_5.html)）
* **変更内容:** ユーザー認証における「セッションベース認証」のサポート終了予告と、JSON Web Token（JWT）への完全移行。
* **貴チームへの影響度:** **【中〜高（運用方法による）】**
  * ジョブの起動や監視を、外部ジョブ管理ツールや自作スクリプトから **Informatica REST API** を叩いて実行している場合、認証エラーでジョブが動かなくなる長期的なリスクがあります。

### ④ コネクタ強化（[Enhanced connectors](https://onlinehelp.informatica.com/iics/prod/WhatsNew/en/index.htm#page/aa-iics-whats-new/Enhanced_connectors.html)）
* **変更内容:** Databricks（カスタムクエリ対応）、Microsoft Azure Synapse/Fabric（推奨JDBCドライバーの構成）の機能強化。
* **貴チームへの影響度:** **【なし】**

---

## 3. 影響確認チェックリスト

リリース後のトラブルを未然に防ぐため、以下の運用・実装がないか最終確認を行ってください。

* [ ] **Q1. Salesforce Connector のアップデートに伴うテスト計画はありますか？**
  * ➔ **推奨アクション:** 7月リリース適用後、本番反映前に主要な Salesforce ➔ Snowflake 連携ジョブが正常に動作するか、検証環境でテスト実行を行ってください。
* [ ] **Q2. 外部システムからInformaticaのREST APIを呼び出していますか？**
  * ➔ **YESの場合:** 認証ヘッダーに `icSessionId` などを設定する従来のセッション方式を使っている場合、JWT方式へのスクリプト改修を計画してください。
* [ ] **Q3. ユーザー作成やアセット削除、ソース管理（Bitbucket）をAPIや外部連携で自動制御していますか？**
  * ➔ **YESの場合:** リクエストパラメータや削除対象のIDタイプが厳密にあっているか、スクリプトのロジックを確認してください。

---

## 4. 総括

今回の2026年7月リリースにおいて、ジョブが完全に動作しなくなるような致命的な仕様変更はありません。現在開発中のマッピングロジックはそのまま進めて問題ありません。

しかし、**Salesforce Connector自体のパッケージ更新**が行われるため、念のための動作確認テストが必要です。また、中長期的な**「APIのJWT認証化」**に向け、運用・インフラ担当者との情報共有と対策ロードマップの策定を進めておくことを推奨いたします。
