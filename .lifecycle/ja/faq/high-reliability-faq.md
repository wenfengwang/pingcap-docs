---
title: 高信頼性FAQ
summary: TiDBの高信頼性に関連するFAQについて学びます。

# 高信頼性FAQ

このドキュメントは、TiDBの高信頼性に関連するFAQをまとめたものです。

## TiDBはデータ暗号化をサポートしていますか？

はい。ネットワークトラフィックでデータを暗号化するには、TiDBクライアントとサーバー間でのTLSを[有効にすることができます](/enable-tls-between-clients-and-servers.md)。ストレージエンジンでデータを暗号化するには、[透過的データ暗号化（TDE）](/encryption-at-rest.md)を有効にすることができます。

## TiDBは、サーバーのMySQLバージョン文字列を特定のものに変更することをサポートしていますか？セキュリティの脆弱性スキャンツールで必要とされるものに。

- v3.0.8以降、TiDBは構成ファイルの[`server-version`](/tidb-configuration-file.md#server-version)を変更することによってサーバーのバージョン文字列を変更することをサポートしています。

- v4.0以降、TiUPを使用してTiDBを展開する場合、次のセクションを編集するために`tiup cluster edit-config <cluster-name>`を実行し、適切なバージョン文字列を指定することもできます。

    ```
    server_configs:
      tidb:
        server-version: 'YOUR_VERSION_STRING'
    ```

    その後、`tiup cluster reload <cluster-name> -R tidb`コマンドを使用して、前述の変更を有効にし、セキュリティ脆弱性スキャンの失敗を回避します。

## TiDBはどの認証プロトコルをサポートしていますか？プロセスはどのようなものですか？

MySQL同様、TiDBはユーザーのログイン認証およびパスワード処理にSASLプロトコルをサポートしています。

クライアントがTiDBに接続すると、チャレンジ・レスポンス認証モードが開始します。プロセスは以下のようになります。

1. クライアントがサーバーに接続する。
2. サーバーがランダムな文字列のチャレンジをクライアントに送信する。
3. クライアントがユーザー名とレスポンスをサーバーに送信する。
4. サーバーがレスポンスを検証します。

## ユーザーパスワードと権限はどのように変更すればよいですか？

TiDBでユーザーパスワードを変更するには、`ALTER USER`（例: `ALTER USER 'test'@'localhost' IDENTIFIED BY 'mypass';`）を使用することをお勧めします。これにより、他のノードのパスワードが適時に更新されない状況が生じる可能性がある`UPDATE mysql.user`を使用しないでください。

ユーザーパスワードと権限を変更する際は、公式の標準文を使用することをお勧めします。詳細については、[TiDBユーザーアカウント管理](/user-account-management.md)を参照してください。
