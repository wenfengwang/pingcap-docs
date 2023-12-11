---
title: アップグレードおよびアップグレード後のFAQ
summary: TiDBのアップグレード中およびアップグレード後に発生するいくつかのFAQとその解決策について学びましょう。
aliases: ['/docs/dev/faq/upgrade-faq/','/docs/dev/faq/upgrade/']
---

# アップグレードおよびアップグレード後のFAQ

このドキュメントでは、TiDBをアップグレードする際およびアップグレード後に発生するいくつかのFAQとそれに対する解決策について紹介します。

## アップグレードのFAQ

このセクションでは、TiDBをアップグレードする際に発生するいくつかのFAQとそれに対する解決策をリストアップします。

### ローリングアップデートの影響は何ですか？

TiDBサービスにローリングアップデートを適用すると、稼働中のアプリケーションにさまざまな程度で影響があります。したがって、ビジネスのピーク時間中にローリングアップデートを実行しないことをお勧めします。最小クラスタトポロジ（TiDB \* 2、PD \* 3、TiKV \* 3）を構成する必要があります。クラスタにPumpまたはDrainerサービスが関与している場合は、ローリングアップデートの前にDrainerを停止することをお勧めします。TiDBのアップグレード時には、Pumpもアップグレードされます。

### DDL実行中にTiDBクラスタをアップグレードできますか？

* アップグレード前のTiDBバージョンがv7.1.0より古い場合：

    * **クラスタでDDLステートメントが実行中の場合**（通常は`ADD INDEX`や列の型変更など、時間がかかるDDLステートメントを指します）、TiDBクラスタをアップグレードしないでください。アップグレードの前に、[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用して、TiDBクラスタで実行中のDDLジョブを確認することをお勧めします。クラスタにDDLジョブがある場合、アップグレードするためには、DDLの実行が終了するのを待つか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルしてからクラスタをアップグレードしてください。

    * クラスタのアップグレード中には、**DDLステートメントを実行しないでください**。さもないと、未定義の動作の問題が発生する可能性があります。

* アップグレード前のTiDBバージョンがv7.1.0以上の場合：

    * 以前のバージョンからv7.1.0にアップグレードする際の制限に従う必要はありません。つまり、TiDBはアップグレード中にユーザーのDDLタスクを受け取ることができます。詳細については、[TiDBスムーズアップグレード](/smooth-upgrade-tidb.md)を参照してください。

### バイナリを使用してTiDBをアップグレードする方法は？

バイナリを使用してTiDBをアップグレードすることはお勧めしません。代わりに、バージョンの一貫性と互換性を確保するために、[TiUPを使用してTiDBをアップグレードする](/upgrade-tidb-using-tiup.md)か、[Kubernetes上のTiDBクラスタをアップグレードする](https://docs.pingcap.com/tidb-in-kubernetes/stable/upgrade-a-tidb-cluster)ことをお勧めします。

## アップグレード後のFAQ

このセクションでは、TiDBをアップグレードした後に発生するいくつかのFAQとそれに対する解決策をリストアップします。

### DDL操作を実行中に文字セット（charset）エラーが発生する

v2.1.0およびそれ以前のバージョン（v2.0のすべてのバージョンを含む）では、TiDBの文字セットはデフォルトでUTF-8です。しかし、v2.1.1からはデフォルトの文字セットがUTF8MB4に変更されました。

v2.1.0またはそれ以前のバージョンで新しく作成されたテーブルの文字セットをUTF-8と明確に指定した場合、TiDBをv2.1.1にアップグレードした後にDDL操作を実行できなくなる場合があります。

この問題を回避するには、次の点に注意する必要があります：

- v2.1.3よりも以前のバージョンでは、TiDBは列の文字セットの変更をサポートしていません。そのため、DDL操作を実行する際には、新しい列の文字セットが元の列と一致していることを確認する必要があります。

- v2.1.3よりも以前のバージョンでは、列の文字セットがテーブルの文字セットと異なる場合でも、`show create table`では列の文字セットが表示されません。ただし、次の例のように、HTTP APIを使用してテーブルのメタデータを取得することで確認できます。

#### `unsupported modify column charset utf8mb4 not match origin utf8`

- アップグレード前に、次の操作がv2.1.0およびそれ以前のバージョンで実行されています。

    {{< copyable "sql" >}}

    ```sql
    create table t(a varchar(10)) charset=utf8;
    ```

    ```
    クエリが正常に完了しました
    Time: 0.106s
    ```

    {{< copyable "sql" >}}

    ```sql
    show create table t;
    ```

    ```
    +-------+-------------------------------------------------------+
    | Table | Create Table                                          |
    +-------+-------------------------------------------------------+
    | t     | CREATE TABLE `t` (                                    |
    |       |   `a` varchar(10) DEFAULT NULL                        |
    |       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin |
    +-------+-------------------------------------------------------+
    1行がセットされています
    Time: 0.006s
    ```

- アップグレード後に、次のエラーがv2.1.1およびv2.1.2で報告されますが、v2.1.3以降ではこのようなエラーは発生しません。

    {{< copyable "sql" >}}

    ```sql
    alter table t change column a a varchar(20);
    ```

    ```
    ERROR 1105 (HY000): unsupported modify column charset utf8mb4 not match origin utf8
    ```

解決策：

列の文字セットを元の文字セットと同じに明示的に指定できます。

{{< copyable "sql" >}}

```sql
alter table t change column a a varchar(22) character set utf8;
```

- Point #1に従えば、列の文字セットを指定しない場合、デフォルトでUTF8MB4が使用されるため、元の文字セットと一致するように列の文字セットを指定する必要があります。

- Point #2に従えば、HTTP APIを使用してテーブルのメタデータを取得し、列名とキーワード「Charset」を検索することで、列の文字セットを確認できます。

    {{< copyable "shell-regular" >}}

    ```sh
    curl "http://$IP:10080/schema/test/t" | python -m json.tool
    ```

    ここではJSONをフォーマットするためにPythonツールを使用していますが、これは必須ではなく、コメントを追加するための便宜上のみです。

    ```json
    {
        "ShardRowIDBits": 0,
        "auto_inc_id": 0,
        "charset": "utf8",   # テーブルの文字セット。
        "collate": "",
        "cols": [            # 列に関する情報。
            {
                ...
                "id": 1,
                "name": {
                    "L": "a",
                    "O": "a"   # 列名。
                },
                "offset": 0,
                "origin_default": null,
                "state": 5,
                "type": {
                    "Charset": "utf8",   # 列aの文字セット。
                    "Collate": "utf8_bin",
                    "Decimal": 0,
                    "Elems": null,
                    "Flag": 0,
                    "Flen": 10,
                    "Tp": 15
                }
            }
        ],
        ...
    }
    ```

#### `unsupported modify charset from utf8mb4 to utf8`

- アップグレード前に、次の操作がv2.1.1およびv2.1.2で実行されています。

    {{< copyable "sql" >}}

    ```sql
    create table t(a varchar(10)) charset=utf8;
    ```

    ```
    クエリが正常に完了しました
    Time: 0.109s
    ```

    {{< copyable "sql" >}}

    ```sql
    show create table t;
    ```

    ```
    +-------+-------------------------------------------------------+
    | Table | Create Table                                          |
    +-------+-------------------------------------------------------+
    | t     | CREATE TABLE `t` (                                    |
    |       |   `a` varchar(10) DEFAULT NULL                        |
    |       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin |
    +-------+-------------------------------------------------------+
    ```

    上記の例では、`show create table`はテーブルの文字セットのみを表示し、列の文字セットは実際にはUTF8MB4ですが、HTTP APIを使用してスキーマを取得することで確認できます。ただし、新しいテーブルを作成する際は、列の文字セットはテーブルと一致している必要があります。このバグはv2.1.3で修正されました。

- アップグレード後に、次の操作がv2.1.3およびそれ以降で実行されます。

    {{< copyable "sql" >}}

    ```sql
    show create table t;
    ```

    ```
    +-------+--------------------------------------------------------------------+
    | Table | Create Table                                                       |
    +-------+--------------------------------------------------------------------+
    | t     | CREATE TABLE `t` (                                                 |
    |       |   `a` varchar(10) CHARSET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL |
    |       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin              |
    +-------+--------------------------------------------------------------------+
    1行がセットされています
    Time: 0.007s
    ```

    {{< copyable "sql" >}}

    ```sql
    alter table t change column a a varchar(20);
    ```

    ```
```
エラー1105（HY000）：UTF8MB4からUTF8への文字セットの変更がサポートされていません。
```

ソリューション：

- v2.1.3以降、TiDBは列とテーブルの文字セットを変更することをサポートしているため、テーブルの文字セットをUTF8MB4に変更することが推奨されています。

    {{< コピー可能 "sql" >}}

    ```sql
    alter table t convert to character set utf8mb4;
    ```

- Issue＃1で指定されているように、列の文字セットを指定することもできます。これにより元の列の文字セット（UTF8MB4）と一貫した状態になります。

    {{< コピー可能 "sql" >}}

    ```sql
    alter table t change column a a varchar(20) character set utf8mb4;
    ```

#### `ERROR 1366 (HY000): column aに対して正しくないutf8値 f09f8c80(🌀)`

TiDB v2.1.1およびそれ以前のバージョンでは、文字セットがUTF-8の場合、挿入された4バイトのデータに対してUTF-8 Unicodeエンコーディングのチェックは行われませんでした。しかし、v2.1.2およびそれ以降のバージョンでは、このチェックが追加されました。

- アップグレードする前に、以下の操作がv2.1.1およびそれ以前のバージョンで実行されます。

    {{< コピー可能 "sql" >}}

    ```sql
    create table t(a varchar(100) charset utf8);
    ```

    ```
    クエリが実行され、0行が影響を受けました
    ```

    {{< コピー可能 "sql" >}}

    ```sql
    insert t values (unhex('f09f8c80'));
    ```

    ```
    クエリが実行され、1行が影響を受けました
    ```

- アップグレード後、v2.1.2およびそれ以降のバージョンで以下のエラーが報告されます。

    {{< コピー可能 "sql" >}}

    ```sql
    insert t values (unhex('f09f8c80'));
    ```

    ```
    ERROR 1366 (HY000): column aに対して正しくないutf8値 f09f8c80(🌀)
    ```

ソリューション：

- v2.1.2：このバージョンでは列の文字セットの変更がサポートされていないため、UTF-8チェックをスキップする必要があります。

    {{< コピー可能 "sql" >}}

    ```sql
    set @@session.tidb_skip_utf8_check=1;
    ```

    ```
    クエリが実行され、0行が影響を受けました
    ```

    {{< コピー可能 "sql" >}}

    ```sql
    insert t values (unhex('f09f8c80'));
    ```

    ```
    クエリが実行され、1行が影響を受けました
    ```

- v2.1.3およびそれ以降のバージョン：列の文字セットをUTF8MB4に変更することが推奨されています。または、UTF-8のチェックをスキップするために`tidb_skip_utf8_check`を設定することができます。ただし、チェックをスキップすると、TiDBからMySQLへのデータのレプリケーションに失敗する可能性があります。

    {{< コピー可能 "sql" >}}

    ```sql
    alter table t change column a a varchar(100) character set utf8mb4;
    ```

    ```
    クエリが実行され、0行が影響を受けました
    ```

    {{< コピー可能 "sql" >}}

    ```sql
    insert t values (unhex('f09f8c80'));
    ```

    ```
    クエリが実行され、1行が影響を受けました
    ```

    具体的には、変数`tidb_skip_utf8_check`を使用してデータの正当なUTF-8およびUTF8MB4のチェックをスキップできます。ただし、チェックをスキップすると、TiDBからMySQLへのデータのレプリケーションに失敗する可能性があります。

    UTF-8のチェックのみをスキップしたい場合は、`tidb_check_mb4_value_in_utf8`を設定できます。この変数はv2.1.3で`config.toml`ファイルに追加され、構成ファイルで`check-mb4-value-in-utf8`を修正してクラスターを再起動して有効にすることができます。

    v2.1.5以降、`tidb_check_mb4_value_in_utf8`をHTTP APIおよびセッション変数を介して設定できます。

    * HTTP API（HTTP APIは単一のサーバーでのみ有効にできます）

        * HTTP APIを有効にするには：

            {{< コピー可能 "shell-regular" >}}

            ```sh
            curl -X POST -d "check_mb4_value_in_utf8=1" http://{TiDBIP}:10080/settings
            ```

        * HTTP APIを無効にするには：

            {{< コピー可能 "shell-regular" >}}

            ```sh
            curl -X POST -d "check_mb4_value_in_utf8=0" http://{TiDBIP}:10080/settings
            ```

    * セッション変数

        * セッション変数を有効にするには：

            {{< コピー可能 "sql" >}}

            ```sql
            set @@session.tidb_check_mb4_value_in_utf8 = 1;
            ```

        * セッション変数を無効にするには：

            {{< コピー可能 "sql" >}}

            ```sql
            set @@session.tidb_check_mb4_value_in_utf8 = 0;
            ```