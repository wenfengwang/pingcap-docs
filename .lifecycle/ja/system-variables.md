---
title: システム変数
summary: システム変数を使用してパフォーマンスを最適化したり、実行動作を変更します。
aliases: ['/tidb/dev/tidb-specific-system-variables','/docs/dev/system-variables/','/docs/dev/reference/configuration/tidb-server/mysql-variables/', '/docs/dev/tidb-specific-system-variables/','/docs/dev/reference/configuration/tidb-server/tidb-specific-variables/']
---

# システム変数

TiDBのシステム変数はMySQLと同様の動作をします。設定は `SESSION` または `GLOBAL` のスコープで適用されます:

- `SESSION` のスコープでの変更は現在のセッションのみに影響します。
- `GLOBAL` のスコープでの変更は即座に適用されます。この変数が`SESSION`のスコープも持っている場合、すべてのセッション（あなたのセッションも含む）は現在のセッション値を引き続き使用します。
- 変更は[`SET`ステートメント](/sql-statements/sql-statement-set-variable.md)を使用して行います:

```sql
# これらの2つの同一のステートメントはセッション変数を変更します
SET tidb_distsql_scan_concurrency = 10;
SET SESSION tidb_distsql_scan_concurrency = 10;

# これらの2つの同一のステートメントはグローバル変数を変更します
SET @@global.tidb_distsql_scan_concurrency = 10;
SET GLOBAL tidb_distsql_scan_concurrency = 10;
```

> **注意:**
>
> いくつかの`GLOBAL`変数はTiDBクラスタに永続化されます。このドキュメントの変数のうちいくつかは`クラスタに永続化`設定を持ち、`Yes`または`No`に設定できます。
>
> - `クラスタに永続化: Yes`設定の変数について、グローバル変数が変更されると、すべてのTiDBサーバーに対してシステム変数キャッシュをリフレッシュする通知が送信されます。追加のTiDBサーバーを追加するか既存のTiDBサーバーを再起動すると、永続化された構成値が自動的に使用されます。
> - `クラスタに永続化: No`設定の変数について、変更は接続しているローカルのTiDBインスタンスにのみ適用されます。設定した値を維持するには、`tidb.toml`構成ファイルで変数を指定する必要があります。
>
> さらに、TiDBはいくつかのMySQL変数を読み取り可能かつ設定可能として提示します。これは互換性のために必要です。アプリケーションやコネクタがMySQL変数を読み取ることが一般的です。たとえば、JDBCコネクタはクエリキャッシュ設定を読み取りかつ依存しないにもかかわらず、それを設定します。

> **注意:**
>
> 大きな値が常によりよいパフォーマンスをもたらすわけではありません。また、ステートメントを実行している同時接続数を考慮することも重要です。ほとんどの設定は各接続に適用されますので、変数の単位を考慮することが重要です:
>
> * スレッドの場合、安全な値は通常CPUコア数までです。
> * バイトの場合、安全な値は通常システムメモリ量より少ないです。
> * 時間の場合、単位が秒またはミリ秒であることに注意してください。
>
> 同じ単位を使用する変数は同じリソースセットを競合させる可能性があります。

v7.4.0から、`SET_VAR`を使ってステートメント実行中に一部の`SESSION`変数の値を一時的に変更できるようになりました。ステートメントが実行された後、現在のセッションのシステム変数の値は自動的に元の値に戻ります。このヒントは、オプティマイザと実行エンジンに関連するいくつかのシステム変数を変更するために使用できます。このドキュメントの変数には`ヒントSET_VARに適用`設定があり、`Yes`または`No`に設定できます。

- `ヒントSET_VARに適用: Yes`設定の変数については、ステートメントの実行中に現在のセッションのシステム変数の値を変更するために[`SET_VAR`](/optimizer-hints.md#set_varvar_namevar_value)ヒントを使用できます。
- `ヒントSET_VARに適用: No`設定の変数については、ステートメントの実行中に現在のセッションのシステム変数の値を変更するために[`SET_VAR`](/optimizer-hints.md#set_varvar_namevar_value)ヒントを使用できません。

`SET_VAR`ヒントの詳細については、[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)を参照してください。

## 変数リファレンス

### allow_auto_random_explicit_insert <span class="version-mark">v4.0.3で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: ブール
- デフォルト値: `OFF`
- `INSERT`ステートメントで`AUTO_RANDOM`属性のカラムの値を明示的に指定するかどうかを決定します。

### authentication_ldap_sasl_auth_method_name <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 列挙
- デフォルト値: `SCRAM-SHA-1`
- 可能な値: `SCRAM-SHA-1`、`SCRAM-SHA-256`、`GSSAPI`
- LDAP SASL認証において、この変数は認証方法名を指定します。

### authentication_ldap_sasl_bind_base_dn <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証において、この変数は検索ツリー内の検索範囲を制限します。`AS ...`句なしでユーザを作成すると、TiDBはユーザ名に応じてLDAPサーバ内で`dn`を自動的に検索します。

### authentication_ldap_sasl_bind_root_dn <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証において、この変数はLDAPサーバにログインしてユーザを検索するための`dn`を指定します。

### authentication_ldap_sasl_bind_root_pwd <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証において、この変数はLDAPサーバにログインするためのパスワードを指定します。

### authentication_ldap_sasl_ca_path <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証において、この変数はStartTLS接続のための証明機関ファイルの絶対パスを指定します。

### authentication_ldap_sasl_init_pool_size <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 整数
- デフォルト値: `10`
- 範囲: `[1, 32767]`
- LDAP SASL認証において、この変数はLDAPサーバへの接続プール内の初期接続を指定します。

### authentication_ldap_sasl_max_pool_size <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 整数
- デフォルト値: `1000`
- 範囲: `[1, 32767]`
- LDAP SASL認証において、この変数はLDAPサーバへの接続プール内の最大接続数を指定します。

### authentication_ldap_sasl_server_host <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証において、この変数はLDAPサーバのホスト名またはIPアドレスを指定します。

### authentication_ldap_sasl_server_port <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: 整数
- デフォルト値: `389`
- 範囲: `[1, 65535]`
- LDAP SASL認証において、この変数はLDAPサーバのTCP/IPポート番号を指定します。

### authentication_ldap_sasl_tls <span class="version-mark">v7.1.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: Yes
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: No
- タイプ: ブール
- デフォルト値: `OFF`
- LDAP SASL認証において、この変数はプラグインによるLDAPサーバへの接続がStartTLSで保護されているかどうかを制御します。
### authentication_ldap_simple_auth_method_name <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 列挙型
- デフォルト値: `SIMPLE`
- 可能な値: `SIMPLE`。
- LDAP シンプル認証の場合、この変数は認証メソッド名を指定します。サポートされているのは `SIMPLE` のみです。

### authentication_ldap_simple_bind_base_dn <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 文字列
- デフォルト値: ""
- LDAP シンプル認証の場合、この変数は検索ツリー内の検索範囲を制限します。`AS ...` 句なしでユーザーが作成されると、TiDB は自動的に LDAP サーバー内でユーザー名に基づいて `dn` を検索します。

### authentication_ldap_simple_bind_root_dn <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 文字列
- デフォルト値: ""
- LDAP シンプル認証の場合、この変数は LDAP サーバーにログインしてユーザーを検索するために使用される `dn` を指定します。

### authentication_ldap_simple_bind_root_pwd <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 文字列
- デフォルト値: ""
- LDAP シンプル認証の場合、この変数は LDAP サーバーにログインしてユーザーを検索するために使用されるパスワードを指定します。

### authentication_ldap_simple_ca_path <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 文字列
- デフォルト値: ""
- LDAP シンプル認証の場合、この変数は StartTLS 接続のための証明書機関ファイルの絶対パスを指定します。

### authentication_ldap_simple_init_pool_size <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `10`
- 範囲: `[1, 32767]`
- LDAP シンプル認証の場合、この変数は LDAP サーバーへの接続プール内の初期接続を指定します。

### authentication_ldap_simple_max_pool_size <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `1000`
- 範囲: `[1, 32767]`
- LDAP シンプル認証の場合、この変数は LDAP サーバーへの接続プール内の最大接続数を指定します。

### authentication_ldap_simple_server_host <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 文字列
- デフォルト値: ""
- LDAP シンプル認証の場合、この変数は LDAP サーバーのホスト名または IP アドレスを指定します。

### authentication_ldap_simple_server_port <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `389`
- 範囲: `[1, 65535]`
- LDAP シンプル認証の場合、この変数は LDAP サーバーの TCP/IP ポート番号を指定します。

### authentication_ldap_simple_tls <span class="version-mark">v7.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 真偽値
- デフォルト値: `OFF`
- LDAP シンプル認証の場合、この変数はプラグインによる LDAP サーバーへの接続が StartTLS で保護されているかどうかを制御します。

### auto_increment_increment

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `1`
- 範囲: `[1, 65535]`
- 列に割り当てられる `AUTO_INCREMENT` 値のステップ サイズを制御します。通常は `auto_increment_offset` と組み合わせて使用されます。

### auto_increment_offset

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `1`
- 範囲: `[1, 65535]`
- 列に割り当てられる `AUTO_INCREMENT` 値の初期オフセットを制御します。この設定は通常、`auto_increment_increment` と組み合わせて使用されます。例:

```sql
mysql> CREATE TABLE t1 (a int not null primary key auto_increment);
Query OK, 0 rows affected (0.10 sec)

mysql> set auto_increment_offset=1;
Query OK, 0 rows affected (0.00 sec)

mysql> set auto_increment_increment=3;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (),(),(),();
Query OK, 4 rows affected (0.04 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+
| a  |
+----+
|  1 |
|  4 |
|  7 |
| 10 |
+----+
4 rows in set (0.00 sec)
```

### autocommit

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 真偽値
- デフォルト値: `ON`
- 明示的なトランザクション内でない場合に、ステートメントが自動的にコミットされるかどうかを制御します。詳細については [トランザクションの概要](/transaction-overview.md#autocommit) を参照してください。

### block_encryption_mode

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 列挙型
- デフォルト値: `aes-128-ecb`
- 値のオプション: `aes-128-ecb`, `aes-192-ecb`, `aes-256-ecb`, `aes-128-cbc`, `aes-192-cbc`, `aes-256-cbc`, `aes-128-ofb`, `aes-192-ofb`, `aes-256-ofb`, `aes-128-cfb`, `aes-192-cfb`, `aes-256-cfb`
- この変数は、組み込み関数 `AES_ENCRYPT()` および `AES_DECRYPT()` の暗号化モードを設定します。

### character_set_client

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4`
- クライアントから送信されるデータの文字セットを指定します。文字セットと照合順序の詳細については、TiDB での文字セットと照合順序の使用に関する詳細については、[文字セットと照合順序](/character-set-and-collation.md) を参照してください。必要に応じて [`SET NAMES`](/sql-statements/sql-statement-set-names.md) を使用して文字セットを変更することをお勧めします。

### character_set_connection

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4`
- 指定された文字セットのない文字列リテラルの文字セットを指定します。

### character_set_database

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4`
- この変数は使用中のデフォルトデータベースの文字セットを示します。**この変数を設定することはお勧めしません**。新しいデフォルトデータベースが選択されると、サーバーは変数値を変更します。

### character_set_results
- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4`
- サーバーのデフォルトの文字セットです。

### collation_connection

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4_bin`
- この変数は現在の接続で使用される照合を示します。これはMySQL変数`collation_connection`と一致しています。

### collation_database

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4_bin`
- この変数は使用中のデータベースのデフォルトの照合を示します。**この変数を設定することはお勧めしません**。新しいデータベースが選択されると、TiDBはこの変数の値を変更します。

### collation_server

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `utf8mb4_bin`
- データベース作成時に使用されるデフォルトの照合です。

### cte_max_recursion_depth

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `1000`
- 範囲: `[0, 4294967295]`
- 共通テーブル式での最大再帰深度を制御します。

### datadir

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)ではサポートされていません。

<CustomContent platform="tidb">

- スコープ: 無し
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: コンポーネントとデプロイ方法によって異なります。
    - `"/tmp/tidb"`: [`--store`](/command-line-flags-for-tidb-configuration.md#--store)に`"unistore"`を設定するか、`--store`を設定しない場合。
    - `${pd-ip}:${pd-port}`: TiKVを使用する場合、つまりTiUPおよびTiDB Operator for Kubernetesデプロイメントのデフォルトストレージエンジンの場合。
- この変数はデータが格納される場所を示します。この場所はローカルパス`/tmp/tidb`であるか、データがTiKVに格納されている場合はPDサーバーを指すことができます。`${pd-ip}:${pd-port}`形式の値は、TiDBが起動時に接続するPDサーバーを示します。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: 無し
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: コンポーネントとデプロイ方法によって異なります。
    - `"/tmp/tidb"`: [`--store`](https://docs.pingcap.com/tidb/stable/command-line-flags-for-tidb-configuration#--store)に`"unistore"`を設定するか、`--store`を設定しない場合。
    - `${pd-ip}:${pd-port}`: TiKVを使用する場合、つまりTiUPおよびTiDB Operator for Kubernetesデプロイメントのデフォルトストレージエンジンの場合。
- この変数はデータが格納される場所を示します。この場所はローカルパス`/tmp/tidb`であるか、データがTiKVに格納されている場合はPDサーバーを指すことができます。`${pd-ip}:${pd-port}`形式の値は、TiDBが起動時に接続するPDサーバーを示します。

</CustomContent>

### ddl_slow_threshold

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

- スコープ: GLOBAL
- クラスターへの永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `300`
- 範囲: `[0, 2147483647]`
- 単位: ミリ秒
- 実行時間がしきい値値を超えたDDL操作を記録します。

### default_authentication_plugin

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 列挙型
- デフォルト値: `mysql_native_password`
- 可能な値: `mysql_native_password`, `caching_sha2_password`, `tidb_sm3_password`, `tidb_auth_token`, `authentication_ldap_sasl`, および `authentication_ldap_simple`.
- `tidb_auth_token`認証メソッドはTiDB Cloudの内部操作でのみ使用されます。**この値を設定しないでください**。
- この変数は、サーバーとクライアントの接続が確立されている際にサーバーが広告する認証メソッドを設定します。
- `tidb_sm3_password`メソッドを使用して認証するには、[TiDB-JDBC](https://github.com/pingcap/mysql-connector-j/tree/release/8.0-sm3)を使用してTiDBに接続することができます。

<CustomContent platform="tidb">

この変数の可能な値については、[Authentication plugin status](/security-compatibility-with-mysql.md#authentication-plugin-status)を参照してください。

</CustomContent>

### default_collation_for_utf8mb4 <span class="version-mark">v7.4.0で新規</span>

- スコープ: GLOBAL | SESSION
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 文字列
- デフォルト値: `utf8mb4_bin`
- 値のオプション: `utf8mb4_bin`, `utf8mb4_general_ci`, `utf8mb4_0900_ai_ci`
- この変数は`utf8mb4`文字セットのデフォルトの[照合](/character-set-and-collation.md)を設定するために使用されます。次の文の動作に影響を与えます。
    - [`SHOW COLLATION`](/sql-statements/sql-statement-show-collation.md)および[`SHOW CHARACTER SET`](/sql-statements/sql-statement-show-character-set.md)ステートメントに表示されるデフォルトの照合。
    - [`CREATE TABLE`](/sql-statements/sql-statement-create-table.md)および[`ALTER TABLE`](/sql-statements/sql-statement-alter-table.md)ステートメントが`CHARACTER SET utf8mb4`句を含むとき、照合が指定されていない場合、この変数で指定された照合が使用されます。`CHARACTER SET`句が使用されていない場合の動作には影響しません。
    - [`CREATE DATABASE`](/sql-statements/sql-statement-create-database.md)および[`ALTER DATABASE`](/sql-statements/sql-statement-alter-database.md)ステートメントが`CHARACTER SET utf8mb4`句を含むが照合が指定されていない場合、この変数で指定された照合が使用されます。`CHARACTER SET`句が使用されていない場合の動作には影響しません。
    - `COLLATE`句が使用されていない場合、`_utf8mb4'string'`形式のいかなるリテラル文字列もこの変数で指定された照合を使用します。

### default_password_lifetime <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 65535]`
- 自動パスワードの有効期限のグローバルポリシーを設定します。デフォルト値`0`はパスワードの有効期限が無期限であることを示します。このシステム変数が正の整数`N`に設定されている場合、パスワードの有効期限は`N`日であり、`N`日以内にパスワードを変更する必要があります。

### default_week_format

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 7]`
- `WEEK()`関数で使用される週のフォーマットを設定します。

### disconnect_on_expired_password <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は読み取り専用です。TiDBがパスワードの有効期限が切れたときにクライアント接続を切断するかどうかを示します。変数が`ON`に設定されている場合、パスワードの有効期限が切れたときにクライアント接続が切断されます。変数が`OFF`に設定されている場合、クライアント接続は「サンドボックスモード」に制限され、ユーザーはパスワードリセット操作のみを実行できます。

<CustomContent platform="tidb">

- ユーザーの質問にあるように、期限切れのパスワードのクライアント接続のデフォルトの動作を変更する必要がある場合は、構成ファイル内の[`security.disconnect-on-expired-password`](/tidb-configuration-file.md#disconnect-on-expired-password-new-in-v650)構成項目を変更してください。

   <CustomContent platform="tidb-cloud">

   - 期限切れのパスワードのクライアント接続の動作を変更する必要がある場合は、[TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

   </CustomContent>

### error_count

- スコープ：SESSION
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：整数
- デフォルト値：`0`
- 直近のステートメントで発生したエラーメッセージの数を示す読み取り専用の変数です。

### foreign_key_checks

- スコープ：SESSION | GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：v6.6.0より前は `OFF` 、v6.6.0以降は `ON`
- この変数は外部キー制約のチェックを有効にするかどうかを制御します。

### group_concat_max_len

- スコープ：SESSION | GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：整数
- デフォルト値：`1024`
- 範囲：`[4, 18446744073709551615]`
- `GROUP_CONCAT()` 関数内のアイテムの最大バッファサイズです。

### have_openssl

- スコープ：N/A
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`DISABLED`
- MySQLの互換性のための読み取り専用の変数です。サーバーにTLSが有効になっている場合、サーバーが`YES`と設定します。

### have_ssl

- スコープ：N/A
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`DISABLED`
- MySQLの互換性のための読み取り専用の変数です。サーバーにTLSが有効になっている場合、サーバーが`YES`と設定します。

### hostname

- スコープ：N/A
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：(システムのホスト名)
- TiDBサーバーのホスト名を示す読み取り専用の変数です。

### identity <span class="version-mark">v5.3.0で新規</span>

この変数は [`last_insert_id`](#last_insert_id) のエイリアスです。

### init_connect

- スコープ：GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：""
- `init_connect` 機能は、TiDBサーバーに最初に接続したときに自動的に実行されるSQLステートメントを許可します。`CONNECTION_ADMIN` または `SUPER` 権限がある場合、この `init_connect` ステートメントは実行されません。 `init_connect` ステートメントがエラーを引き起こした場合、ユーザーの接続が終了します。

### innodb_lock_wait_timeout

- スコープ：SESSION | GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：整数
- デフォルト値：`50`
- 範囲：`[1, 3600]`
- 単位：秒
- 悲観的トランザクションのためのロック待ちタイムアウト（デフォルト）です。

### interactive_timeout

> **注意：**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ：SESSION | GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：整数
- デフォルト値：`28800`
- 範囲：`[1, 31536000]`
- 単位：秒
- この変数は、対話型ユーザーセッションのアイドルタイムアウトを表します。対話型ユーザーセッションとは、`mysql_real_connect()` API を使用して `CLIENT_INTERACTIVE` オプションを呼び出して確立されたセッションのことです（例：MySQL Shell および MySQL クライアント）。この変数はMySQLと完全に互換性があります。

### last_insert_id

- スコープ：SESSION
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：整数
- デフォルト値：`0`
- 範囲：`[0, 18446744073709551615]`
- この変数は、挿入ステートメントで生成された最後の `AUTO_INCREMENT` または `AUTO_RANDOM` の値を返します。
- `last_insert_id` の値は、関数 `LAST_INSERT_ID()` によって返される値と同じです。

### last_plan_from_binding <span class="version-mark">v4.0で新規</span>

- スコープ：SESSION
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、前のステートメントで使用された実行計画が [プランバインディング](/sql-plan-management.md) に影響を受けたかどうかを示すために使用されます。

### last_plan_from_cache <span class="version-mark">v4.0で新規</span>

- スコープ：SESSION
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、以前の `execute` ステートメントで使用された実行計画がプランキャッシュから直接取得されたかどうかを示すために使用されます。

### last_sql_use_alloc <span class="version-mark">v6.4.0で新規</span>

- スコープ：SESSION
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：`OFF`
- この変数は読み取り専用です。前のステートメントがキャッシュされたチャンクオブジェクト（チャンクの割り当て）を使用したかどうかを示すために使用されます。

### license

- スコープ：N/A
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：`Apache License 2.0`
- この変数は、TiDBサーバーインストールのライセンスを示します。

### log_bin

- スコープ：N/A
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、[TiDB Binlog](https://docs.pingcap.com/tidb/stable/tidb-binlog-overview) の使用を示します。

### max_allowed_packet <span class="version-mark">v6.1.0で新規</span>

> **注意：**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ：SESSION | GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：`67108864`
- 範囲：`[1024, 1073741824]`
- この値は1024の整数倍である必要があります。1024で割り切れない値の場合、警告が表示され、値は切り捨てられます。たとえば、値が1025に設定されている場合、TiDBでの実際の値は1024になります。
- サーバーおよびクライアントが1回のパケット送信で許可される最大パケットサイズです。
- この変数はMySQLと互換性があります。

### password_history <span class="version-mark">v6.5.0で新規</span>

- スコープ：GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：整数
- デフォルト値：`0`
- 範囲：`[0, 4294967295]`
- この変数は、パスワード変更の回数に基づいてパスワードの再利用を制限するTiDBのパスワード再利用ポリシーを確立するために使用されます。デフォルト値 `0` は、パスワード変更回数に基づくパスワード再利用ポリシーを無効にすることを意味します。この変数を正の整数 `N` に設定した場合、直近の `N` 個のパスワードの再利用が許可されません。

### mpp_exchange_compression_mode <span class="version-mark">v6.6.0で新規</span>

- スコープ：SESSION | GLOBAL
- クラスターに保存：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- デフォルト値：`UNSPECIFIED`
- 値のオプション：`NONE`、`FAST`、`HIGH_COMPRESSION`、`UNSPECIFIED`
- この変数は、MPP Exchange演算子のデータ圧縮モードを指定するために使用されます。この変数は、TiDBがバージョン番号 `1` のMPP実行計画を選択したときに効果を発揮します。変数値の意味は次のとおりです。
    - `UNSPECIFIED`：未指定。TiDBは自動的に圧縮モードを選択します。現在、TiDBは自動的に `FAST` モードを選択します。
  - `NONE`: データ圧縮は使用されません。
    - `FAST`: 高速モード。全体のパフォーマンスは優れており、圧縮比は `HIGH_COMPRESSION` よりも低いです。
    - `HIGH_COMPRESSION`: 高圧縮モード。

### mpp_version <span class="version-mark">v6.6.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスター内で永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `UNSPECIFIED`
- 値のオプション: `UNSPECIFIED`, `0`, `1`
- この変数は、MPP 実行プランの異なるバージョンを指定するために使用されます。バージョンが指定されると、TiDB は指定された MPP 実行プランのバージョンを選択します。変数値の意味は次の通りです。
    - `UNSPECIFIED`: 未指定を意味します。TiDB は自動的に最新バージョン `1` を選択します。
    - `0`: すべての TiDB クラスター バージョンと互換性があります。MPP バージョンが `0` よりも大きい機能はこのモードでは有効になりません。
    - `1`: v6.6.0 で新規追加、TiFlash のデータ圧縮を有効にするために使用されます。詳細については、[MPP バージョンおよびデータ交換圧縮](/explain-mpp.md#mpp-version-and-exchange-data-compression) を参照してください。

### password_reuse_interval <span class="version-mark">v6.5.0 で新規追加</span>

- スコープ: GLOBAL
- クラスター内で永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 4294967295]`
- この変数は、経過時間に基づくパスワード再利用ポリシーを確立するために使用され、TiDB によって経過時間に基づくパスワード再利用ポリシーが無効になっています。この変数を正の整数 `N` に設定する場合、直近の `N` 日間に使用されたパスワードの再利用は許可されません。

### max_connections

- スコープ: GLOBAL
- クラスター内で永続化: いいえ、接続している現在の TiDB インスタンスにのみ適用されます。
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 100000]`
- 単一の TiDB インスタンスで許可される同時接続数の最大値。この変数はリソース制御に使用できます。
- デフォルト値 `0` は制限がありません。この変数の値が `0` よりも大きく、接続数がその値に達すると、TiDB サーバーはクライアントからの新しい接続を拒否します。

### max_execution_time

- スコープ: SESSION | GLOBAL
- クラスター内で永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- 単位: ミリ秒
- ステートメントの最大実行時間。デフォルト値は無制限（ゼロ）です。

> **注意:**
>
> `max_execution_time` システム変数は現在、読み取り専用の SQL ステートメントの最大実行時間を制御しています。タイムアウト値の精度はおおよそ 100ms です。つまり、指定したミリ秒単位で正確にステートメントを終了させるわけではありません。

<CustomContent platform="tidb">

[`MAX_EXECUTION_TIME`](/optimizer-hints.md#max_execution_timen) ヒントのある SQL ステートメントに対して、このステートメントの最大実行時間は、この変数ではなくヒントによって制限されます。このヒントは、[SQL FAQ](/faq/sql-faq.md#how-to-prevent-the-execution-of-a-particular-sql-statement) で説明されているように SQL バインディングとともに使用できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

[`MAX_EXECUTION_TIME`](/optimizer-hints.md#max_execution_timen) ヒントのある SQL ステートメントに対して、このステートメントの最大実行時間は、この変数ではなくヒントによって制限されます。このヒントは、[SQL FAQ](https://docs.pingcap.com/tidb/stable/sql-faq) で説明されているように SQL バインディングとともに使用できます。

</CustomContent>

### max_prepared_stmt_count

- スコープ: GLOBAL
- クラスター内で永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 1048576]`
- 現在の TiDB インスタンスでの [`PREPARE`](/sql-statements/sql-statement-prepare.md) ステートメントの最大数を指定します。
- `-1` の値は、現在の TiDB インスタンスでの `PREPARE` ステートメントの最大数に制限がないことを意味します。
- 変数を上限値 `1048576` を超える値に設定すると、代わりに `1048576` が使用されます:

```sql
mysql> SET GLOBAL max_prepared_stmt_count = 1048577;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> SHOW WARNINGS;
+---------+------+--------------------------------------------------------------+
| Level   | Code | Message                                                      |
+---------+------+--------------------------------------------------------------+
| Warning | 1292 | Truncated incorrect max_prepared_stmt_count value: '1048577' |
+---------+------+--------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'max_prepared_stmt_count';
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| max_prepared_stmt_count | 1048576 |
+-------------------------+---------+
1 row in set (0.00 sec)
```

### plugin_dir

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)でサポートされていません。

- スコープ: GLOBAL
- クラスター内で永続化: いいえ、接続している現在の TiDB インスタンスにのみ適用されます。
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- コマンドライン フラグによって指定されたプラグインをロードするディレクトリを示します。

### plugin_load

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)でサポートされていません。

- スコープ: GLOBAL
- クラスター内で永続化: いいえ、接続している現在の TiDB インスタンスにのみ適用されます。
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- TiDB の起動時にロードするプラグインを示します。これらのプラグインは、コマンドライン フラグによって指定され、カンマで区切られています。

### port

- スコープ: NONE
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `4000`
- 範囲: `[0, 65535]`
- MySQL プロトコルを使用して `tidb-server` が待ち受けるポート。

### rand_seed1

- スコープ: SESSION
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数は、`RAND()` SQL 関数で使用されるランダム値ジェネレータのシードに使用されます。
- この変数の動作は MySQL と互換性があります。

### rand_seed2

- スコープ: SESSION
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数は、`RAND()` SQL 関数で使用されるランダム値ジェネレータのシードに使用されます。
- この変数の動作は MySQL と互換性があります。

### require_secure_transport <span class="version-mark">v6.1.0 で新規追加</span>

> **注意:**
>
> 現在、この変数は[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)でサポートされていません。TiDB Dedicated クラスターでこの変数を有効にしないでください。そうしないと SQL クライアント接続の失敗が発生する可能性があります。この制限は一時的な制御策であり、将来のリリースで解決されます。

- スコープ: GLOBAL
- クラスター内で永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: 自己ホスト型 TiDB および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) の場合は `OFF`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) の場合は `ON`

<CustomContent platform="tidb">

- この変数は、すべての TiDB への接続がローカル ソケットを介して行われるか、TLS を使用して行われることを確実にします。詳細については、[TiDB クライアントとサーバー間での TLS を有効にする](/enable-tls-between-clients-and-servers.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">
- この変数は、TiDBへのすべての接続がローカルソケットを使用するか、TLSを使用することを保証します。

</CustomContent>

- この変数を `ON` に設定すると、TLSが有効になっているセッションからのみTiDBに接続する必要があります。これにより、TLSが正しく構成されていない場合のロックアウトシナリオを防ぐことができます。
- この設定は以前は `tidb.toml` のオプション (`security.require-secure-transport`) でしたが、TiDB v6.1.0からシステム変数に変更されました。

### skip_name_resolve <span class="version-mark">v5.2.0 で新規追加</span>

> **注意:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では、この変数は読み取り専用です。

- スコープ: GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は `tidb-server` インスタンスが接続のハンドシェイクの一部としてホスト名を解決するかどうかを制御します。
- DNSが信頼性に欠ける場合、このオプションを有効にしてネットワークパフォーマンスを向上させることができます。

> **注意:**
>
> `skip_name_resolve=ON` の場合、識別子にホスト名が含まれるユーザーはサーバーにログインできなくなります。例:
>
> ```sql
> CREATE USER 'appuser'@'apphost' IDENTIFIED BY 'app-password';
> ```
>
> 上記の例では、`apphost` をIPアドレスまたはワイルドカード (`%`) で置き換えることをお勧めします。

### socket

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- `tidb-server` がMySQLプロトコルを使用してリッスンするローカルUNIXソケットファイル。

### sql_log_bin

> **注意:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では、この変数は読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- 変更を [TiDB Binlog](https://docs.pingcap.com/tidb/stable/tidb-binlog-overview) に書き込むかどうかを示します。

> **注意:**
>
> `sql_log_bin` をグローバル変数として設定することは推奨されません。なぜなら、将来のTiDBバージョンでは、この変数をセッション変数としてのみ設定できるようになる可能性があるからです。

### sql_mode

- スコープ: SESSION | GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`
- この変数はMySQL互換性の動作を制御します。詳細については、[SQL Mode](/sql-mode.md) を参照してください。

### sql_require_primary_key <span class="version-mark">v6.3.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、テーブルにプライマリキーがあることを要求するかどうかを制御します。この変数が有効になった後は、プライマリキーのないテーブルを作成または変更しようとするとエラーが発生します。
- この機能は、MySQL 8.0 の同様の名前の [`sql_require_primary_key`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_require_primary_key) に基づいています。
- TiCDCを使用する場合、この変数を有効にすることを強くお勧めします。これは、MySQLシンクへの変更のレプリケーションにはテーブルにプライマリキーが必要であるためです。

### sql_select_limit <span class="version-mark">v4.0.2 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `18446744073709551615`
- 範囲: `[0, 18446744073709551615]`
- 単位: 行
- `SELECT` 文で返される行の最大数。

### ssl_ca

<CustomContent platform="tidb">

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- 証明書機関ファイルの場所（存在する場合）。この変数の値はTiDB構成項目 [`ssl-ca`](/tidb-configuration-file.md#ssl-ca) によって定義されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- 証明書機関ファイルの場所（存在する場合）。この変数の値はTiDB構成項目 [`ssl-ca`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-ca) によって定義されます。

</CustomContent>

### ssl_cert

<CustomContent platform="tidb">

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- SSL/TLS接続に使用される証明書ファイル（存在する場合）の場所。この変数の値はTiDB構成項目 [`ssl-cert`](/tidb-configuration-file.md#ssl-cert) によって定義されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- SSL/TLS接続に使用される証明書ファイル（存在する場合）の場所。この変数の値はTiDB構成項目 [`ssl-cert`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-cert) によって定義されます。

</CustomContent>

### ssl_key

<CustomContent platform="tidb">

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- SSL/TLS接続に使用されるプライベートキーファイル（存在する場合）の場所。この変数の値はTiDB構成項目 [`ssl-key`](/tidb-configuration-file.md#ssl-cert) によって定義されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- SSL/TLS接続に使用されるプライベートキーファイル（存在する場合）の場所。この変数の値はTiDB構成項目 [`ssl-key`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-key) によって定義されます。

</CustomContent>

### system_time_zone

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: (システムに依存)
- この変数はTiDBが最初にブートストラップされたときのシステムタイムゾーンを示します。[`time_zone`](#time_zone) も参照してください。

### tidb_adaptive_closest_read_threshold <span class="version-mark">v6.3.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `4096`
- 範囲: `[0, 9223372036854775807]`
- 単位: バイト
- この変数は、[`tidb_replica_read`](#tidb_replica_read-new-in-v40) が `closest-adaptive` に設定されている場合に、TiDBサーバーが同じ可用性ゾーンにあるレプリカに読み取りリクエストを送信する閾値を制御します。推定結果がこの閾値よりも大きいか等しい場合、TiDBは同じ可用性ゾーンにあるレプリカに読み取りリクエストを送信します。それ以外の場合、TiDBはリーダーレプリカに読み取りリクエストを送信します。

### tidb_allow_batch_cop <span class="version-mark">v4.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに持続: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[0, 2]`
- この変数は、TiDBがcoprocessorリクエストをTiFlashに送信する方法を制御します。以下の値があります:

    * `0`: リクエストをバッチで送信しない
    * `1`: 集計および結合リクエストはバッチで送信されます
    * `2`: すべてのcoprocessorリクエストはバッチで送信されます
### tidb_allow_fallback_to_tikv <span class="version-mark">v5.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: ""
- この変数は、TiKVにフォールバックする可能性のあるストレージエンジンのリストを指定するために使用されます。指定したストレージエンジンの実行に失敗した場合、TiDBはこのSQLステートメントをTiKVで再実行します。この変数は""または"tiflash"に設定できます。この変数が"tiflash"に設定されている場合、TiFlashがタイムアウトエラーを返す（エラーコード: ErrTiFlashServerTimeout）と、TiDBはこのSQLステートメントをTiKVで再実行します。

### tidb_allow_function_for_expression_index <span class="version-mark">v5.2.0で新規追加</span>

- スコープ: NONE
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `json_array`, `json_array_append`, `json_array_insert`, `json_contains`, `json_contains_path`, `json_depth`, `json_extract`, `json_insert`, `json_keys`, `json_length`, `json_merge_patch`, `json_merge_preserve`, `json_object`, `json_pretty`, `json_quote`, `json_remove`, `json_replace`, `json_search`, `json_set`, `json_storage_size`, `json_type`, `json_unquote`, `json_valid`, `lower`, `md5`, `reverse`, `tidb_shard`, `upper`, `vitess_hash`
- この変数は、式インデックスを作成するために使用できる関数を表示するために使用されます。

### tidb_allow_mpp <span class="version-mark">v5.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: ブール値
- デフォルト値: `ON`
- クエリを実行する際にTiFlashのMPPモードを使用するかどうかを制御します。値のオプションは以下の通りです:
    - `0`または`OFF`: MPPモードを使用しないことを意味します。
    - `1`または`ON`: 最適化プログラムは通常、コスト見積りに基づいてMPPモードの使用を決定します。

MPPはTiFlashエンジンが提供する分散コンピューティングフレームワークであり、ノード間でのデータ交換を可能にし、高性能で高スループットのSQLアルゴリズムを提供します。MPPモードの選択の詳細については、[MPPモードを選択するかどうかを制御する](/tiflash/use-tiflash-mpp-mode.md#control-whether-to-select-the-mpp-mode)を参照してください。

### tidb_allow_remove_auto_inc <span class="version-mark">v2.1.18およびv3.0.4で新規追加</span>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

- スコープ: SESSION
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`ALTER TABLE MODIFY`または`ALTER TABLE CHANGE`ステートメントを実行して列の`AUTO_INCREMENT`プロパティを削除することが許可されているかどうかを設定するために使用されます。デフォルトでは許可されていません。

### tidb_analyze_partition_concurrency

> **警告:**
>
> この変数によって制御される機能は現在のTiDBバージョンで完全に機能していません。デフォルト値を変更しないでください。

- スコープ: SESSION | GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `2`。v7.4.0およびそれ以前のバージョンではデフォルト値は`1`です。
- この変数は、TiDBが分析する分割テーブルの統計情報を読み書きする際の並行性を指定します。

### tidb_analyze_version <span class="version-mark">v5.1.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `2`
- 範囲: `[1, 2]`
- TiDBが統計情報を収集する方法を制御します。
    - TiDBセルフホストの場合、この変数のデフォルト値はv5.3.0から`1`から`2`に変更されます。
    - TiDB Cloudの場合、この変数のデフォルト値はv6.5.0から`1`から`2`に変更されます。
    - クラスタが以前のバージョンからアップグレードされた場合、`tidb_analyze_version`のデフォルト値はアップグレード後も変更されません。
- この変数の詳細な説明については、[統計情報の紹介](/statistics.md)を参照してください。

### tidb_analyze_skip_column_types <span class="version-mark">v7.2.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: "json,blob,mediumblob,longblob"
- 可能な値: "json,blob,mediumblob,longblob,text,mediumtext,longtext"
- この変数は、`ANALYZE`コマンドを実行して統計情報を収集する際に、どの種類の列を統計情報収集の対象から除外するかを制御します。この変数は、`tidb_analyze_version = 2`の場合のみ適用されます。`ANALYZE TABLE t COLUMNS c1, ... , cn`を使用して列を指定しても、その列のタイプが`tidb_analyze_skip_column_types`に含まれている場合は統計情報が収集されません。

```
mysql> SHOW CREATE TABLE t;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                             |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `a` int(11) DEFAULT NULL,
  `b` varchar(10) DEFAULT NULL,
  `c` json DEFAULT NULL,
  `d` blob DEFAULT NULL,
  `e` longblob DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT @@tidb_analyze_skip_column_types;
+----------------------------------+
| @@tidb_analyze_skip_column_types |
+----------------------------------+
| json,blob,mediumblob,longblob    |
+----------------------------------+
1 row in set (0.00 sec)

mysql> ANALYZE TABLE t;
Query OK, 0 rows affected, 1 warning (0.05 sec)

mysql> SELECT job_info FROM mysql.analyze_jobs ORDER BY end_time DESC LIMIT 1;
+---------------------------------------------------------------------+
| job_info                                                            |
+---------------------------------------------------------------------+
| analyze table columns a, b with 256 buckets, 500 topn, 1 samplerate |
+---------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> ANALYZE TABLE t COLUMNS a, c;
Query OK, 0 rows affected, 1 warning (0.04 sec)

mysql> SELECT job_info FROM mysql.analyze_jobs ORDER BY end_time DESC LIMIT 1;
+------------------------------------------------------------------+
| job_info                                                         |
+------------------------------------------------------------------+
| analyze table columns a with 256 buckets, 500 topn, 1 samplerate |
+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### tidb_auto_analyze_end_time

- スコープ: GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 時刻
- デフォルト値: `23:59 +0000`
- この変数は、統計情報の自動更新が許可される時間枠を制限するために使用されます。たとえば、UTC時間で1時から3時の間に自動統計情報の更新を許可するようにするには、`tidb_auto_analyze_start_time='01:00 +0000'`および`tidb_auto_analyze_end_time='03:00 +0000'`を設定します。

### tidb_auto_analyze_partition_batch_size <span class="version-mark">v6.4.0で新規追加</span>

- スコープ: GLOBAL
- クラスタへ永続化: はい
- ヒントへ適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `1`
- 範囲: `[1, 1024]`
- この変数は、分割テーブルを分析（つまり分割テーブルの統計情報を自動的に収集する）する際にTiDBが自動的に分析する分割の数を指定します。
- この変数の値が分割の数よりも小さい場合、TiDBは分割テーブルのすべての分割を複数のバッチで自動的に分析します。この変数の値が分割よりも大きいか等しい場合、TiDBは分割テーブルのすべての分割を同時に分析します。
- 分割テーブルの分割数がこの変数の値よりもはるかに大きく、自動分析に時間がかかる場合、この変数の値を増やして時間を短縮することができます。

### tidb_auto_analyze_ratio

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 浮動小数点数
- デフォルト値: `0.5`
- レンジ: `[0, 18446744073709551615]`
- この変数は、TiDBがバックグラウンドスレッドでテーブル統計情報を更新するために`ANALYZE TABLE`を自動的に実行する閾値を設定するために使用されます。例えば、0.5の値は、テーブルの行の50%以上が変更された場合に自動分析がトリガーされることを意味します。自動分析は、`tidb_auto_analyze_start_time`と`tidb_auto_analyze_end_time`を指定することで、一日の特定の時間帯にのみ実行されるように制限することができます。

> **注意:**
>
> この機能を使用するには、システム変数`tidb_enable_auto_analyze`を`ON`に設定する必要があります。

### tidb_auto_analyze_start_time

- スコープ: グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 時刻
- デフォルト値: `00:00 +0000`
- この変数は、自動的に統計情報を更新する時間枠を制限するために使用されます。例えば、UTC時間で1時から3時の間に自動統計情報の更新だけを許可する場合は、`tidb_auto_analyze_start_time='01:00 +0000'`および`tidb_auto_analyze_end_time='03:00 +0000'`を設定します。

### tidb_auto_build_stats_concurrency <span class="version-mark">v6.5.0で新規</span>

- スコープ: グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `1`
- レンジ: `[1, 256]`
- この変数は、統計情報の自動的な更新を実行する際の並列性を設定するために使用されます。

### tidb_backoff_lock_fast

- スコープ: セッション | グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `10`
- レンジ: `[1, 2147483647]`
- この変数は、読み取りリクエストがロックに遭遇した場合に`backoff`時間を設定するために使用されます。

### tidb_backoff_weight

- スコープ: セッション | グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `2`
- レンジ: `[0, 2147483647]`
- この変数は、TiDBの`backoff`の最大時間、つまり、内部ネットワークや他のコンポーネント（TiKV、PD）の障害に遭遇した場合にリトライリクエストを送信するための最大リトライ時間を調整するために使用されます。この変数を使用すると、最大リトライ時間を調整し、最小値は1に設定することができます。

    例えば、TiDBがPDからTSOを取得するための基本タイムアウトは15秒です。`tidb_backoff_weight = 2`の場合、TSOを取得するための最大タイムアウトは次の通りです： *基本時間 \* 2 = 30秒*。

    ネットワーク環境が悪い場合は、この変数の値を適切に増やすことでタイムアウトによるアプリケーションエンドへのエラー報告を効果的に緩和することができます。アプリケーションエンドがエラー情報をより速く受け取りたい場合は、この変数の値を最小限に抑えてください。

### tidb_batch_commit

> **警告:**
>
> この変数を有効にすることは**推奨されません**。

- スコープ: セッション
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、非推奨のバッチコミット機能を有効にするかどうかを制御するために使用されます。この変数を有効にすると、トランザクションはいくつかのステートメントをグループ化して非原子的にコミットすることがありますが、これは推奨されません。

### tidb_batch_delete

> **警告:**
>
> この変数は、データの破損を引き起こす可能性がある非推奨のバッチDML機能に関連しています。そのため、バッチDMLのためにこの変数を有効にすることは推奨されません。代わりに、[非トランザクショナル DML](/non-transactional-dml.md)を使用してください。

- スコープ: セッション
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、非推奨のバッチ削除機能を有効にするかどうかを制御するために使用されます。この変数を有効にすると、`DELETE`ステートメントが複数のトランザクションに分割され、非原子的にコミットされる場合があります。この変数を機能させるためには、`tidb_enable_batch_dml`を有効にし、`tidb_dml_batch_size`に正の値を設定する必要がありますが、これは推奨されません。

### tidb_batch_insert

> **警告:**
>
> この変数は、データの破損を引き起こす可能性がある非推奨のバッチDML機能に関連しています。そのため、バッチDMLのためにこの変数を有効にすることは推奨されません。代わりに、[非トランザクショナル DML](/non-transactional-dml.md)を使用してください。

- スコープ: セッション
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、非推奨のバッチ挿入機能を有効にするかどうかを制御するために使用されます。この変数を有効にすると、`INSERT`ステートメントが複数のトランザクションに分割され、非原子的にコミットされる場合があります。この変数を機能させるためには、`tidb_enable_batch_dml`を有効にし、`tidb_dml_batch_size`に正の値を設定する必要がありますが、これは推奨されません。

### tidb_batch_pending_tiflash_count <span class="version-mark">v6.0で新規</span>

- スコープ: セッション | グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `4000`
- レンジ: `[0, 4294967295]`
- この変数は、TiFlashレプリカを追加するために`ALTER DATABASE SET TIFLASH REPLICA`を使用する際の許可される利用不可なテーブルの最大数を指定します。利用不可なテーブルの数がこの制限を超えると、操作は停止するか、残りのテーブルのTiFlashレプリカの設定が非常に遅くなります。

### tidb_broadcast_join_threshold_count <span class="version-mark">v5.0で新規</span>

- スコープ: セッション | グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `10240`
- レンジ: `[0, 9223372036854775807]`
- 単位: 行
- 結合操作のオブジェクトがサブクエリに属する場合、最適化機能はサブクエリ結果セットのサイズを推定できません。この状況では、サイズは結果セットの行数によって決定されます。サブクエリの見積もられた行数がこの変数の値よりも少ない場合、Broadcast Hash Joinアルゴリズムが使用されます。それ以外の場合、Shuffled Hash Joinアルゴリズムが使用されます。
- この変数は、[`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710)を有効にした後は効果を発揮しません。

### tidb_broadcast_join_threshold_size <span class="version-mark">v5.0で新規</span>

- スコープ: セッション | グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `104857600` (100 MiB)
- レンジ: `[0, 9223372036854775807]`
- 単位: バイト
- テーブルサイズがこの変数の値未満の場合、Broadcast Hash Joinアルゴリズムが使用されます。それ以外の場合、Shuffled Hash Joinアルゴリズムが使用されます。
- この変数は、[`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710)を有効にした後は効果を発揮しません。

### tidb_build_stats_concurrency

- スコープ: セッション | グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `2`。デフォルト値はv7.4.0およびそれ以前のバージョンでは`4`です。
- レンジ: `[1, 256]`
- 単位: スレッド
- この変数は、`ANALYZE`ステートメントの実行の並列性を設定するために使用されます。
- この変数を大きな値に設定すると、他のクエリの実行パフォーマンスが影響を受けます。

### tidb_build_sampling_stats_concurrency <span class="version-mark">v7.5.0で新規</span>

- スコープ: グローバル
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- 単位: スレッド
```markdown
- デフォルト値：`2`
- 範囲： `[1, 256]`
- この変数は、`ANALYZE` プロセスでのサンプリング並行性を設定するために使用されます。
- 変数を大きな値に設定すると、他のクエリの実行パフォーマンスに影響を与えます。

### tidb_capture_plan_baselines <span class="version-mark">v4.0 で新規追加</span>

- スコープ：GLOBAL
- クラスターに維持：はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、[baseline capturing](/sql-plan-management.md#baseline-capturing) 機能を有効にするかどうかを制御するために使用されます。この機能はステートメントサマリーに依存しているため、baseline capturing を使用する前にステートメントサマリーを有効にする必要があります。
- この機能を有効にした後、ステートメントサマリー内の過去の SQL ステートメントが定期的にトラバースされ、少なくとも 2 回現れる SQL ステートメントに自動的にバインディングが作成されます。

### tidb_cdc_write_source <span class="version-mark">v6.5.0 で新規追加</span>

> **注意:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では読み取り専用です。

- スコープ：SESSION
- クラスターに維持：いいえ
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`0`
- 範囲：`[0, 15]`
- この変数を 0 以外の値に設定すると、このセッションで書き込まれたデータは TiCDC によって書き込まれたものとみなされます。この変数は、TiCDC によってのみ変更できます。いかなる場合においても、この変数を手動で変更しないでください。

### tidb_check_mb4_value_in_utf8

> **注意:**
>
> この TiDB 変数は TiDB Cloud には適用されません。

- スコープ：GLOBAL
- クラスターに維持：いいえ、接続している現在の TiDB インスタンスのみに適用されます。
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`ON`
- この変数は、`utf8` 文字セットが [Basic Multilingual Plane (BMP)](https://en.wikipedia.org/wiki/Plane_(Unicode)#Basic_Multilingual_Plane) からの値のみを格納するように強制するために使用されます。BMP の外の文字を格納するには、`utf8mb4` 文字セットを使用することが推奨されます。
- このオプションを無効にする必要がある場合は、クラスターを早期の TiDB バージョンからアップグレードするときのように、`utf8` のチェックがより緩和されていた場合に詳細は、[FAQs After Upgrade](https://docs.pingcap.com/tidb/stable/upgrade-faq) を参照してください。

### tidb_checksum_table_concurrency

- スコープ：SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`4`
- 範囲：`[1, 256]`
- 単位: Threads
- この変数は、[`ADMIN CHECKSUM TABLE`](/sql-statements/sql-statement-admin-checksum-table.md) ステートメントを実行する際のスキャンインデックスの並行性を設定するために使用されます。
- 変数を大きな値に設定すると、他のクエリの実行パフォーマンスに影響を与えます。

### tidb_committer_concurrency <span class="version-mark">v6.1.0 で新規追加</span>

- スコープ：GLOBAL
- クラスターに維持：はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`128`
- 範囲： `[1, 10000]`
- シングルトランザクションのコミットフェーズで実行されるリクエストのゴールーチンの数。
- コミットするトランザクションが大きすぎる場合、トランザクションがコミットされるときのフローコントロールキューの待機時間が長すぎるかもしれません。このような状況では、構成値を増やしてコミットを高速化することができます。
- この設定は以前は `tidb.toml` オプション (`performance.committer-concurrency`) でしたが、TiDB v6.1.0 からシステム変数に変更されました。

### tidb_config

> **注意:**
>
> この TiDB 変数は TiDB Cloud には適用されません。

- スコープ：GLOBAL
- クラスターに維持：いいえ、接続している現在の TiDB インスタンスのみに適用されます。
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- デフォルト値：""
- この変数は読み取り専用です。現在の TiDB サーバーの構成情報を取得するために使用されます。

### tidb_constraint_check_in_place

- スコープ：SESSION | GLOBAL
- クラスターに維持：はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は楽観的トランザクションのみに適用されます。悲観的トランザクションの場合は、代わりに [`tidb_constraint_check_in_place_pessimistic`](#tidb_constraint_check_in_place_pessimistic-new-in-v630) を使用してください。
- この変数を `OFF` に設定すると、一意インデックス内の重複値のチェックはトランザクションがコミットされるまで延期されます。これによりパフォーマンスが向上しますが、一部のアプリケーションにとっては予期しない動作となるかもしれません。詳細については、[Constraints](/constraints.md#optimistic-transactions) を参照してください。

    - `tidb_constraint_check_in_place` を `OFF` に設定し、楽観的トランザクションを使用する場合：

        ```sql
        tidb> create table t (i int key);
        tidb> insert into t values (1);
        tidb> begin optimistic;
        tidb> insert into t values (1);
        Query OK, 1 row affected
        tidb> commit; -- トランザクションがコミットされたときにチェックされます。
        ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
        ```

    - `tidb_constraint_check_in_place` を `ON` に設定し、楽観的トランザクションを使用する場合：

        ```sql
        tidb> set @@tidb_constraint_check_in_place=ON;
        tidb> begin optimistic;
        tidb> insert into t values (1);
        ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
        ```

### tidb_constraint_check_in_place_pessimistic <span class="version-mark">v6.3.0 で新規追加</span>

- スコープ：SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値

<CustomContent platform="tidb">

- デフォルト値: デフォルトでは [`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640) の構成項目は `true` なので、この変数のデフォルト値は `ON` です。[`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640) が `false` に設定された場合、この変数のデフォルト値は `OFF` です。

</CustomContent>

<CustomContent platform="tidb-cloud">

- デフォルト値: `ON`

</CustomContent>

- この変数は悲観的トランザクションのみに適用されます。楽観的トランザクションの場合は、代わりに [`tidb_constraint_check_in_place`](#tidb_constraint_check_in_place) を使用してください。
- この変数を `OFF` に設定すると、TiDB は、一意インデックスのユニーク制約チェックを次回にインデックスのロックを必要とするステートメントを実行するか、トランザクションをコミットする時まで延期します。これによりパフォーマンスが向上しますが、一部のアプリケーションにとっては予期しない動作となるかもしれません。詳細については、[Constraints](/constraints.md#pessimistic-transactions) を参照してください。
- この変数を無効にすると、TiDB は悲観的トランザクションで `LazyUniquenessCheckFailure` エラーが発生する可能性があります。このエラーが発生した場合、TiDB は現在のトランザクションをロールバックします。
- この変数を無効にすると、悲観的トランザクションで [`SAVEPOINT`](/sql-statements/sql-statement-savepoint.md) を使用することができません。
- この変数を無効にすると、悲観的トランザクションをコミットする場合、`Write conflict` または `Duplicate entry` エラーが発生する可能性があります。このようなエラーが発生した場合、TiDB は現在のトランザクションをロールバックします。

    - `tidb_constraint_check_in_place_pessimistic` を `OFF` に設定し、悲観的トランザクションを使用する場合：

        {{< copyable "sql" >}}

        ```sql
        set @@tidb_constraint_check_in_place_pessimistic=OFF;
        create table t (i int key);
        insert into t values (1);
        begin pessimistic;
        insert into t values (1);
        ```

        ```
        Query OK, 1 row affected
        ```

        ```sql
        tidb> commit; -- トランザクションがコミットされるときにチェックされます。
        ```

        ```
        ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
        ```

    - `tidb_constraint_check_in_place_pessimistic` を `ON` に設定し、悲観的トランザクションを使用する場合：

        ```sql
        set @@tidb_constraint_check_in_place_pessimistic=ON;
        begin pessimistic;
        insert into t values (1);
        ```

        ```
        ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
        ```
```
### tidb_cost_model_version <span class="version-mark">v6.2.0で新規追加</span>

> **注意:**
>
> - TiDB v6.5.0以降、新しく作成されたクラスターではデフォルトでCost Model Version 2が使用されます。TiDB v6.5.0以前のバージョンからv6.5.0以降にアップグレードすると、`tidb_cost_model_version`の値は変更されません。
> - コストモデルのバージョンを切り替えると、クエリプランに変更が生じる可能性があります。

- 対象: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `2`
- 値のオプション:
    - `1`: TiDB v6.4.0およびそれ以前のバージョンでデフォルトで使用されているコストモデルバージョン1を有効にします。
    - `2`: TiDB v6.5.0で一般提供されており、内部テストではバージョン1よりも精度が高い[Cost Model Version 2](/cost-model.md#cost-model-version-2)を有効にします。
- コストモデルのバージョンは、最適化プランの決定に影響を与えます。詳細は、[Cost Model](/cost-model.md)を参照してください。

### tidb_current_ts

- 対象: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 9223372036854775807]`
- この変数は読み取り専用です。現在のトランザクションのタイムスタンプを取得するために使用されます。

### tidb_ddl_disk_quota <span class="version-mark">v6.3.0で新規追加</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- 対象: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `107374182400` (100 GiB)
- 範囲: `[107374182400, 1125899906842624]` ([100 GiB, 1 PiB])
- 単位: バイト
- この変数は、[`tidb_ddl_enable_fast_reorg`](#tidb_ddl_enable_fast_reorg-new-in-v630)が有効な場合にのみ効果があります。これにより、インデックスの作成時のバックフィリング中のローカルストレージの使用制限を設定します。

### tidb_ddl_enable_fast_reorg <span class="version-mark">v6.3.0で新規追加</span>

> **注意:**
>
> - この変数を使用してインデックスの作成の加速を行う際に、TiDB Dedicatedクラスターを使用している場合は、TiDBクラスターがAWSでホストされ、TiDBノードのサイズが少なくとも8 vCPUであることを確認してください。
> - [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは、この変数は読み取り専用です。

- 対象: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、`ADD INDEX`および`CREATE INDEX`の加速を有効にするかどうかを制御します。この変数の値を`ON`に設定することで、大量のデータを持つテーブルでのインデックス作成のパフォーマンスを向上させることができます。
- v7.1.0以降、インデックスの加速操作はチェックポイントをサポートしています。TiDBがオーナーノードが失敗によって再起動または変更された場合でも、TiDBは定期的に自動更新されるチェックポイントから進行状況を回復できます。
- 完了した `ADD INDEX` の操作が加速されているかどうかを確認するには、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md#admin-show-ddl-jobs) ステートメントを実行して、`JOB_TYPE`列に`ingest`が表示されるかどうかを確認できます。

<CustomContent platform="tidb">

> **警告:**
>
> 現在、PITRリカバリは、インデックスの加速によって作成されたインデックスをログバックアップ中に互換性を実現するために追加処理を行います。詳細については、GitHubの[Why is the acceleration of adding indexes feature incompatible with PITR?](/faq/backup-and-restore-faq.md#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr)を参照してください。

> **注意:**
>
> * インデックスの加速には、書き込み可能で十分な空き容量がある[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)が必要です。`temp-dir`を使用できない場合、TiDBは非加速のインデックス構築にフォールバックします。SSDディスクに`temp-dir`を配置することをお勧めします。
>
> * TiDBをv6.5.0以降にアップグレードする前に、TiDBの[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)のパスが正しくSSDディスクにマウントされているかどうかを確認することをお勧めします。TiDBを実行するオペレーティングシステムユーザーがこのディレクトリに対して読み取りおよび書き込みの権限を持っていることを確認してください。そうしないと、DDL操作に予測不可能な問題が発生する可能性があります。このパスはTiDB構成項目であり、TiDBが再起動した後に有効になります。したがって、アップグレード前にこの構成項目を設定することで、別の再起動を回避できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **警告:**
>
> 現在、この機能は、単一の`ALTER TABLE`ステートメントで複数の列またはインデックスを変更することと互換性がありません。インデックスの加速を行う場合は、同じステートメントで他の列やインデックスを変更しないようにしてください。
>
> 現在、PITRリカバリは、インデックスの加速によって作成されたインデックスをログバックアップ中に互換性を実現するために追加処理を行います。詳細については、[Why is the acceleration of adding indexes feature incompatible with PITR?](https://docs.pingcap.com/tidb/v7.0/backup-and-restore-faq#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr)を参照してください。

</CustomContent>

### tidb_enable_dist_task <span class="version-mark">v7.1.0で新規追加</span>

- 対象: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `OFF`
- この変数は、[TiDBバックエンドタスク分散実行フレームワーク](/tidb-distributed-execution-framework.md)を有効にするかどうかを制御するために使用されます。フレームワークを有効にした場合、DDLやインポートなどのバックエンドタスクは、クラスター内の複数のTiDBノードによって分散して実行および完了されます。
- TiDB v7.1.0以降では、フレームワークはパーティションされたテーブルに対して[`ADD INDEX`](/sql-statements/sql-statement-add-index.md)ステートメントの分散実行をサポートします。
- TiDB v7.2.0以降では、フレームワークは [`IMPORT INTO`](https://docs.pingcap.com/tidb/v7.2/sql-statement-import-into) ステートメントのTiDB自己ホスト型インポートジョブの分散実行をサポートします。TiDB Cloudの場合、`IMPORT INTO`ステートメントは適用されません。
- この変数は`tidb_ddl_distribute_reorg`から改名されました。

### tidb_cloud_storage_uri <span class="version-mark">v7.4.0で新規追加</span>

> **警告:**
>
> この機能は実験的です。本番環境で使用しないことを推奨します。この機能は事前の通知なしに変更または削除される場合があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

- 対象: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `""`

<CustomContent platform="tidb">

- この変数は、[グローバルソート](/tidb-global-sort.md)を有効にするためにAmazon S3クラウドストレージURIを指定するために使用されます。分散実行フレームワークを有効にした後、必要な権限でクラウドストレージのパスを指定することで、グローバルソート機能を使用することができます。詳細は、[Amazon S3のURI形式](/external-storage-uri.md#amazon-s3-uri-format)を参照してください。
- 以下のステートメントで、グローバルソート機能を使用できます。
    - [`ADD INDEX`](/sql-statements/sql-statement-add-index.md) ステートメント。
    - TiDB自己ホスト型インポートジョブのための [`IMPORT INTO`](/sql-statements/sql-statement-import-into.md) ステートメント。TiDB Cloudの場合、`IMPORT INTO`ステートメントは適用されません。

</CustomContent>
<CustomContent platform="tidb-cloud">

- この変数は、[Global Sort](/tidb-global-sort.md)を有効にするためのクラウドストレージURIを指定するために使用されます。[distributed execution framework](/tidb-distributed-execution-framework.md) を有効にした後、必要な権限でクラウドストレージへのアクセス権を持つ適切なクラウドストレージパスを構成することで、URIを構成し、Global Sort機能を使用できます。詳細については、[URI Formats of External Storage Services](https://docs.pingcap.com/tidb/stable/external-storage-uri)を参照してください。

- 以下のステートメントでGlobal Sort機能を使用できます。
    - [`ADD INDEX`](/sql-statements/sql-statement-add-index.md) ステートメント。
    - TiDB Self-Hostedのインポートジョブの `IMPORT INTO` ステートメント。TiDB Cloudの場合、`IMPORT INTO` ステートメントは適用されません。

</CustomContent>

### tidb_ddl_error_count_limit

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。


- スコープ: GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `512`
- 範囲: `[0, 9223372036854775807]`
- この変数は、DDL操作が失敗した場合のリトライ回数を設定するために使用されます。 リトライ回数がパラメータ値を超過すると、誤ったDDL操作がキャンセルされます。

### tidb_ddl_flashback_concurrency <span class="version-mark">v6.3.0で追加</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `64`
- 範囲: `[1, 256]`
- この変数は、[`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)の並列度を制御します。

### tidb_ddl_reorg_batch_size

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数  
- デフォルト値: `256`
- 範囲: `[32, 10240]`
- 単位: 行
- この変数は、DDL操作の `re-organize` フェーズ中のバッチサイズを設定するために使用されます。 たとえば、TiDBが `ADD INDEX` 操作を実行するとき、インデックスデータは `tidb_ddl_reorg_worker_cnt`（数値）の並列ワーカーによってバックフィルされます。 各ワーカーはバッチでインデックスデータをバックフィルします。
    - `UPDATE` や `REPLACE` などの多くの更新操作が `ADD INDEX` 操作中に存在する場合、大きなバッチサイズはトランザクションの競合の確率を高めます。 この場合、バッチサイズを小さな値に調整する必要があります。 最小値は32です。
    - トランザクションの競合が存在しない場合、バッチサイズを大きな値に設定できます（ワーカー数を考慮してください。 参照のために[Interaction Test on Online Workloads and `ADD INDEX` Operations](https://docs.pingcap.com/tidb/stable/online-workloads-and-add-index-operations)を参照してください）。 これにより、データのバックフィル速度が向上しますが、TiKVへの書き込み圧力も高くなります。

### tidb_ddl_reorg_priority

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 列挙型
- デフォルト値: `PRIORITY_LOW`
- 値のオプション: `PRIORITY_LOW`、`PRIORITY_NORMAL`、`PRIORITY_HIGH`
- この変数は、`re-organize` フェーズにおける `ADD INDEX` 操作の優先度を設定するために使用されます。
- この変数の値を`PRIORITY_LOW`、`PRIORITY_NORMAL`、または`PRIORITY_HIGH`に設定できます。

### tidb_ddl_reorg_worker_cnt

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `4`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`re-organize` フェーズにおけるDDL操作の並列性を設定するために使用されます。

### tidb_default_string_match_selectivity <span class="version-mark">v6.2.0で追加</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- タイプ: 浮動小数
- デフォルト値: `0.8`
- 範囲: `[0, 1]`
- この変数は、フィルタ条件での `like`、`rlike`、および `regexp` 関数のデフォルトの選択率を設定するために使用されます。 この変数はまた、これらの関数の推定を支援するためにTopNを有効にするかどうかも制御します。
- TiDBは統計を使用して `like` をフィルタ条件で推定しようとします。 しかし、`like` が複雑な文字列に一致する場合、または `rlike` や `regexp` を使用する場合、TiDBはしばしば統計を完全に使用できないため、デフォルト値`0.8`は選択率として設定され、不正確な推定結果をもたらします。
- この変数を変更することで、前述の動作を変更することができます。 変数を `0` 以外の値に設定すると、選択率は指定された変数値になります。
- 変数が `0` に設定されている場合、TiDBは統計でTopNを使用して評価し、性能にわずかに影響を与えるかもしれないが、前述の3つの関数の推定を改善するためにNULL数を考慮します。 前提条件は、[`tidb_analyze_version`](#tidb_analyze_version-new-in-v510)が `2` に設定されているということです。
- 変数が `0.8` 以外の値に設定されている場合、TiDBは `not like`、`not rlike`、および `not regexp` の推定をそれに応じて調整します。

### tidb_disable_txn_auto_retry

- スコープ: SESSION | GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、明示的な楽観的トランザクションの自動リトライを無効にするかどうかを設定するために使用されます。 `ON` のデフォルト値は、TiDBではトランザクションが自動的にリトライされず、`COMMIT` ステートメントからエラーが返される可能性があることを意味します。 

    値を `OFF` に設定すると、TiDBはトランザクションを自動的にリトライし、`COMMIT` ステートメントからエラーが少なくなります。 この変更を行う際は注意してください、なぜなら更新が失われる可能性があるからです。

    この変数は、自動的にコミットされる暗黙的なトランザクションおよびTiDBで内部的に実行されるトランザクションには影響しません。 これらのトランザクションの最大リトライ回数は、`tidb_retry_limit`の値によって決まります。

    詳細については [limits of retry](/optimistic-transaction.md#limits-of-retry) を参照してください。

    <CustomContent platform="tidb">

    この変数は楽観的なトランザクションにのみ適用され、悲観的なトランザクションには適用されません。 悲観的なトランザクションのリトライ回数は[`max_retry_count`](/tidb-configuration-file.md#max-retry-count)によって制御されます。

    </CustomContent>

    <CustomContent platform="tidb-cloud">

    この変数は楽観的なトランザクションにのみ適用され、悲観的なトランザクションには適用されません。 悲観的なトランザクションのリトライ回数は256です。

    </CustomContent>

### tidb_distsql_scan_concurrency

- スコープ: SESSION | GLOBAL
- クラスタに永続化する: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `15`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`scan` 操作の並列性を設定するために使用されます。
- OLAPシナリオでは大きな値を使用し、OLTPシナリオでは小さな値を使用します。
- OLAPシナリオでは、最大値はすべてのTiKVノードのCPUコア数を超えないようにする必要があります。
- テーブルに多くのパーティションがある場合、変数値を適切に減らすことができます（スキャンするデータのサイズとスキャンの頻度によって決まります）でTIKVがメモリ不足になるのを避けるためです。
> この変数は廃止予定のバッチdml機能と関連しており、データの破損を引き起こす可能性があります。そのため、バッチdmlにこの変数を有効にすることはお勧めしません。代わりに[非トランザクショナルDML](/non-transactional-dml.md)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- 単位: 行
- この値が`0`よりも大きい場合、TiDBは`INSERT`などのステートメントをより小さいトランザクションにバッチコミットします。これによりメモリ使用量が削減され、`txn-total-size-limit`が一括変更によって超過されないようになります。
- 値`0`の場合のみACID準拠になります。この値を他のどの値に設定すると、TiDBの原子性と分離性の保証が壊れます。
- この変数を機能させるためには、`tidb_enable_batch_dml`を有効にし、`tidb_batch_insert`および`tidb_batch_delete`の少なくとも1つを有効にする必要があります。

> **注記:**
>
> v7.0.0からは、[`LOAD DATA`ステートメント](/sql-statements/sql-statement-load-data.md)に`tidb_dml_batch_size`はもはや効果がありません。

### tidb_enable_1pc <span class="version-mark">v5.0での新機能</span>

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、1つのリージョンに影響を与えるトランザクションの1段階コミット機能を有効にするかどうかを指定するために使用されます。よく使用される2段階コミットと比較して、1段階コミットではトランザクションコミットの待ち時間が大幅に短縮され、スループットが向上します。

> **注記:**
>
> - `ON`のデフォルト値は新規クラスターにのみ適用されます。クラスターが旧バージョンのTiDBからのアップグレードの場合、代わりに値`OFF`が使用されます。
> - TiDB Binlogを有効にしている場合、この変数を有効にしてもパフォーマンスは向上しません。パフォーマンスを向上させるには、[TiCDC](https://docs.pingcap.com/tidb/stable/ticdc-overview)を使用することをお勧めします。
> - このパラメータを有効にすると、1段階コミットがトランザクションコミットのオプションモードになります。実際には、最適なトランザクションコミットモードはTiDBによって決定されます。

### tidb_enable_analyze_snapshot <span class="version-mark">v6.2.0での新機能</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`ANALYZE`を実行する際に履歴データまたは最新データを読むかを制御します。この変数が`ON`に設定されている場合、`ANALYZE`は`ANALYZE`の時点で利用可能な履歴データを読み取ります。この変数が`OFF`に設定されている場合、`ANALYZE`は最新データを読み取ります。
- v5.2以前は`ANALYZE`は最新データを読み取ります。v5.2からv6.1まで、`ANALYZE`は`ANALYZE`の時点で利用可能な履歴データを読み取ります。

> **警告:**
>
> もし`ANALYZE`が`ANALYZE`の時点で利用可能な履歴データを読み取る場合、`AUTO ANALYZE`の時間が長引くと`GC life time is shorter than transaction duration`エラーが発生する可能性があります。なぜならば、履歴データがガベージコレクトされるからです。

### tidb_enable_async_commit <span class="version-mark">v5.0での新機能</span>

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、2段階トランザクションコミットの第2段階を非同期でバックグラウンドで実行するための非同期コミット機能を有効にするかどうかを制御します。この機能を有効にすると、トランザクションコミットの待ち時間が短縮されます。

> **注記:**
>
> - `ON`のデフォルト値は新規クラスターにのみ適用されます。クラスターが旧バージョンのTiDBからのアップグレードの場合、代わりに値`OFF`が使用されます。
> - TiDB Binlogを有効にしている場合、この変数を有効にしてもパフォーマンスは向上しません。パフォーマンスを向上させるには、[TiCDC](https://docs.pingcap.com/tidb/stable/ticdc-overview)を使用することをお勧めします。
> - このパラメータを有効にすると、非同期コミットがトランザクションコミットのオプションモードになります。実際には、最適なトランザクションコミットモードはTiDBによって決定されます。

### tidb_enable_auto_analyze <span class="version-mark">v6.1.0での新機能</span>

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- TiDBがバックグラウンドで自動的にテーブルの統計情報を更新するかどうかを決定します。
- この設定は以前は`tidb.toml`のオプション(`performance.run-auto-analyze`)でしたが、TiDB v6.1.0からはシステム変数に変更されました。

### tidb_enable_auto_increment_in_generated

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、生成列または式のインデックスを作成する際に`AUTO_INCREMENT`列を含めるかどうかを決定します。

### tidb_enable_batch_dml

> **警告:**
>
> この変数は廃止予定のバッチdml機能と関連しており、データの破損を引き起こす可能性があります。そのため、バッチdmlにこの変数を有効にすることはお勧めしません。代わりに[非トランザクショナルDML](/non-transactional-dml.md)を使用してください。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、廃止予定のバッチdml機能を有効にするかを制御します。有効にすると、特定のステートメントが複数のトランザクションに分割される場合があり、これは非原子的で注意して使用する必要があります。バッチdmlを使用する際には、操作中のデータに競合する操作がないようにする必要があります。機能させるためには、`tidb_batch_dml_size`に正の値を指定し、`tidb_batch_insert`および`tidb_batch_delete`の少なくとも1つを有効にする必要があります。

### tidb_enable_cascades_planner

> **警告:**
>
> 現在、カスケードプランナーは実験的な機能です。本番環境で使用しないことをお勧めします。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、カスケードプランナーを有効にするかどうかを制御します。

### tidb_enable_check_constraint <span class="version-mark">v7.2.0での新機能</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、[`CHECK` constraint](/constraints.md#check)機能を有効にするかどうかを制御します。

### tidb_enable_chunk_rpc <span class="version-mark">v4.0での新機能</span>

- スコープ: SESSION
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、Coprocessorで`Chunk`データエンコード形式を有効にするかどうかを制御します。

### tidb_enable_clustered_index <span class="version-mark">v5.0での新機能</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 列挙型
- デフォルト値: `ON`
- 可能な値: `OFF`, `ON`, `INT_ONLY`
- この変数は、デフォルトでプライマリキーを[クラスタリングインデックス](/clustered-indexes.md)として作成するかどうかを制御するために使用されます。ここでの「デフォルト」とは、明示的にキーワード'CLUSTERED'/'NONCLUSTERED'を指定しない場合を指します。サポートされている値は`OFF`、`ON`、`INT_ONLY`です:

    - `OFF`：プライマリキーはデフォルトで非クラスタリングインデックスとして作成されます。
    - `ON`：プライマリキーはデフォルトでクラスタリングインデックスとして作成されます。
    - `INT_ONLY`：動作は構成項目`alter-primary-key`によって制御されます。`alter-primary-key`が`true`に設定されている場合、すべてのプライマリキーはデフォルトで非クラスタリングインデックスとして作成されます。`false`に設定されている場合、整数列から構成されるプライマリキーのみクラスタリングインデックスとして作成されます。

### tidb_enable_ddl <span class="version-mark">v6.3.0 で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: いいえ、接続している現在の TiDB インスタンスにのみ適用されます。
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- デフォルト値: `ON`
- 可能な値: `OFF`、`ON`
- この変数は対応する TiDB インスタンスが DDL オーナーになれるかどうかを制御します。現在の TiDB クラスタに TiDB インスタンスが1つしかない場合、それをDDLオーナーにすることを防ぐことはできません。

### tidb_enable_collect_execution_info

> **注意:**
>
> この TiDB 変数は TiDB Cloud には適用されません。

- スコープ: GLOBAL
- クラスタに永続化: いいえ、接続している現在の TiDB インスタンスにのみ適用されます。
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は遅いクエリログ内の各オペレータの実行情報を記録するかどうかを制御します。

### tidb_enable_column_tracking <span class="version-mark">v5.4.0 で新規</span>

> **警告:**
>
> 現在、`PREDICATE COLUMNS`の統計情報の収集は実験的な機能です。本番環境で使用しないことをお勧めします。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は TiDB が `PREDICATE COLUMNS` の収集を有効にするかどうかを制御します。収集を有効にした後、それを無効にすると、以前に収集された `PREDICATE COLUMNS` の情報がクリアされます。詳細については、[一部のカラムの統計情報を収集](/statistics.md#collect-statistics-on-some-columns)を参照してください。

### tidb_enable_enhanced_security

- スコープ: NONE
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値

<CustomContent platform="tidb">

- デフォルト値: `OFF`
- この変数は、接続している TiDB サーバーがセキュリティ強化モード（SEM）が有効かどうかを示します。値を変更するには、TiDB サーバーの構成ファイル内の`enable-sem`の値を変更して、TiDB サーバーを再起動する必要があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- デフォルト値: `ON`
- この変数は読み取り専用です。TiDB Cloud では、セキュリティ強化モード（SEM）がデフォルトで有効になっています。

</CustomContent>

- SEMは、[セキュリティ強化 Linux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux)などのシステムの設計からインスピレーションを受けています。MySQLの`SUPER`特権を持つユーザーの機能を削減し、代わりに`RESTRICTED`の細かい権限を付与するよう求めます。これらの細かい権限には次のものが含まれます：
    - `RESTRICTED_TABLES_ADMIN`：`mysql`スキーマのシステムテーブルにデータを書き込んだり、`information_schema`テーブルの機密カラムを見る権限。
    - `RESTRICTED_STATUS_ADMIN`：`SHOW STATUS`で機密変数を見る権限。
    - `RESTRICTED_VARIABLES_ADMIN`：`SHOW [GLOBAL] VARIABLES`および`SET`で機密変数を見る権限。
    - `RESTRICTED_USER_ADMIN`：他のユーザーによる変更やユーザーアカウントの削除を防ぐ権限。

### tidb_enable_exchange_partition

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、[テーブルとのパーティションの交換](/partitioned-table.md#partition-management)機能を有効にするかどうかを制御します。デフォルト値は`ON`で、つまり、`テーブルとのパーティションの交換`はデフォルトで有効になります。
- この変数は v6.3.0 以降廃止されました。その値はデフォルト値である`ON`に固定され、つまり`テーブルとのパーティションの交換`はデフォルトで有効になります。

### tidb_enable_extended_stats

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、TiDBが最適化をガイドするための拡張統計情報を収集できるかどうかを示します。詳細については、[拡張統計情報の概要](/extended-statistics.md)を参照してください。

### tidb_enable_external_ts_read <span class="version-mark">v6.4.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数が`ON`に設定されている場合、TiDBは[`tidb_external_ts`](#tidb_external_ts-new-in-v640)で指定されたタイムスタンプでデータを読み取ります。

### tidb_external_ts <span class="version-mark">v6.4.0 で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `0`
- [`tidb_enable_external_ts_read`](#tidb_enable_external_ts_read-new-in-v640)が`ON`に設定されている場合、TiDBはこの変数で指定されたタイムスタンプでデータを読み取ります。

### tidb_enable_fast_analyze

> **警告:**
>
> v7.5.0から、この変数は非推奨となりました。

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は統計`Fast Analyze`機能の有効化を設定するために使用されます。
- 統計`Fast Analyze`機能が有効になっていると、TiDBはデータの約10,000行をサンプリングします。データが均等に分布していない場合やデータサイズが小さい場合、統計の精度が低くなることがあります。これにより、正しくない実行計画が選択される可能性があります。通常の`Analyze`ステートメントの実行時間が許容できる場合、`Fast Analyze`機能を無効にすることをお勧めします。

### tidb_enable_fast_table_check <span class="version-mark">v7.2.0 で新規</span>

> **注意:**
>
> この変数は[多値インデックス](/sql-statements/sql-statement-create-index.md#multi-valued-indexes)やプレフィックスインデックスには適用されません。

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、テーブルのデータとインデックスの整合性を迅速にチェックするためにチェックサムを使用する方法を制御します。デフォルト値`ON`は、この機能がデフォルトで有効になっていることを意味します。
- この変数が有効になっている場合、TiDBは[`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md)ステートメントをより迅速に実行できます。

### tidb_enable_foreign_key <span class="version-mark">v6.3.0 で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: v6.6.0以前はデフォルト値は`OFF`、v6.6.0以降はデフォルト値は`ON`です。
- この変数は`FOREIGN KEY`機能を有効にするかどうかを制御します。

### tidb_enable_gc_aware_memory_track

> **警告:**
>
> この変数は TiDB のデバッグ用の内部変数です。将来のリリースで削除される可能性があります。**この変数は**設定しないでください。

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数はGC-Awareメモリトラッキングを有効にするかどうかを制御します。

### tidb_enable_non_prepared_plan_cache

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数は[Non-prepared plan cache](/sql-non-prepared-plan-cache.md)機能を有効にするかどうかを制御します。この機能を有効にすると、追加のメモリおよびCPUオーバーヘッドが発生する可能性があり、すべての状況に適しているわけではありません。実際のシナリオに応じてこの機能を有効にするかどうかを判断してください。

### tidb_enable_non_prepared_plan_cache_for_dml <span class="version-mark">v7.1.0で新規追加</span>

> **警告:**
>
> DMLステートメントのための非準備済み実行計画キャッシュは実験的な機能です。本番環境で使用しないでください。この機能は予告なく変更される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数はDMLステートメントのための[Non-prepared plan cache](/sql-non-prepared-plan-cache.md)機能を有効にするかどうかを制御します。

### tidb_enable_gogc_tuner <span class="version-mark">v6.4.0で新規追加</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数はGOGC Tunerを有効にするかどうかを制御します。

### tidb_enable_historical_stats

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数は歴史統計情報を有効にするかどうかを制御します。デフォルト値は`OFF`から`ON`に変更され、つまり歴史統計情報がデフォルトで有効になります。

### tidb_enable_historical_stats_for_capture

> **警告:**
>
> この変数が制御する機能は現在のTiDBバージョンで完全に機能していません。デフォルト値を変更しないでください。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数は`PLAN REPLAYER CAPTURE`によってキャプチャされる情報にデフォルトで歴史統計を含めるかどうかを制御します。デフォルト値の`OFF`は、歴史統計がデフォルトで含まれないことを意味します。

### tidb_enable_index_merge <span class="version-mark">v4.0で新規追加</span>

> **注意:**
>
> - TiDBクラスターをv4.0.0より前のバージョンからv5.4.0以降にアップグレードした後、この変数は実行計画の変更によるパフォーマンスの低下を防ぐためにデフォルトで無効になります。
>
> - TiDBクラスターをv4.0.0以降からv5.4.0以降にアップグレードした後、この変数はアップグレード前の設定のままです。
>
> - v5.4.0以降、新しくデプロイされたTiDBクラスターでは、この変数はデフォルトで有効になります。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数はインデックスマージ機能を有効にするかどうかを制御します。

### tidb_enable_index_merge_join

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 真偽値
- デフォルト値: `OFF`
- `IndexMergeJoin`オペレーターを有効にするかどうかを指定します。
- この変数はTiDBの内部操作にのみ使用されます。調整することは**推奨されません**。それ以外の場合、データの正確性に影響を与える可能性があります。

### tidb_enable_legacy_instance_scope <span class="version-mark">v6.0.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数は`INSTANCE`スコープの変数を`SET SESSION`および`SET GLOBAL`の構文を使用して設定することを許可します。
- このオプションは、以前のバージョンのTiDBとの互換性のためにデフォルトで有効になっています。

### tidb_enable_list_partition <span class="version-mark">v5.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数は`LIST（COLUMNS）TABLE PARTITION`機能を有効にするかどうかを設定します。

### tidb_enable_local_txn

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数はリリースされていない機能に使用されます。変数の値を変更しないでください。

### tidb_enable_metadata_lock <span class="version-mark">v6.3.0で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数は[Metadata lock](/metadata-lock.md)機能を有効にするかどうかを設定します。この変数を設定する際には、クラスターに実行中のDDLステートメントがないことを確認する必要があります。そうでない場合、データが不正確または一貫性がない可能性があります。

### tidb_enable_mutation_checker <span class="version-mark">v6.0.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数はTiDBミューテーションチェッカーを有効にするかどうかを制御します。これは、DMLステートメントの実行中にデータとインデックスの一貫性を確認するために使用されるツールです。チェッカーがステートメントに対してエラーを返した場合、TiDBはステートメントの実行をロールバックします。この変数を有効にすると、CPU使用率がわずかに増加します。詳細については、[データとインデックスの不一致のトラブルシューティング](/troubleshoot-data-inconsistency-errors.md)を参照してください。新しいバージョンのクラスターの場合、デフォルト値は`ON`です。v6.0.0以前のバージョンからアップグレードした既存のクラスターのデフォルト値は`OFF`です。

### tidb_enable_new_cost_interface <span class="version-mark">v6.2.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- TiDB v6.2.0は以前のコストモデルの実装を再構築しました。この変数は再構築されたコストモデルの実装を有効にするかどうかを制御します。
- この変数は以前と同じコスト式を使用するため、プランの決定が変わらないため、デフォルトで有効になっています。
- クラスターがv6.1からv6.2にアップグレードされると、この変数は引き続き`OFF`になります。手動で有効にすることをお勧めします。クラスターがv6.1より前のバージョンからアップグレードされる場合、この変数はデフォルトで`ON`に設定されます。

### tidb_enable_new_only_full_group_by_check <span class="version-mark">v6.1.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数は、TiDBが`ONLY_FULL_GROUP_BY`チェックを実行する際の挙動を制御します。`ONLY_FULL_GROUP_BY`の詳細については、[MySQLドキュメント](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by)を参照してください。v6.1.0では、TiDBはこのチェックをより厳密かつ正確に処理します。
- バージョンアップによる互換性の問題を避けるため、この変数のデフォルト値はv6.1.0では`OFF`です。

### tidb_enable_noop_functions <span class="version-mark">v4.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)なし
- タイプ：列挙型
- デフォルト値：`OFF`
- 可能な値：`OFF`、`ON`、`WARN`
- デフォルトでは、TiDBはまだ実装されていない機能の構文を使用しようとするとエラーを返します。変数値を`ON`に設定すると、TiDBはそのような利用できない機能をサイレントに無視するため、SQLコードを変更できない場合に役立ちます。
- `noop`関数を有効にすると、以下の動作が制御されます：
    * `LOCK IN SHARE MODE`構文
    * `SQL_CALC_FOUND_ROWS`構文
    * `START TRANSACTION READ ONLY`および`SET TRANSACTION READ ONLY`構文
    * `tx_read_only`、`transaction_read_only`、`offline_mode`、`super_read_only`、`read_only`および`sql_auto_is_null`システム変数
    * `GROUP BY <expr> ASC|DESC`構文

> **警告:**
>
> デフォルト値の`OFF`のみが安全と見なされます。`tidb_enable_noop_functions=1`に設定すると、TiDBがエラーを表示せずに特定の構文を無視する可能性があります。たとえば、`START TRANSACTION READ ONLY`構文は許可されていますが、トランザクションは読み取り専用モードのままです。

### tidb_enable_noop_variables <span class="version-mark">v6.2.0で新規追加</span>

- スコープ：グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)なし
- デフォルト値：`ON`
- 変数値を`OFF`に設定すると、TiDBの挙動は次のようになります：
    * `noop`変数を設定するために`SET`を使用すると、TiDBは`"setting *変数名* has no effect in TiDB"`の警告を返します。
    * `SHOW [SESSION | GLOBAL] VARIABLES`の結果に`noop`変数が含まれなくなります。
    * `noop`変数を読み取るために`SELECT`を使用すると、TiDBは`"variable *変数名* has no effect in TiDB"`の警告を返します。
- TiDBインスタンスが`noop`変数を設定および読み取ったかどうかを確認するには、`SELECT * FROM INFORMATION_SCHEMA.CLIENT_ERRORS_SUMMARY_GLOBAL;`ステートメントを使用できます。

### tidb_enable_null_aware_anti_join <span class="version-mark">v6.3.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)あり
- デフォルト値：v7.0.0以前、デフォルト値は`OFF`です。v7.0.0以降、デフォルト値は`ON`です。
- タイプ：真偽
- この変数は、特定のセット演算子によって導かれるサブクエリでANTI JOINが生成された場合に、TiDBがNull Aware Hash Joinを適用するかどうかを制御します。
- 以前のバージョンからv7.0.0またはそれ以降のクラスタにアップグレードすると、この機能は自動的に有効になるため、この変数は`ON`に設定されます。

### tidb_enable_outer_join_reorder  <span class="version-mark">v6.1.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)あり
- タイプ：真偽
- デフォルト値：`ON`
- TiDBの[Join Reorder](/join-reorder.md)アルゴリズムはv6.1.0以降、Outer Joinをサポートするようになりました。この変数は、TiDBがJoin ReorderのOuter Joinサポートを有効にするかどうかを制御します。
- TiDBのクラスタが以前のバージョンからアップグレードされる場合は、次の点に注意してください：

    - アップグレード前のTiDBバージョンがv6.1.0よりも古い場合、アップグレード後のこの変数のデフォルト値は`ON`です。
    - アップグレード前のTiDBバージョンがv6.1.0以上の場合、アップグレード後のデフォルト値は、アップグレード前の値に従います。

### `tidb_enable_inl_join_inner_multi_pattern` <span class="version-mark">v7.0.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)あり
- タイプ：真偽
- デフォルト値：`OFF`
- この変数は、内部テーブルに`Selection`または`Projection`演算子がある場合にIndex Joinがサポートされるかどうかを制御します。デフォルト値の`OFF`は、このシナリオでのIndex Joinがサポートされていないことを意味します。

### tidb_enable_ordered_result_mode

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)あり
- タイプ：真偽
- デフォルト値：`OFF`
- 最終出力結果を自動的にソートするかどうかを指定します。
- たとえば、この変数を有効にすると、TiDBは`SELECT a, MAX(b) FROM t GROUP BY a`を`SELECT a, MAX(b) FROM t GROUP BY a ORDER BY a, MAX(b)`として処理します。

### tidb_enable_paging <span class="version-mark">v5.4.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)あり
- タイプ：真偽
- デフォルト値：`ON`
- この変数は、ページングメソッドを使用してコプロセッサリクエストを送信するかどうかを制御します。v5.4.0からv6.2.0のTiDBバージョンでは、この変数は`IndexLookup`演算子にのみ影響します。v6.2.0以降のバージョンでは、この変数はグローバルに影響します。v6.4.0からは、この変数のデフォルト値が`OFF`から`ON`に変更されました。
- ユーザーシナリオ：

    - すべてのOLTPシナリオでは、ページングメソッドを使用することが推奨されます。
    - `IndexLookup`および`Limit`を使用する読み取りクエリで`Limit`が`IndexScan`にプッシュダウンされない場合、読み取りクエリの待機時間が長くなり、TiKV「Unified read pool CPU」の使用量が高くなることがあります。このような場合、`Limit`演算子は少量のデータのみを必要とするため、[`tidb_enable_paging`](#tidb_enable_paging-new-in-v540)を`ON`に設定すると、TiDBは少ないデータを処理し、クエリの待機時間とリソース消費を削減できます。
    - [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)を使用したデータエクスポートやフルテーブルスキャンなどのシナリオでは、ページングを有効にすると、TiDBプロセスのメモリ消費を効果的に削減できます。

> **注意:**
>
> TiFlashの代わりにストレージエンジンとしてTiKVを使用するOLAPシナリオでは、ページングを有効にすると、一部の場合でパフォーマンスが低下する可能性があります。低下が発生した場合は、ページングを無効にするためにこの変数を使用するか、[`tidb_min_paging_size`](/system-variables.md#tidb_min_paging_size-new-in-v620)および[`tidb_max_paging_size`](/system-variables.md#tidb_max_paging_size-new-in-v630)を使用してページングサイズの範囲を調整することを検討してください。

### tidb_enable_parallel_apply <span class="version-mark">v5.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)なし
- タイプ：真偽
- デフォルト値：`OFF`
- この変数は、`Apply`演算子の並行性を有効にするかどうかを制御します。並行数は`tidb_executor_concurrency`変数で制御されます。`Apply`演算子はデフォルトで相関サブクエリを処理し、デフォルトでは並行性がありません。変数値を`1`に設定すると、並行性が増加し、実行速度が向上します。現在は、`Apply`の並行性はデフォルトで無効になっています。

### tidb_enable_pipelined_window_function

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)なし
- タイプ：真偽
- デフォルト値：`ON`
- この変数は、ウィンドウ関数に対してパイプライン実行アルゴリズムを使用するかどうかを指定します。

### tidb_enable_plan_cache_for_param_limit <span class="version-mark">v6.6.0で新規追加</span>

- スコープ：セッション | グローバル
- クラスタに永続化：はい
- ヒントに適用：[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)なし
- タイプ：真偽
- デフォルト値：`ON`
- この変数は、`LIMIT ?` のような変数を持つ実行計画を Prepared Plan Cache がキャッシュするかどうかを制御します。デフォルト値は `ON` で、これは Prepared Plan Cache がこのような実行計画をキャッシュすることを意味します。Prepared Plan Cache は 10000 を超える変数を持つ実行計画のキャッシュをサポートしないことに注意してください。

### tidb_enable_plan_cache_for_subquery <span class="version-mark">v7.0.0 で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`ON`
- この変数は、サブクエリを含むクエリを Prepared Plan Cache がキャッシュするかどうかを制御します。

### tidb_enable_plan_replayer_capture

<CustomContent platform="tidb-cloud">

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、`PLAN REPLAYER CAPTURE` 機能を有効にするかどうかを制御します。デフォルト値 `OFF` は `PLAN REPLAYER CAPTURE` 機能を無効にすることを意味します。

</CustomContent>

<CustomContent platform="tidb">

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`ON`
- この変数は、[`PLAN REPLAYER CAPTURE` 機能](/sql-plan-replayer.md#use-plan-replayer-capture-to-capture-target-plans) を有効にするかどうかを制御します。デフォルト値 `ON` は `PLAN REPLAYER CAPTURE` 機能を有効にすることを意味します。

</CustomContent>

### tidb_enable_plan_replayer_continuous_capture <span class="version-mark">v7.0.0 で新規追加</span>

<CustomContent platform="tidb-cloud">

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、`PLAN REPLAYER CONTINUOUS CAPTURE` 機能を有効にするかどうかを制御します。デフォルト値 `OFF` はこの機能を無効にするこ とを意味します。

</CustomContent>

<CustomContent platform="tidb">

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、[`PLAN REPLAYER CONTINUOUS CAPTURE` 機能](/sql-plan-replayer.md#use-plan-replayer-continuous-capture) を有効にするかどうかを制御します。デフォルト値 `OFF` はこの機能を無効にすることを意味します。

</CustomContent>

### tidb_enable_prepared_plan_cache <span class="version-mark">v6.1.0 で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- タイプ：ブール値
- デフォルト値：`ON`
- [Prepared Plan Cache](/sql-prepared-plan-cache.md) を有効にするかどうかを決定します。これが有効の場合、`Prepare` と `Execute` の実行計画がキャッシュされ、後続の実行は実行計画の最適化をスキップするため、パフォーマンスが向上します。
- この設定は以前は `tidb.toml` オプション (`prepared-plan-cache.enabled`) でしたが、TiDB v6.1.0 からシステム変数に変更されました。

### tidb_enable_prepared_plan_cache_memory_monitor <span class="version-mark">v6.4.0 で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：`ON`
- この変数は、Prepared Plan Cache にキャッシュされた実行計画によって消費されるメモリをカウントするかどうかを制御します。詳細については、[Prepared Plan Cache のメモリ管理](/sql-prepared-plan-cache.md#memory-management-of-prepared-plan-cache)を参照してください。

### tidb_enable_pseudo_for_outdated_stats <span class="version-mark">v5.3.0 で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、統計情報が古い場合にテーブルの統計情報をオプティマイザが使用する動作を制御します。

<CustomContent platform="tidb">

- オプティマイザは、テーブルの統計情報が古いかどうかを次のように判断します：テーブルの統計情報を取得するために `ANALYZE` が直近に実行されてから、テーブルの行数の 80% が変更された場合（変更された行数を総行数で割ったもの）。この比率は [`pseudo-estimate-ratio`](/tidb-configuration-file.md#pseudo-estimate-ratio) 構成を使用して変更できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- オプティマイザは、テーブルの統計情報が古いかどうかを次のように判断します：テーブルの統計情報を取得するために `ANALYZE` が直近に実行されてから、テーブルの行数の 80% が変更された場合（変更された行数を総行数で割ったもの）。

</CustomContent>

- デフォルト値である「`OFF`」の場合、テーブルの統計情報が古くても、オプティマイザは引き続きテーブルの統計情報を使用します。変数の値を `ON` に設定すると、オプティマイザはテーブルの統計情報が総行数以外の面で信頼できなくなったと判断します。そのため、オプティマイザは擬似統計情報を使用します。
- 表のデータが `ANALYZE` を実行せずに頻繁に変更される場合、実行計画を安定させるため、変数の値を `OFF` に設定することをお勧めします。

### tidb_enable_rate_limit_action

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、データを読み取る演算子のための動的メモリ制御機能を有効にするかどうかを制御します。デフォルトでは、この演算子は、[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency) が許可するスレッドの最大数を使用してデータを読み取ります。単一の SQL ステートメントのメモリ使用量が毎回 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を超過すると、データを読み取る演算子はスレッドを 1 つ停止します。

<CustomContent platform="tidb">

- データを読み取る演算子が 1 つのスレッドだけになり、単一の SQL ステートメントのメモリ使用量が常に[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を超過する場合、この SQL ステートメントはディスクにデータを書き出すなど、他のメモリ制御動作をトリガーします。
- この変数は、SQL ステートメントがデータを読み取る場合、メモリの使用を効果的に制御します。計算操作（結合や集約操作など）が必要な場合、メモリ使用量は `tidb_mem_quota_query` の制御外になることがあり、OOM のリスクを高めます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- データを読み取る演算子が 1 つのスレッドだけになり、単一の SQL ステートメントのメモリ使用量が常に[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を超過する場合、この SQL ステートメントはディスクにデータを書き出すなど、他のメモリ制御動作をトリガーします。

</CustomContent>

### tidb_enable_resource_control <span class="version-mark">v6.6.0 で新規追加</span>

> **注意:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ：GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：`ON`
- クラスター内で[リソース制御機能](/tidb-resource-control.md)のスイッチです。この変数が `ON` に設定されている場合、TiDB クラスターはリソースグループに基づいてアプリケーションのリソースを分離できます。

### tidb_enable_reuse_chunk <span class="version-mark">v6.4.0 で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスターへの永続化：あり
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：なし
- デフォルト値：`ON`
- 値のオプション：`OFF`, `ON`
- この変数は、TiDB がチャンクオブジェクトのキャッシュを有効にするかどうかを制御します。値が `ON` の場合、TiDB はキャッシュされたチャンクオブジェクトを使用し、要求されたオブジェクトがキャッシュにない場合にのみシステムから要求します。値が `OFF` の場合、TiDB は直接システムからチャンクオブジェクトを要求します。

### tidb_enable_slow_log

> **注意:**
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続中の現在のTiDBインスタンスにのみ適用されます。
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `ON`
- この変数はスロークエリログ機能を有効にするかどうかを制御するために使用されます。

### tidb_enable_tmp_storage_on_oom

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- デフォルト値: `ON`
- 値のオプション: `OFF`, `ON`
- システム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)で指定されたメモリクォータを超えた単一のSQLステートメントの一部のオペレータのための一時ストレージを有効にするかどうかを制御します。
- v6.3.0以前では、TiDB設定項目`oom-use-tmp-storage`を使用してこの機能を有効または無効にできます。v6.3.0以降のクラスターにアップグレードした後、TiDBクラスターは`oom-use-tmp-storage`の値を自動的にこの変数で初期化します。その後、`oom-use-tmp-storage`の値を変更しても**影響しません**。

### tidb_enable_stmt_summary <span class="version-mark">v3.0.4で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `ON`
- この変数はステートメントサマリー機能を有効にするかどうかを制御するために使用されます。有効にすると、SQLの実行情報（時間消費など）が`information_schema.STATEMENTS_SUMMARY`システムテーブルに記録され、SQLのパフォーマンスの問題を特定およびトラブルシュートすることができます。

### tidb_enable_strict_double_type_check <span class="version-mark">v5.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `ON`
- この変数はテーブルが型`DOUBLE`の無効な定義で作成できるかどうかを制御するために使用されます。この設定は、以前のTiDBのバージョンよりも型を厳密に検証していなかった際のアップグレードパスを提供することを意図しています。
- `ON`のデフォルト値はMySQLと互換性があります。

たとえば、型`DOUBLE(10)`は、浮動小数点数の精度が保証されていないため、無効と見なされます。`tidb_enable_strict_double_type_check`を`OFF`に変更した後、次のようにテーブルを作成できます。

```sql
mysql> CREATE TABLE t1 (id int, c double(10));
ERROR 1149 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use

mysql> SET tidb_enable_strict_double_type_check = 'OFF';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t1 (id int, c double(10));
Query OK, 0 rows affected (0.09 sec)
```

> **注意:**
>
> この設定は`FLOAT`タイプの精度を指定することを許可するMySQLの振る舞いについてのみ適用されます。この動作はMySQL 8.0.17をもって廃止予定とされており、`FLOAT`または`DOUBLE`タイプの精度を指定することは推奨されていません。

### tidb_enable_table_partition

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 列挙型
- デフォルト値: `ON`
- 可能な値: `OFF`, `ON`, `AUTO`
- この変数は`TABLE PARTITION`機能を有効にするかどうかを設定するために使用されます：
    - `ON`は範囲パーティショニング、ハッシュパーティショニング、および範囲列パーティショニング（単一の列）を有効にします。
    - `AUTO`は`ON`と同じように機能します。
    - `OFF`は`TABLE PARTITION`機能を無効にします。この場合、パーティションテーブルを作成する構文は実行できますが、作成されるテーブルはパーティション化されたものではありません。

### tidb_enable_telemetry <span class="version-mark">v4.0.2で新規</span>

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `OFF`

<CustomContent platform="tidb">

- この変数は、TiDBのテレメトリ収集が有効かどうかを動的に制御するために使用されます。現在のバージョンでは、デフォルトでテレメトリは無効になっています。すべてのTiDBインスタンスで[`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-new-in-v402) TiDB設定項目が`false`に設定されている場合、テレメトリ収集は常に無効になり、このシステム変数は効果を発揮しません。詳細については[Telemetry](/telemetry.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、TiDBのテレメトリ収集が有効かどうかを動的に制御するために使用されます。

</CustomContent>

### tidb_enable_tiflash_read_for_write_stmt <span class="version-mark">v6.3.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `ON`
- この変数は、`INSERT`、`DELETE`、`UPDATE`を含むSQLステートメントの読み込み操作をTiFlashにプッシュダウンできるかどうかを制御します。たとえば：

    - `SELECT`クエリを含む`INSERT INTO SELECT`ステートメント（典型的な使用シナリオ：[TiFlashクエリ結果のマテリアリゼーション](/tiflash/tiflash-results-materialization.md)）
    - `UPDATE`および`DELETE`ステートメント内の`WHERE`条件フィルタリング
- v7.1.0からは、この変数は非推奨です。[`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50)時、オプティマイザは現在のセッションの[SQLモード](/sql-mode.md)とTiFlashレプリカのコスト推定に基づいて、クエリをTiFlashにプッシュダウンするかどうかをインテリジェントに決定します。なお、TiDBは現在のセッションの[SQLモード](/sql-mode.md)が厳密でない限り（つまり、`sql_mode`の値に`STRICT_TRANS_TABLES`および`STRICT_ALL_TABLES`が含まれていない場合）にのみ、`INSERT INTO SELECT`などの`INSERT`、`DELETE`、`UPDATE`を含むSQLステートメントの読み込み操作をTiFlashにプッシュダウンすることが可能です。

### tidb_enable_top_sql <span class="version-mark">v5.4.0で新規</span>

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `OFF`

<CustomContent platform="tidb">

- この変数は[Top SQL](/dashboard/top-sql.md)機能を有効にするかどうかを制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は[Top SQL](https://docs.pingcap.com/tidb/stable/top-sql)機能を有効にするかどうかを制御するために使用されます。

</CustomContent>

### tidb_enable_tso_follower_proxy <span class="version-mark">v5.3.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブーリアン
- デフォルト値: `OFF`
- この変数はTSO Follower Proxy機能を有効にします。値が`OFF`の場合、TiDBはPDリーダーからのTSOのみを取得します。この機能を有効にすると、TiDBはすべてのPDノードに均等にリクエストを送信し、TSOリクエストをPDフォロワーを介して転送します。これにより、PDリーダーのCPU負荷が軽減されます。
- TSO Follower Proxyを有効にするためのシナリオ：
    * TSOリクエストの高い圧力により、PDリーダーのCPUがボトルネックに達し、TSO RPCリクエストの遅延が発生する。
    * TiDBクラスターには多くのTiDBインスタンスがあり、[`tidb_tso_client_batch_max_wait_time`](#tidb_tso_client_batch_max_wait_time-new-in-v530)の値を増やしてもTSO RPCリクエストの遅延問題が緩和されない場合。

> **注意:**
>
> TSO RPCの遅延がCPUの使用率のボトルネック以外の理由で増加した場合（ネットワークの問題など）、TiDBの実行遅延が増加し、クラスタのQPSパフォーマンスに影響を与える可能性があります。

### tidb_enable_unsafe_substitute <span class="version-mark">v6.3.0で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、生成された列の式を安全でない方法で置き換えるかどうかを制御します。デフォルト値は`OFF`で、つまり安全な置換はデフォルトで無効化されています。詳細については、[Generated Columns](/generated-columns.md)を参照してください。

### tidb_enable_vectorized_expression <span class="version-mark">v4.0で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- タイプ：ブール値
- デフォルト値：`ON`
- この変数は、ベクトル化された実行を有効にするかどうかを制御するために使用されます。

### tidb_enable_window_function

- スコープ：SESSION | GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`ON`
- この変数は、ウィンドウ関数のサポートを有効にするかどうかを制御します。ウィンドウ関数は予約語を使用する場合があります。これにより、TiDBをアップグレードした後、通常実行できたSQLステートメントが構文解析できなくなる可能性があります。この場合は、`tidb_enable_window_function`を`OFF`に設定できます。

### `tidb_enable_row_level_checksum` <span class="version-mark">v7.1.0で新規追加</span>

- スコープ：GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`OFF`

<CustomContent platform="tidb">

- この変数は、単一行データの[TiCDCデータ整合性検証](/ticdc/ticdc-integrity-check.md)機能を有効にするかどうかを制御します。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、単一行データの[TiCDCデータ整合性検証](https://docs.pingcap.com/tidb/stable/ticdc-integrity-check)機能を有効にするかどうかを制御します。

</CustomContent>

### tidb_enforce_mpp <span class="version-mark">v5.1で新規追加</span>

- スコープ：SESSION
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- タイプ：ブール値
- デフォルト値：`OFF`

<CustomContent platform="tidb">

- この変数を変更するには、[`performance.enforce-mpp`](/tidb-configuration-file.md#enforce-mpp)設定値を変更してください。

</CustomContent>

- オプティマイザのコスト推定を無視し、強制的にTiFlashのMPPモードをクエリ実行に使用するかどうかを制御します。値のオプションは以下のとおりです：
    - `0`または`OFF`：MPPモードは強制的に使用されません（デフォルト）。
    - `1`または`ON`：コスト推定が無視され、MPPモードが強制的に使用されます。この設定は、`tidb_allow_mpp=true`のときにのみ有効です。

MPPはTiFlashエンジンが提供する分散コンピューティングフレームワークで、ノード間のデータ交換を可能にし、高性能で高スループットなSQLアルゴリズムを提供します。MPPモードの選択の詳細については、[MPPモードを選択するかどうかを制御する](/tiflash/use-tiflash-mpp-mode.md#control-whether-to-select-the-mpp-mode)を参照してください。

### tidb_evolve_plan_baselines <span class="version-mark">v4.0で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、ベースライン進化機能を有効にするかどうかを制御します。詳細な紹介または使用法については、[ベースライン進化](/sql-plan-management.md#baseline-evolution)を参照してください。
- クラスタへの影響を軽減するには、次の設定を使用してください：
    - `tidb_evolve_plan_task_max_time`を設定して、各実行計画の最大実行時間を制限します。デフォルト値は600秒です。
    - `tidb_evolve_plan_task_start_time`および`tidb_evolve_plan_task_end_time`を設定して、時間ウィンドウを制限します。デフォルト値はそれぞれ`00:00 +0000`および`23:59 +0000`です。

### tidb_evolve_plan_task_end_time <span class="version-mark">v4.0で新規追加</span>

- スコープ：GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：Time
- デフォルト値：`23:59 +0000`
- この変数は、1日のベースライン進化の終了時刻を設定するために使用されます。

### tidb_evolve_plan_task_max_time <span class="version-mark">v4.0で新規追加</span>

- スコープ：GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`600`
- 範囲：`[-1, 9223372036854775807]`
- 単位：秒
- この変数は、ベースライン進化機能における各実行計画の最大実行時間を制限するために使用されます。

### tidb_evolve_plan_task_start_time <span class="version-mark">v4.0で新規追加</span>

- スコープ：GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：Time
- デフォルト値：`00:00 +0000`
- この変数は、1日のベースライン進化の開始時刻を設定するために使用されます。

### tidb_executor_concurrency <span class="version-mark">v5.0で新規追加</span>

- スコープ：SESSION | GLOBAL
- クラスタに永続化する：はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- タイプ：整数
- デフォルト値：`5`
- 範囲：`[1, 256]`
- 単位：スレッド

この変数は、以下のSQLオペレータの並行性を1つの値に設定するために使用されます：

- `index lookup`
- `index lookup join`
- `hash join`
- `hash aggregation`（`partial`および`final`フェーズ）
- `window`
- `projection`

`tidb_executor_concurrency`は、より簡単な管理のために、以下の既存のシステム変数をまとめて取り込んでいます：

+ `tidb_index_lookup_concurrency`
+ `tidb_index_lookup_join_concurrency`
+ `tidb_hash_join_concurrency`
+ `tidb_hashagg_partial_concurrency`
+ `tidb_hashagg_final_concurrency`
+ `tidb_projection_concurrency`
+ `tidb_window_concurrency`

v5.0以降、引き続き上記のシステム変数を別々に変更できます（廃止警告が返されます）、そして、あなたの変更は対応する単一のオペレータにのみ影響を与えます。その後、`tidb_executor_concurrency`を使用してオペレータの並行性を修正すると、別々に修正されたオペレータに影響を与えなくなります。`tidb_executor_concurrency`を使用してすべてのオペレータの並行性を変更したい場合は、上記のすべての変数の値を`-1`に設定できます。

以前のバージョンからv5.0にアップグレードされたシステムの場合、これらの変数の値を1つも変更していない（`tidb_hash_join_concurrency`の値が`5`で、残りの値は`4`であることを意味します）場合、以前にこれらの変数で管理されていたオペレータの並行性は自動的に`tidb_executor_concurrency`によって管理されます。これらの変数のいずれかを変更した場合、対応するオペレータの並行性は引き続き変更された変数によって制御されます。

### tidb_expensive_query_time_threshold

> **注意：**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ：GLOBAL
- クラスタに永続化する：いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`60`
- 範囲：`[10, 2147483647]`
- 単位：秒
- この変数は、高価なクエリログを出力するかどうかを決定する閾値値を設定するために使用されます。高価なクエリログと遅いクエリログの違いは次のとおりです：
    - 遅いログはステートメントの実行後に出力されます。
    - 高価なクエリログは、閾値値を超える実行時間を持つ実行中のステートメントとそれに関連する情報を出力します。

### tidb_expensive_txn_time_threshold <span class="version-mark">v7.2.0で新規追加</span>

<CustomContent platform="tidb-cloud">

> **注意：**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターへの永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `600`
- 範囲: `[60, 2147483647]`
- 単位: 秒
- この変数は、デフォルトで600秒のコストのかかるトランザクションのログ記録の閾値を制御します。トランザクションの実行時間が閾値を超え、かつトランザクションがコミットされずロールバックされない場合、コストのかかるトランザクションと見なされ、ログに記録されます。

### tidb_force_priority

> **注意:**

> このTiDBの変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターへの永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 列挙型
- デフォルト値: `NO_PRIORITY`
- 可能な値: `NO_PRIORITY`, `LOW_PRIORITY`, `HIGH_PRIORITY`, `DELAYED`
- この変数は、TiDBサーバー上で実行されるステートメントのデフォルトの優先度を変更するために使用されます。使用例は、OLTPクエリを実行するユーザーに対してOLAPクエリを実行している特定のユーザーに対して低い優先度を確保することです。デフォルト値である `NO_PRIORITY` は、ステートメントの優先度を強制的に変更しないことを意味します。

### tidb_gc_concurrency <span class="version-mark">v5.0で新規</span>

> **注意:**

> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- GCの[ロックの解決](/garbage-collection-overview.md#resolve-locks)ステップにおけるスレッドの数を指定します。値が `-1` の場合、TiDBはガベージコレクションに使用するスレッドの数を自動的に決定します。

### tidb_gc_enable <span class="version-mark">v5.0で新規</span>

> **注意:**

> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- TiKVのガベージコレクションを有効にします。ガベージコレクションを無効にすると、古いバージョンの行は削除されなくなり、システムのパフォーマンスが低下します。

### tidb_gc_life_time <span class="version-mark">v5.0で新規</span>

> **注意:**

> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 期間
- デフォルト値: `10m0s`
- 範囲: `[10m0s, 8760h0m0s]`
- 各GCの実行ごとにデータが保持される時間制限をGo Duration形式で指定します。GCが発生した時点からこの値を引いた時刻がセーフポイントです。

> **注意:**

> - 頻繁な更新のシナリオでは、`tidb_gc_life_time`に大きな値（数日または数ヶ月）を設定すると、次のような潜在的な問題が発生する可能性があります:
>     - より大きなストレージ使用
>     - 大量の履歴データがパフォーマンスに一定の影響を及ぼす可能性があり、特に `select count(*) from t` のような範囲クエリに対して
> - `tidb_gc_life_time`よりも長い実行時間のトランザクションがある場合、GC実行時に`start_ts`以降のデータがこのトランザクションの実行を継続するために保持されます。たとえば、`tidb_gc_life_time` を10分に設定した場合、実行中のすべてのトランザクションの中で最も早く開始したトランザクションが15分間実行されている場合、GCは最近の15分間のデータを保持します。

### tidb_gc_max_wait_time <span class="version-mark">v6.1.0で新規</span>

> **注意:**

> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `86400`
- 範囲: `[600, 31536000]`
- 単位: 秒
- この変数は、アクティブなトランザクションがGCセーフポイントをブロックする最大時間を設定するために使用されます。各GC時、デフォルトではセーフポイントは実行中のトランザクションの開始時刻を上回りません。アクティブなトランザクションの実行時間がこの変数の値を超えない場合、GCセーフポイントは、実行時間がこの値を超えるまでブロックされます。

### tidb_gc_run_interval <span class="version-mark">v5.0で新規</span>

> **注意:**

> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 期間
- デフォルト値: `10m0s`
- 範囲: `[10m0s, 8760h0m0s]`
- GCの間隔をGo Duration形式で指定します。たとえば、`"1h30m"`、`"15m"` のような形式です。

### tidb_gc_scan_lock_mode <span class="version-mark">v5.0で新規</span>

> **警告:**

> 現在、Green GCは実験的な機能です。本番環境で使用することはお勧めしません。

> **注意:**

> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 列挙型
- デフォルト値: `LEGACY`
- 可能な値: `PHYSICAL`, `LEGACY`
    - `LEGACY`: 古いスキャン方法を使用し、つまりGreen GCを無効にします。
    - `PHYSICAL`: 物理的なスキャン方法を使用し、つまりGreen GCを有効にします。

<CustomContent platform="tidb">

- この変数は、GCの[ロックの解決](/garbage-collection-overview.md#resolve-locks)ステップにおけるロックのスキャン方法を指定します。`LEGACY` の値が設定されている場合、TiDBはリージョン単位でのロックのスキャンを行います。`PHYSICAL` を使用すると、各TiKVノードがRaftレイヤーをバイパスして直接データをスキャンできるようになります。これにより、[ヒバネートリージョン](/tikv-configuration-file.md#hibernate-regions)機能が有効になっているときにGCがすべてのリージョンを起こす影響を効果的に緩和し、ロックの解決ステップの実行速度が向上します。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、GCの[ロックの解決](/garbage-collection-overview.md#resolve-locks)ステップにおけるロックのスキャン方法を指定します。`LEGACY` の値が設定されている場合、TiDBはリージョン単位でのロックのスキャンを行います。`PHYSICAL` を使用すると、各TiKVノードがRaftレイヤーをバイパスして直接データをスキャンできるようになります。これにより、GCがすべてのリージョンを起こす影響を効果的に緩和し、ロックの解決ステップの実行速度が向上します。

</CustomContent>

### tidb_general_log

> **注意:**

> このTiDBの変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターへの永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`

<CustomContent platform="tidb-cloud">

- この変数は、すべてのSQLステートメントをログに記録するかどうかを設定するために使用されます。この機能はデフォルトで無効になっています。問題を特定する際にすべてのSQLステートメントを追跡する必要がある場合は、この機能を有効にします。

</CustomContent>

<CustomContent platform="tidb">

- この変数は、[ログ](/tidb-configuration-file.md#logfile)にすべてのSQLステートメントを記録するかどうかを設定するために使用されます。この機能はデフォルトで無効になっています。問題を特定する際にすべてのSQLステートメントを追跡する必要があるメンテナンス担当者は、この機能を有効にすることができます。

- この機能のレコードをログで表示するには、TiDB構成項目 [`log.level`](/tidb-configuration-file.md#level) を `"info"` または `"debug"` に設定し、次に `"GENERAL_LOG"` 文字列をクエリする必要があります。以下の情報が記録されます:
    - `conn`: 現在のセッションのID。
    - `user`: 現在のセッションユーザー。
    - `schemaVersion`: 現在のスキーマバージョン。
    - `txnStartTS`: 現在のトランザクションが開始されたタイムスタンプ。
    - `forUpdateTS`: 悲観的トランザクションモードでは、`forUpdateTS` は SQL ステートメントの現在のタイムスタンプです。TiDB は悲観的トランザクションで書き込み競合が発生した場合、実行中の SQL ステートメントを再試行し、このタイムスタンプを更新します。[`max-retry-count`](/tidb-configuration-file.md#max-retry-count) を介して再試行回数を構成できます。楽観的トランザクションモデルでは、`forUpdateTS` は `txnStartTS` と同等です。
    - `isReadConsistency`: 現在のトランザクション分離レベルが Read Committed (RC) であることを示します。
    - `current_db`: 現在のデータベース名。
    - `txn_mode`: トランザクションモード。値のオプションは `OPTIMISTIC` および `PESSIMISTIC` です。
    - `sql`: 現在のクエリに対応する SQL ステートメント。

</CustomContent>

### tidb_non_prepared_plan_cache_size

> **注意:**
>
> v7.1.0 からこの変数は非推奨となりました。代わりに[`tidb_session_plan_cache_size`](#tidb_session_plan_cache_size-new-in-v710) を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[1, 100000]`
- この変数は、[非準備済みプランキャッシュ](/sql-non-prepared-plan-cache.md) によってキャッシュできる最大の実行計画数を制御します。

### tidb_generate_binary_plan <span class="version-mark">v6.2.0 で新規</span>

> **ノート:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は遅いログとステートメントサマリーでバイナリ符号化された実行計画を生成するかどうかを制御します。
- この変数が `ON` に設定されている場合、TiDB ダッシュボードで視覚的な実行計画を表示できます。なお、TiDB ダッシュボードはこの変数が有効になった後に生成された実行計画の視覚表示のみを提供します。
- `SELECT tidb_decode_binary_plan('xxx...')` ステートメントを実行して、バイナリ計画から特定の計画を解析できます。

### tidb_gogc_tuner_max_value <span class="version-mark">v7.5.0 で新規</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 整数
- デフォルト値: `500`
- 範囲: `[10, 2147483647]`
- この変数は GOGC Tuner が調整できる GOGC の最大値を制御するために使用されます。

### tidb_gogc_tuner_min_value <span class="version-mark">v7.5.0 で新規</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[10, 2147483647]`
- この変数は GOGC Tuner が調整できる GOGC の最小値を制御するために使用されます。

### tidb_gogc_tuner_threshold <span class="version-mark">v6.4.0 で新規</span>

> **ノート:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- デフォルト値: `0.6`
- 範囲: `[0, 0.9)`
- この変数は、GOGC を調整するための最大メモリ閾値を指定します。メモリがこの閾値を超えると、GOGC Tuner は動作を停止します。

### tidb_guarantee_linearizability <span class="version-mark">v5.0 で新規</span>

> **ノート:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は非同期コミットのためにコミット TS を計算する方法を制御します。デフォルト値 (`ON`) の場合、2 相コミットは PD サーバーから新しい TS を要求し、TS を使用して最終的なコミット TS を計算します。この状況では、すべての並行トランザクションに対して線形性が保証されます。
- この変数を `OFF` に設定すると、PD サーバーから TS を取得するプロセスがスキップされ、因果整合性のみが保証されますが、線形性は保証されません (コストがかかります)。詳細については、ブログ記事 [Async Commit, the Accelerator for Transaction Commit in TiDB 5.0](https://en.pingcap.com/blog/async-commit-the-accelerator-for-transaction-commit-in-tidb-5-0/) を参照してください。
- 因果整合性のみが必要なシナリオでは、パフォーマンス向上のためにこの変数を `OFF` に設定できます。

### tidb_hash_exchange_with_new_collation

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、新しい照合順序が有効なクラスターで MPP ハッシュパーティション交換演算子が生成されるかどうかを制御します。`true` は演算子を生成し、`false` は生成しないことを意味します。
- この変数は TiDB の内部操作に使用されます。この変数を設定することは**推奨されません**。

### tidb_hash_join_concurrency

> **注意:**
>
> v5.0 以降、この変数は非推奨となりました。代わりに [`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50) を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は `hash join` アルゴリズムの並行性を設定するために使用されます。
- `-1` の値は `tidb_executor_concurrency` の値が代わりに使用されることを意味します。

### tidb_hashagg_final_concurrency

> **注意:**
>
> v5.0 以降、この変数は非推奨となりました。代わりに [`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50) を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`final` フェーズで並行して `hash aggregation` アルゴリズムを実行する並行性を設定するために使用されます。
- 集約関数のパラメータが重複していない場合、`HashAgg` は 2 つのフェーズでそれぞれ - `partial` フェーズと `final` フェーズ - に並行して実行されます。
- `-1` の値は `tidb_executor_concurrency` の値が代わりに使用されることを意味します。

### tidb_hashagg_partial_concurrency

> **注意:**
>
> v5.0 以降、この変数は非推奨となりました。代わりに [`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50) を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`partial` フェーズで並行して `hash aggregation` アルゴリズムを実行する並行性を設定するために使用されます。
- 集約関数のパラメータが重複していない場合、`HashAgg` は 2 つのフェーズでそれぞれ - `partial` フェーズと `final` フェーズ - に並行して実行されます。
- `-1` の値は `tidb_executor_concurrency` の値が代わりに使用されることを意味します。

### tidb_historical_stats_duration <span class="version-mark">v6.6.0 で新規</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: 期間
- デフォルト値: `168h` (7 日)
- この変数は、歴史的な統計情報がストレージに保持される期間を制御します。
### tidb_ignore_prepared_cache_close_stmt <span class="version-mark">v6.0.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブーリアン
- デフォルト値: `OFF`
- この変数は、プリペアドステートメントのキャッシュをクローズするコマンドを無視するかどうかを設定するために使用されます。
- この変数を `ON` に設定すると、バイナリプロトコルの `COM_STMT_CLOSE` コマンドとテキストプロトコルの [`DEALLOCATE PREPARE`](/sql-statements/sql-statement-deallocate.md) ステートメントが無視されます。詳細については、[「`COM_STMT_CLOSE` コマンドおよび `DEALLOCATE PREPARE` ステートメントの無視」](/sql-prepared-plan-cache.md#ignore-the-com_stmt_close-command-and-the-deallocate-prepare-statement)を参照してください。

### tidb_index_join_batch_size

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `25000`
- 範囲: `[1, 2147483647]`
- 単位: 行
- この変数は、`index lookup join` 操作のバッチサイズを設定するために使用されます。
- OLAP シナリオでは大きい値、OLTP シナリオでは小さい値を使用してください。

### tidb_index_join_double_read_penalty_cost_rate <span class="version-mark">v6.6.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 浮動小数点数
- デフォルト値: `0`
- 範囲: `[0, 18446744073709551615]`
- この変数は、インデックス結合の選択にペナルティコストを適用するかどうかを決定し、オプティマイザがインデックス結合を選択する可能性を減少させ、ハッシュ結合や Tiflash 結合などの代替結合方法を選択する可能性を増加させます。
- インデックス結合が選択されると、多くのテーブルルックアップリクエストがトリガーされ、多くのリソースを消費します。この変数を使用して、オプティマイザがインデックス結合を選択する可能性を減少させることができます。
- この変数は、 [`tidb_cost_model_version`](/system-variables.md#tidb_cost_model_version-new-in-v620) 変数が `2` に設定されている場合のみ有効です。

### tidb_index_lookup_concurrency

> **警告:**
>
> v5.0以降、この変数は非推奨です。代わりに [`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50) を使用して設定してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`index lookup` 操作の並列性を設定するために使用されます。
- OLAP シナリオでは大きい値、OLTP シナリオでは小さい値を使用してください。
- `-1` の値は、`tidb_executor_concurrency` の値が使用されることを意味します。

### tidb_index_lookup_join_concurrency

> **警告:**
>
> v5.0以降、この変数は非推奨です。代わりに [`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50) を使用して設定してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`index lookup join` アルゴリズムの並列性を設定するために使用されます。
- `-1` の値は、`tidb_executor_concurrency` の値が使用されることを意味します。

### tidb_index_merge_intersection_concurrency <span class="version-mark">v6.5.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- この変数は、インデックスマージが行う交差操作の最大並列性を設定します。これは、TiDBが動的プルーニングモードでパーティションテーブルにアクセスする場合にのみ有効です。実際の並列性は、`tidb_index_merge_intersection_concurrency` とパーティション化されたテーブルの数の小さい方の値です。
- デフォルト値 `-1` は、`tidb_executor_concurrency` の値が使用されることを意味します。

### tidb_index_lookup_size

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `20000`
- 範囲: `[1, 2147483647]`
- 単位: 行
- この変数は、`index lookup` 操作のバッチサイズを設定するために使用されます。
- OLAP シナリオでは大きい値、OLTP シナリオでは小さい値を使用してください。

### tidb_index_serial_scan_concurrency

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`serial scan` 操作の並列性を設定するために使用されます。
- OLAP シナリオでは大きい値、OLTP シナリオでは小さい値を使用してください。

### tidb_init_chunk_size

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `32`
- 範囲: `[1, 32]`
- 単位: 行
- この変数は、実行プロセス中の初期チャンクの行数を設定するために使用されます。

### tidb_isolation_read_engines <span class="version-mark">v4.0で新規追加</span>

> **注意:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `tikv,tiflash,tidb`
- この変数は、データを読み取る際にTiDBが使用できるストレージエンジンのリストを設定するために使用されます。

### tidb_last_ddl_info <span class="version-mark">v6.0.0で新規追加</span>

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- タイプ: 文字列
- この変数は読み取り専用です。これは、現在のセッション内での最後のDDL操作の情報を取得するためにTiDB内部で使用されます。
    - "query": 最後のDDLクエリ文字列。
    - "seq_num": 各DDL操作のシーケンス番号。これはDDL操作の順序を識別するために使用されます。

### tidb_last_query_info <span class="version-mark">v4.0.14で新規追加</span>

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- この変数は読み取り専用です。これは、TiDB内部で最後のDMLステートメントのトランザクション情報をクエリするために使用されます。情報には次のものが含まれます：
    - `txn_scope`: トランザクションのスコープ。`global` または `local` となります。
    - `start_ts`: トランザクションの開始タイムスタンプ。
    - `for_update_ts`: 前に実行したDMLステートメントの `for_update_ts`。これはTiDBの内部用語であり、通常はこの情報を無視して問題ありません。
    - `error`: エラーメッセージがある場合は表示されます。

### tidb_last_txn_info <span class="version-mark">v4.0.9で新規追加</span>

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 文字列
- この変数は、現在のセッション内での最後のトランザクション情報を取得するために使用されます。これは読み取り専用変数です。トランザクション情報には次のものが含まれます：
    - トランザクションのスコープ。
    - 開始およびコミットTS。
    - トランザクションのコミットモード。2 相、1 相、または非同期コミットからのトランザクションのコミット情報。
    - 遭遇したエラー。

### tidb_last_plan_replayer_token <span class="version-mark">v6.3.0で新規追加</span>

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 文字列
- この変数は読み取り専用であり、現在のセッションで実行された最後の `PLAN REPLAYER DUMP` の結果を取得するために使用されます。
### tidb_load_based_replica_read_threshold <span class="version-mark">v7.0.0 で新規追加</span>

<CustomContent platform="tidb">

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `"1s"`
- 範囲: `[0s, 1h]`
- タイプ: 文字列
- この変数は、ロードベースのレプリカ読み取りをトリガーするための閾値を設定するために使用されます。リーダーノードの推定キュー時間が閾値を超えると、TiDB はデータをフォロワーノードから読み取るように優先します。フォーマットは時間の長さで、例えば `"100ms"` または `"1s"` です。詳細については、[ホットスポットの問題のトラブルシューティング](/troubleshoot-hot-spot-issues.md#scatter-read-hotspots)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `"1s"`
- 範囲: `[0s, 1h]`
- タイプ: 文字列
- この変数は、ロードベースのレプリカ読み取りをトリガーするための閾値を設定するために使用されます。リーダーノードの推定キュー時間が閾値を超えると、TiDB はデータをフォロワーノードから読み取るように優先します。フォーマットは時間の長さで、例えば `"100ms"` または `"1s"` です。詳細については、[ホットスポットの問題のトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#scatter-read-hotspots)を参照してください。

</CustomContent>

### `tidb_lock_unchanged_keys` <span class="version-mark">v7.1.1 および v7.3.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール
- デフォルト値: `ON`
- この変数は特定のシナリオでキーをロックするかどうかを制御するために使用されます。値が `ON` に設定されている場合、これらのキーはロックされます。値が `OFF` に設定されている場合、これらのキーはロックされません。
    - `INSERT IGNORE` および `REPLACE` ステートメントの重複キー。v6.1.6 より前では、これらのキーはロックされませんでした。この問題は [#42121](https://github.com/pingcap/tidb/issues/42121) で修正されています。
    - `UPDATE` ステートメント内の一意キー。キーの値が変更されていない場合。v6.5.2 より前では、これらのキーはロックされませんでした。この問題は [#36438](https://github.com/pingcap/tidb/issues/36438) で修正されています。
- トランザクションの一貫性と合理性を維持するために、この値を変更することはお勧めしません。これらの 2 つの修正による TiDB のアップグレードが深刻なパフォーマンスの問題を引き起こし、ロックのない動作が許容できる場合（前述の問題を参照）、この変数を `OFF` に設定することができます。

### tidb_log_file_max_days <span class="version-mark">v5.3.0 で新規追加</span>

> **注記:**
>
> この変数は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) に対して読み取り専用です。

- スコープ: GLOBAL
- クラスタへの永続化: いいえ、接続中の現在の TiDB インスタンスにのみ適用されます。
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`

<CustomContent platform="tidb">

- この変数は、現在の TiDB インスタンスに保持されるログの最大日数を設定するために使用されます。その値は通常、設定ファイルの [`max-days`](/tidb-configuration-file.md#max-days) 構成の値をデフォルトとしています。変数の値の変更は、現在の TiDB インスタンスにのみ影響します。TiDB を再起動した後、変数の値がリセットされ、設定値には影響しません。
</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、現在の TiDB インスタンスに保持されるログの最大日数を設定するために使用されます。

</CustomContent>

### tidb_low_resolution_tso

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、低解像度 TSO 機能を有効にするかどうかを設定するために使用されます。この機能を有効にした後、新しいトランザクションは、データを読み取るために 2 秒ごとに更新されるタイムスタンプを使用します。
- 主な適用シナリオは、古いデータの読み取りが可能な場合に、小さな読み取り専用トランザクションでの TSO 取得のオーバーヘッドを削減することです。

### tidb_max_auto_analyze_time <span class="version-mark">v6.1.0 で新規追加</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `43200`
- 範囲: `[0, 2147483647]`
- 単位: 秒
- この変数は、自動 `ANALYZE` タスクの最大実行時間を指定するために使用されます。自動 `ANALYZE` タスクの実行時間が指定時間を超過すると、タスクは中止されます。この変数の値が `0` の場合、自動 `ANALYZE` タスクの最大実行時間に制限はありません。

### tidb_max_bytes_before_tiflash_external_group_by <span class="version-mark">v7.0.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlash の `GROUP BY` とのハッシュ集約演算子の最大メモリ使用量をバイト単位で指定するために使用されます。メモリ使用量が指定値を超えると、TiFlash はハッシュ集約演算子をディスクへスパイルさせます。この変数の値が `-1` の場合、TiDB はこの変数を TiFlash に渡しません。この変数の値が `0` の場合、メモリ使用量は無制限となり、つまり TiFlash のハッシュ集約演算子はスパイルをトリガーしません。詳細については、[TiFlash Spill to Disk](/tiflash/tiflash-spill-disk.md)を参照してください。

<CustomContent platform="tidb">

> **注記:**
>
> - TiDB クラスタに複数の TiFlash ノードがある場合、集約は通常、複数の TiFlash ノードで分散して実行されます。この変数は、単一の TiFlash ノード上で集約演算子の最大メモリ使用量を制御します。
> - この変数が `-1` に設定されている場合、TiFlash は、自身の構成項目 [`max_bytes_before_external_group_by`](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters) の値に基づいて集約演算子の最大メモリ使用量を決定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注記:**
>
> - TiDB クラスタに複数の TiFlash ノードがある場合、集約は通常、複数の TiFlash ノードで分散して実行されます。この変数は、単一の TiFlash ノード上で集約演算子の最大メモリ使用量を制御します。
> - この変数が `-1` に設定されている場合、TiFlash は、自身の構成項目 `max_bytes_before_external_group_by` の値に基づいて集約演算子の最大メモリ使用量を決定します。

</CustomContent>

### tidb_max_bytes_before_tiflash_external_join <span class="version-mark">v7.0.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlash の `JOIN` とのハッシュ結合演算子の最大メモリ使用量をバイト単位で指定するために使用されます。メモリ使用量が指定値を超えると、TiFlash はハッシュ結合演算子をディスクへスパイルさせます。この変数の値が `-1` の場合、TiDB はこの変数を TiFlash に渡しません。この変数の値が `0` の場合、メモリ使用量は無制限となり、つまり TiFlash のハッシュ結合演算子はスパイルをトリガーしません。詳細については、[TiFlash Spill to Disk](/tiflash/tiflash-spill-disk.md)を参照してください。

<CustomContent platform="tidb">

> **注記:**
>
> - TiDB クラスタに複数の TiFlash ノードがある場合、結合は通常、複数の TiFlash ノードで分散して実行されます。この変数は、単一の TiFlash ノード上で結合演算子の最大メモリ使用量を制御します。
> - この変数を`-1`に設定すると、TiFlashは、その自身の構成項目[`max_bytes_before_external_join`](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters)の値に基づいて、結合演算子の最大メモリ使用量を決定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、joinは通常、複数のTiFlashノードで分散して実行されます。この変数は単一のTiFlashノード上の結合演算子の最大メモリ使用量を制御します。
> - この変数を`-1`に設定すると、TiFlashは、その自身の構成項目`max_bytes_before_external_join`の値に基づいて、結合演算子の最大メモリ使用量を決定します。

</CustomContent>

### tidb_max_bytes_before_tiflash_external_sort <span class="version-mark">v7.0.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlashのTopNおよびSort演算子の最大メモリ使用量（バイト単位）を指定するために使用されます。メモリ使用量が指定された値を超えると、TiFlashはTopNおよびSort演算子をディスクにスピルさせます。この変数の値が`-1`の場合、TiDBはこの変数をTiFlashに渡しません。この変数の値が`0`以上の場合にのみ、TiDBはこの変数をTiFlashに渡します。この変数の値が`0`の場合、メモリ使用量が制限されず、つまりTiFlashのTopNおよびSort演算子はスピルをトリガーしません。詳細については、[TiFlash Spill to Disk](/tiflash/tiflash-spill-disk.md)を参照してください。

<CustomContent platform="tidb">

> **注:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、TopNおよびSortは通常、複数のTiFlashノードで分散して実行されます。この変数は単一のTiFlashノード上のTopNおよびSort演算子の最大メモリ使用量を制御します。
> - この変数を`-1`に設定すると、TiFlashは、その自身の構成項目[`max_bytes_before_external_sort`](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters)の値に基づいて、TopNおよびSort演算子の最大メモリ使用量を決定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、TopNおよびSortは通常、複数のTiFlashノードで分散して実行されます。この変数は単一のTiFlashノード上のTopNおよびSort演算子の最大メモリ使用量を制御します。
> - この変数を`-1`に設定すると、TiFlashは、その自身の構成項目`max_bytes_before_external_sort`の値に基づいて、TopNおよびSort演算子の最大メモリ使用量を決定します。

</CustomContent>

### tidb_max_chunk_size

- スコープ: SESSION | GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `1024`
- 範囲: `[32, 2147483647]`
- 単位: 行
- この変数は、実行プロセス中のチャンク内の最大行数を設定するために使用されます。大きすぎる値を設定すると、キャッシュの局所性の問題が発生する可能性があります。

### tidb_max_delta_schema_count <span class="version-mark">v2.1.18およびv3.0.5で新規</span>

- スコープ: GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `1024`
- 範囲: `[100, 16384]`
- この変数は、キャッシュされるスキーマバージョン（対応するバージョンに変更されたテーブルID）の最大数を設定するために使用されます。値の範囲は100から16384です。

### tidb_max_paging_size <span class="version-mark">v6.3.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) はい
- タイプ: 整数
- デフォルト値: `50000`
- 範囲: `[1, 9223372036854775807]`
- 単位: 行
- この変数は、コプロセッサのページングリクエストプロセス中の最大行数を設定するために使用されます。値を小さく設定すると、TiDBとTiKV間のRPC数が増加しますが、値を大きく設定すると、データのロードやフルテーブルスキャンなど、一部のケースで過剰なメモリ使用量が発生します。この変数のデフォルト値は、OLTPシナリオでの性能をOLAPシナリオよりも向上させます。アプリケーションがストレージエンジンとしてTiKVのみを使用する場合、OLAPワークロードクエリの実行時にこの変数の値を増やすことを検討してください。

### tidb_max_tiflash_threads <span class="version-mark">v6.1.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 256]`
- 単位: スレッド
- この変数は、TiFlashがリクエストを実行するための最大同時性を設定するために使用されます。デフォルト値は`-1`で、これはシステム変数が無効であることを示します。値が`0`の場合、最大スレッド数はTiFlashによって自動的に構成されます。

### tidb_mem_oom_action <span class="version-mark">v6.1.0で新規</span>

- スコープ: GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 列挙型
- デフォルト値: `CANCEL`
- 可能な値: `CANCEL`, `LOG`

<CustomContent platform="tidb">

- この変数を使用して、TiDBが`tidb_mem_quota_query`で指定されたメモリクォータを超える単一のSQLステートメントの処理時に行う操作を指定します。詳細については、[TiDBメモリ制御](/configure-memory-usage.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数を使用して、TiDBが[`tidb_mem_quota_query`](#tidb_mem_quota_query)で指定されたメモリクォータを超える単一のSQLステートメントの処理時に行う操作を指定します。

</CustomContent>

- デフォルト値は`CANCEL`ですが、TiDB v4.0.2およびそれ以前のバージョンでは、デフォルト値は`LOG`です。
- この設定は以前は`tidb.toml`のオプション (`oom-action`) でしたが、TiDB v6.1.0からシステム変数に変更されました。

### tidb_mem_quota_analyze <span class="version-mark">v6.1.0で新規</span>

> **警告:**
>
> 現在、`ANALYZE`メモリクォータは実験的な機能であり、本番環境でのメモリ統計情報は不正確になる可能性があります。

- スコープ: GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- 単位: バイト
- この変数は、TiDBが統計情報の更新をコントロールするために使用されます。このようなメモリ使用量は、手動で[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)を実行したときや、TiDBがバックグラウンドで自動的にタスクを分析したときに発生します。合計メモリ使用量がこのしきい値を超えると、ユーザーが実行した`ANALYZE`は終了し、より低いサンプリング率で再試行するか後で再試行するように警告が表示されます。TiDBのバックグラウンドで自動タスクがメモリしきい値を超えて終了し、使用されたサンプリング率がデフォルト値よりも高い場合、TiDBはデフォルトのサンプリング率を使用して更新を再試行します。この変数値が負の値またはゼロの場合、TiDBは手動および自動の更新タスクのメモリ使用量を制限しません。

> **注:**
>
> `auto_analyze`は、TiDBクラスターで`run-auto-analyze`がTiDBの起動構成ファイルで有効になっているときのみトリガされます。

### tidb_mem_quota_apply_cache <span class="version-mark">v5.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化する: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `33554432` (32 MiB)
- 範囲: `[0, 9223372036854775807]`
- 単位: バイト
- この変数は、`Apply`演算子のローカルキャッシュのメモリ使用量のしきい値を設定するために使用されます。
- `Apply`演算子のローカルキャッシュは`Apply`演算子の計算を高速化するために使用されます。この変数を`0`に設定して`Apply`キャッシュ機能を無効にすることができます。
### tidb_mem_quota_binding_cache <span class="version-mark">v6.0.0で新規追加</span>

- Scope: GLOBAL
- クラスターに永続化: はい
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ: 整数
- デフォルト値: `67108864`
- 範囲: `[0, 2147483647]`
- 単位: バイト
- この変数は、バインディングのキャッシュに使用するメモリの閾値を設定するために使用されます。
- システムが過剰なバインディングを生成または取得し、メモリスペースの過剰な使用を引き起こすと、TiDBはログで警告を返します。この場合、キャッシュはすべての利用可能なバインディングを保持できず、どのバインディングを格納するかを決定できません。このため、一部のクエリでバインディングが欠落する可能性があります。この問題に対処するために、この変数の値を増やすことで、バインディングのキャッシュに使用するメモリを増やすことができます。このパラメータを変更した後は、`admin reload bindings`を実行してバインディングをリロードし、変更を検証する必要があります。

### tidb_mem_quota_query

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ: 整数
- デフォルト値: `1073741824` (1 GiB)
- 範囲: `[-1, 9223372036854775807]`
- 単位: バイト

<CustomContent platform="tidb">

- TiDB v6.1.0より前のバージョンでは、この変数はセッションスコープ変数であり、初期値として`tidb.toml`の`mem-quota-query`の値が使用されます。v6.1.0以降、`tidb_mem_quota_query`は`SESSION | GLOBAL`スコープの変数です。
- TiDB v6.5.0より前のバージョンでは、この変数は**クエリ**のメモリクォータの閾値値を設定するために使用されます。実行中のクエリのメモリクォータが閾値値を超える場合、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。
- TiDB v6.5.0およびそれ以降のバージョンでは、この変数は**セッション**のメモリクォータの閾値値を設定するために使用されます。実行中のセッションのメモリクォータが閾値値を超える場合、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。なお、TiDB v6.5.0以降、セッションのメモリ使用量にはセッション内のトランザクションが消費するメモリも含まれます。TiDB v6.5.0およびそれ以降のバージョンでトランザクションメモリ使用量の制御動作については、[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)を参照してください。
- 変数値を`0`または`-1`に設定すると、メモリ閾値は正の無限大になります。128未満の値を設定すると、値はデフォルトで`128`になります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- TiDB v6.1.0より前のバージョンでは、この変数はセッションスコープ変数です。v6.1.0以降、`tidb_mem_quota_query`は`SESSION | GLOBAL`スコープの変数です。
- TiDB v6.5.0より前のバージョンでは、この変数は**クエリ**のメモリクォータの閾値値を設定するために使用されます。実行中のクエリのメモリクォータが閾値値を超える場合、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。
- TiDB v6.5.0およびそれ以降のバージョンでは、この変数は**セッション**のメモリクォータの閾値値を設定するために使用されます。実行中のセッションのメモリクォータが閾値値を超える場合、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。なお、TiDB v6.5.0以降、セッションのメモリ使用量にはセッション内のトランザクションが消費するメモリも含まれます。
- 変数値を`0`または`-1`に設定すると、メモリ閾値は正の無限大になります。128未満の値を設定すると、値はデフォルトで`128`になります。

</CustomContent>

### tidb_memory_debug_mode_alarm_ratio

- Scope: SESSION
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ: 浮動小数点数
- デフォルト値: `0`
- この変数は、TiDBメモリデバッグモードで許容されるメモリ統計エラー値を表します。
- この変数は、TiDBの内部テストに使用されます。この変数を設定することは**推奨されません**。

### tidb_memory_debug_mode_min_heap_inuse

- Scope: SESSION
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ: 整数
- デフォルト値: `0`
- この変数は、TiDBの内部テストに使用されます。この変数を設定することは**推奨されません**。この変数を有効にすると、TiDBのパフォーマンスに影響を与えます。
- このパラメータを構成した後、TiDBはメモリトラッキングの精度を分析するためにメモリデバッグモードに入ります。TiDBは後続のSQLステートメントの実行中に頻繁にGCをトリガーし、実際のメモリ使用量とメモリ統計を比較します。現在のメモリ使用量が`tidb_memory_debug_mode_min_heap_inuse`を超え、メモリ統計エラーが`tidb_memory_debug_mode_alarm_ratio`を超える場合、TiDBは関連するメモリ情報をログとファイルに出力します。

### tidb_memory_usage_alarm_ratio

> **注記：**
>
> このTiDB変数はTiDB Cloudには適用されません。

- Scope: GLOBAL
- クラスターに永続化: はい
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ: 浮動小数点数
- デフォルト値: `0.7`
- 範囲: `[0.0, 1.0]`

<CustomContent platform="tidb">

- この変数は、TiDBメモリ使用量が全体のメモリの70%を超え、[アラーム条件](/configure-memory-usage.md#trigger-the-alarm-of-excessive-memory-usage)のいずれかが満たされた場合に、TiDBメモリアラームをトリガーするメモリ使用量比率を設定します。デフォルトでは、この変数が`0`または`1`に構成されると、メモリ閾値アラーム機能が無効になります。
- この変数を`0`より大きい値および`1`より小さい値に構成すると、メモリ閾値アラーム機能が有効になります。

    - システム変数[`tidb_server_memory_limit`](#tidb_server_memory_limit-new-in-v640)の値が`0`の場合、メモリアラーム閾値は`tidb_memory-usage-alarm-ratio * システムメモリサイズ`になります。
    - システム変数`tidb_server_memory_limit`の値が0より大きい場合、メモリアラーム閾値は`tidb_memory-usage-alarm-ratio * tidb_server_memory_limit`になります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[tidb-serverメモリアラーム](https://docs.pingcap.com/tidb/stable/configure-memory-usage#trigger-the-alarm-of-excessive-memory-usage)をトリガーするメモリ使用量比率を設定します。
- この変数を`0`より大きい値および`1`より小さい値に構成すると、メモリ閾値アラーム機能が有効になります。

</CustomContent>

### tidb_memory_usage_alarm_keep_record_num <span class="version-mark">v6.4.0で新規追加</span>

<CustomContent platform="tidb-cloud">

> **注記：**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

- Scope: GLOBAL
- クラスターに永続化: はい
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- デフォルト値: `5`
- 範囲: `[1, 10000]`
- tidb-serverメモリ使用量がメモリアラームの閾値を超え、アラームをトリガーした場合、TiDBはデフォルトで最近の5回のアラーム中に生成されたステータスファイルのみを保持します。この数は、この変数で調整できます。

### tidb_merge_join_concurrency

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：はい
- タイプ: 整数
- 範囲: `[1, 256]`
- デフォルト値: `1`
- この変数は、クエリの実行時に`MergeJoin`演算子の並行性を設定します。
- この変数の設定は**推奨されません**。この変数の値を変更すると、データの正確性に問題が生じる可能性があります。

### tidb_merge_partition_stats_concurrency

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用： [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- デフォルト値: `1`
- この変数は、TiDBがパーティションテーブルの統計情報をマージする際の並行性を指定します。

### tidb_enable_async_merge_global_stats <span class="version-mark">v7.5.0で新規追加</span>

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: ブール値
- デフォルト値: `ON`。TiDBをv7.5.0以前のバージョンからv7.5.0以上にアップグレードする際、デフォルト値は `OFF` になります。
- この変数はTiDBがグローバルな統計情報を非同期でマージしてOOMの問題を回避するために使用されます。

### tidb_metric_query_range_duration <span class="version-mark">v4.0で新規</span>

> **注意:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: セッション
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `60`
- 範囲: `[10, 216000]`
- 単位: 秒
- この変数は、`METRICS_SCHEMA` をクエリする際に生成されるPrometheusステートメントの範囲を設定するために使用されます。

### tidb_metric_query_step <span class="version-mark">v4.0で新規</span>

> **注意:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: セッション
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 整数
- デフォルト値: `60`
- 範囲: `[10, 216000]`
- 単位: 秒
- この変数は、`METRICS_SCHEMA` をクエリする際に生成されるPrometheusステートメントのステップを設定するために使用されます。

### tidb_min_paging_size <span class="version-mark">v6.2.0で新規</span>

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- 種類: 整数
- デフォルト値: `128`
- 範囲: `[1, 9223372036854775807]`
- 単位: 行
- この変数はコプロセッサのページングリクエスト処理中の最小行数を設定するために使用されます。値が小さすぎるとTiDBとTiKVの間のRPCリクエスト回数が増加し、大きすぎるとLimitを使用してIndexLookupを実行する際のパフォーマンス低下が発生する可能性があります。この変数のデフォルト値は、OLTPシナリオでOLAPシナリオよりも優れたパフォーマンスをもたらします。アプリケーションがストレージエンジンとしてTiKVのみを使用する場合、OLAPワークロードクエリを実行する際に、この変数の値を増やすことを検討してください。これにより、より良いパフォーマンスが得られる可能性があります。

![Paging size impact on TPCH](/media/paging-size-impact-on-tpch.png)

この図に示すように、[`tidb_enable_paging`](#tidb_enable_paging-new-in-v540) が有効になっている場合、`tidb_min_paging_size` および [`tidb_max_paging_size`](#tidb_max_paging_size-new-in-v630) の設定がTPCHのパフォーマンスに影響を与えます。垂直軸は実行時間であり、小さいほど良いです。

### tidb_mpp_store_fail_ttl

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 持続時間
- デフォルト値: `60s`
- 新たに起動したTiFlashノードはサービスを提供しません。クエリが失敗しないように、TiDBは新たに起動したTiFlashノードに対してクエリを送信しない時間範囲を示します。

### tidb_multi_statement_mode <span class="version-mark">v4.0.11で新規</span>

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: 列挙型
- デフォルト値: `OFF`
- 可能な値: `OFF`, `ON`, `WARN`
- この変数は、複数のクエリを同じ `COM_QUERY` 呼び出しで実行することを許可するかどうかを制御します。
- SQLインジェクション攻撃の影響を低減するため、TiDBはデフォルトで複数のクエリが同じ `COM_QUERY` 呼び出しで実行されるのを防止します。この変数はTiDBの以前のバージョンからのアップグレードパスの一部として使用されることを意図しています。次の動作が適用されます:

| クライアント設定         | `tidb_multi_statement_mode` の値 | 複数のステートメントが許可されますか? |
| ------------------------ | --------------------------------- | -------------------------------------- |
| Multiple Statements = ON | OFF                               | はい                                   |
| Multiple Statements = ON | ON                                | はい                                   |
| Multiple Statements = ON | WARN                              | はい                                   |
| Multiple Statements = OFF | OFF                               | いいえ                                 |
| Multiple Statements = OFF | ON                                | はい                                   |
| Multiple Statements = OFF | WARN                              | （警告が返されます） はい             |

> **注意:**
>
> 安全性を確保するには、デフォルト値の `OFF` のみを考慮すべきです。`tidb_multi_statement_mode=ON` を設定することが必要な場合は、アプリケーションが特定の以前のTiDBバージョン向けに特別に設計されている場合です。アプリケーションが複数のステートメントをサポートする必要がある場合、`tidb_multi_statement_mode` オプションの代わりにクライアントライブラリが提供する設定を使用することをお勧めします。例:
>
> * [go-sql-driver](https://github.com/go-sql-driver/mysql#multistatements) (`multiStatements`)
> * [Connector/J](https://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html) (`allowMultiQueries`)
> * PHP [mysqli](https://www.php.net/manual/ja/mysqli.quickstart.multiple-statement.php) (`mysqli_multi_query`)

### tidb_nontransactional_ignore_error <span class="version-mark">v6.1.0で新規</span>

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- 種類: ブール値
- デフォルト値: `OFF`
- この変数は、非トランザクショナルのDMLステートメントでエラーが発生した際にすぐにエラーを返すかどうかを指定します。
- 値が `OFF` の場合、非トランザクショナルのDMLステートメントは最初のエラーで直ちに停止し、エラーを返します。残りのバッチはキャンセルされます。
- 値が `ON` でバッチでエラーが発生した場合、後続のバッチは引き続き実行されます。実行プロセス中に発生したすべてのエラーが結果で一緒に返されます。

### tidb_opt_agg_push_down

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- 種類: ブール値
- デフォルト値: `OFF`
- この変数は、オプティマイザーが結合、プロジェクション、およびUnionAllの前に集約関数を押し下げる最適化操作を実行するかどうかを設定するために使用されます。
- クエリで集約操作が遅い場合、変数の値をONに設定できます。

### tidb_opt_broadcast_cartesian_join

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- 種類: 整数
- デフォルト値: `1`
- 範囲: `[0, 2]`
- ブロードキャストカルテシアン結合を許可するかどうかを示します。
- `0` はブロードキャストカルテシアン結合を許可しないことを意味します。 `1` は、[`tidb_broadcast_join_threshold_count`](#tidb_broadcast_join_threshold_count-new-in-v50) に基づいて許可されます。 `2` は、表のサイズがしきい値を超えていても常に許可されます。
- この変数はTiDB内部で使用されるものであり、その値を変更することは**推奨されません**。

### tidb_opt_concurrency_factor

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- 種類: 浮動小数点数
- 範囲: `[0, 18446744073709551615]`
- デフォルト値: `3.0`
- TiDBでGolangゴルーチンを起動するためのCPUコストを示します。この変数は[コストモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb_opt_copcpu_factor

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- 種類: 浮動小数点数
- 範囲: `[0, 18446744073709551615]`
- デフォルト値: `3.0`
- TiKV Coprocessorが1行を処理するためのCPUコストを示します。この変数は[コストモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb_opt_correlation_exp_factor

- スコープ: セッション | グローバル
- クラスタに永続化: はい
- 適用対象のヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- 種類: 整数
- デフォルト値: `1`
- 範囲: `[0, 2147483647]`
- 列の順序相関に基づく行数の推定方法が利用できない場合、ヒューリスティック推定方法が使用されます。この変数はヒューリスティック方法の動作を制御するために使用されます。
    - 値が0の場合、ヒューリスティック方法は使用されません。
      - 値が0より大きい場合：
        - 大きな値は、ヒューリスティックメソッドでインデックススキャンが使用される可能性が高いことを示します。
        - 小さな値は、ヒューリスティックメソッドで表スキャンが使用される可能性が高いことを示します。

### tidb_opt_correlation_threshold

- スコープ：SESSION | GLOBAL
- クラスタに永続化：あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: 浮動小数点数
- デフォルト値: `0.9`
- 範囲: `[0, 1]`
- この変数は、列順の相関を使用して行数を推定する機能を有効にするかどうかを決定するための閾値値を設定するために使用されます。現在の列と `handle` 列との間の順序相関が閾値を超えると、このメソッドが有効になります。

### tidb_opt_cpu_factor

- スコープ：SESSION | GLOBAL
- クラスタに永続化：あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `3.0`
- TiDB が1行を処理するためのCPUコストを示します。この変数は内部的に [Cost Model](/cost-model.md) で使用され、その値を変更することは**推奨されません**。

### `tidb_opt_derive_topn` <span class="version-mark">v7.0.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: ブール値
- デフォルト値: `OFF`
- [ウィンドウ関数から TopN や Limit を導出](/derive-topn-from-window.md)する最適化ルールを有効にするかどうかを制御します。

### tidb_opt_desc_factor

- スコープ：SESSION | GLOBAL
- クラスタに永続化：あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: 浮動小数点数
- 範囲: `[0, 18446744073709551615]`
- デフォルト値: `3.0`
- TiKV がディスクから1行を降順でスキャンするコストを示します。この変数は内部的に [Cost Model](/cost-model.md) で使用され、その値を変更することは**推奨されません**。

### tidb_opt_disk_factor

- スコープ：SESSION | GLOBAL
- クラスタに永続化：あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: 浮動小数点数
- 範囲: `[0, 18446744073709551615]`
- デフォルト値: `1.5`
- TiDB が一度に1バイトのデータを一時ディスクから読み取るまたは書き込むためのI/Oコストを示します。この変数は内部的に [Cost Model](/cost-model.md) で使用され、その値を変更することは**推奨されません**。

### tidb_opt_distinct_agg_push_down

- スコープ: SESSION
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`distinct`（たとえば `select count(distinct a) from t`）を含む集約関数の最適化操作の実行をオプティマイザが行うかどうかを設定するために使用されます。
- クエリで集約関数と `distinct` 操作が遅い場合、変数の値を `1` に設定できます。

`tidb_opt_distinct_agg_push_down` が有効になる前の例では、TiDB はすべてのデータを TiKV から読み込み、TiDB のサイドで `distinct` を実行する必要があります。`tidb_opt_distinct_agg_push_down` が有効になった後は、`distinct a` が Coprocessor にプッシュダウンされ、`HashAgg_5` に `group by` カラム `test.t.a` が追加されます。

```sql
mysql> desc select count(distinct a) from test.t;
+-------------------------+----------+-----------+---------------+------------------------------------------+
| id                      | estRows  | task      | access object | operator info                            |
+-------------------------+----------+-----------+---------------+------------------------------------------+
| StreamAgg_6             | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#4 |
| └─TableReader_10        | 10000.00 | root      |               | data:TableFullScan_9                     |
|   └─TableFullScan_9     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+-------------------------+----------+-----------+---------------+------------------------------------------+
3 rows in set (0.01 sec)

mysql> set session tidb_opt_distinct_agg_push_down = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> desc select count(distinct a) from test.t;
+---------------------------+----------+-----------+---------------+------------------------------------------+
| id                        | estRows  | task      | access object | operator info                            |
+---------------------------+----------+-----------+---------------+------------------------------------------+
| HashAgg_8                 | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#3 |
| └─TableReader_9           | 1.00     | root      |               | data:HashAgg_5                           |
|   └─HashAgg_5             | 1.00     | cop[tikv] |               | group by:test.t.a,                       |
|     └─TableFullScan_7     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+---------------------------+----------+-----------+---------------+------------------------------------------+
4 rows in set (0.00 sec)
```

### tidb_opt_enable_correlation_adjustment

- スコープ: SESSION | GLOBAL
- クラスタに永続化: あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、オプティマイザが列順相関に基づいて行数を推定するかどうかを制御するために使用されます。

### tidb_opt_enable_hash_join <span class="version-mark">v7.4.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、テーブルのハッシュ結合をオプティマイザが選択するかどうかを制御するために使用されます。デフォルトでは値は `ON` です。`OFF` に設定されている場合、オプティマイザは実行計画の生成時にハッシュ結合を避けますが、他の結合アルゴリズムが利用できない場合を除きます。
- システム変数 `tidb_opt_enable_hash_join` と `HASH_JOIN` ヒントの両方が構成されている場合、`HASH_JOIN` ヒントが優先されます。`tidb_opt_enable_hash_join` が `OFF` に設定されていても、クエリで `HASH_JOIN` ヒントが指定されている場合、TiDB オプティマイザは引き続きハッシュ結合プランを強制します。

### tidb_opt_enable_non_eval_scalar_subquery <span class="version-mark">v7.3.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、最適化段階で展開できる定数サブクエリの実行を `EXPLAIN` 文が無効にするかどうかを制御するために使用されます。`OFF` に設定されている場合、`EXPLAIN` 文は最適化段階でサブクエリを事前に展開します。`ON` に設定されている場合、`EXPLAIN` 文は最適化段階でサブクエリを展開しません。詳細については、[サブクエリの展開を無効にする](/explain-walkthrough.md#disable-the-early-execution-of-subqueries)を参照してください。

### tidb_opt_enable_late_materialization <span class="version-mark">v7.0.0 で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: あり
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) に適用: あり
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、[TiFlash の後発メータリアル化](/tiflash/tiflash-late-materialization.md)機能を有効にするかどうかを制御するために使用されます。TiFlash の後発メータリアル化は [fast scan モード](/tiflash/use-fastscan.md) で有効になることに留意してください。
- この変数を `OFF` に設定して TiFlash の後発メータリアル化機能を無効にした場合、`SELECT` 文を処理する際にフィルタ条件（`WHERE` 句）が関連する列データのみを最初にスキャンでき、条件に一致する行をフィルタリングし、その後これらの行の他の列のデータをスキャンして計算を行うことができます。これにより、IOスキャンやデータ処理の計算が削減されます。
### tidb_opt_enable_mpp_shared_cte_execution <span class="version-mark">v7.2.0で新規</span>

> **警告:**
>
> この変数で制御される機能は実験的です。本番環境で使用しないことをお勧めします。この機能は事前の通知なしに変更または削除される可能性があります。バグが見つかる場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、非再帰型の[共通表式（CTE）](/sql-statements/sql-statement-with.md)がTiFlash MPPで実行されるかどうかを制御します。デフォルトでは、この変数が無効になっていると、CTEはTiDBで実行され、この機能を有効にする場合と比べて大きなパフォーマンスの差があります。

### tidb_opt_fix_control <span class="version-mark">v7.1.0で新規</span>

<CustomContent platform="tidb">

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 文字列
- デフォルト値: `""`
- この変数は、最適化機能のいくつかの内部動作を制御するために使用されます。
- オプティマイザの動作は、ユーザーのシナリオやSQL文によって異なる場合があります。この変数は、オプティマイザの動作の変更によるパフォーマンスの低下を防ぎ、オプティマイザの動作を細かく制御するのに役立ちます。
- 詳細な紹介については、[オプティマイザの修正コントロール](/optimizer-fix-controls.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 文字列
- デフォルト値: `""`
- この変数は、最適化機能のいくつかの内部動作を制御するために使用されます。
- オプティマイザの動作は、ユーザーのシナリオやSQL文によって異なる場合があります。この変数は、オプティマイザの動作の変更によるパフォーマンスの低下を防ぎ、オプティマイザの動作を細かく制御するのに役立ちます。
- 詳細な紹介については、[オプティマイザの修正コントロール](https://docs.pingcap.com/tidb/v7.2/optimizer-fix-controls)を参照してください。

</CustomContent>

### tidb_opt_force_inline_cte <span class="version-mark">v6.3.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、セッション全体の共通表式（CTE）がインライン化されるかどうかを制御するために使用されます。デフォルト値は `OFF` であり、これはデフォルトでCTEのインライン化が強制されないことを意味します。ただし、`MERGE()`ヒントを指定することで依然としてCTEをインライン化することができます。変数が `ON` に設定されると、（再帰型CTE以外の）このセッション内のすべてのCTEが強制的にインライン化されます。

### tidb_opt_advanced_join_hint <span class="version-mark">v7.0.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: ブール
- デフォルト値: `ON`
- この変数は、[`HASH_JOIN()` ヒント](/optimizer-hints.md#hash_joint1_name--tl_name-)や[`MERGE_JOIN()` ヒント](/optimizer-hints.md#merge_joint1_name--tl_name-)などのJoin MethodヒントがJoin Reorder最適化プロセスに影響するかどうかを制御するために使用されます。デフォルト値は `ON` であり、影響を受けないことを意味します。`OFF`に設定すると、Join Methodヒントと`LEADING()`ヒントが同時に使用されるシナリオで競合が発生する可能性があります。

> **注意:**
>
> v7.0.0より前のバージョンの動作は、この変数を`OFF`に設定した状態と一致しています。より柔軟なヒントの動作を得るために、以前のバージョンからv7.0.0以降のクラスターにアップグレードする場合、この変数が`OFF`に設定されます。より柔軟なヒント動作を得るために、パフォーマンスの低下がない状況で、この変数を`ON`に切り替えることを強くお勧めします。

### tidb_opt_insubq_to_join_and_agg

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: ブール
- デフォルト値: `ON`
- この変数は、サブクエリを結合および集計に変換する最適化ルールを有効にするかどうかを設定するために使用されます。
- たとえば、この最適化ルールを有効にした後、サブクエリは次のように変換されます：

    ```sql
    select * from t where t.a in (select aa from t1);
    ```

    サブクエリは次のように結合されます：

    ```sql
    select t.* from t, (select aa from t1 group by aa) tmp_t where t.a = tmp_t.aa;
    ```

    もし`t1`が`aa`カラムで`unique`および`not null`に制限されている場合、次のステートメントを使用できます（集計なし）。

    ```sql
    select t.* from t, t1 where t.a=t1.aa;
    ```

### tidb_opt_join_reorder_threshold

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数は、TiDB Join Reorderアルゴリズムの選択を制御するために使用されます。Join Reorderに参加するノードの数がこの閾値よりも大きい場合、TiDBは貪欲アルゴリズムを選択し、この閾値よりも小さい場合は動的プログラミングアルゴリズムを選択します。
- 現在、OLTPクエリではデフォルト値を維持することが推奨されます。OLAPクエリでは、この変数の値を10〜15に設定することで、OLAPシナリオでより良い接続順序を得ることが推奨されます。

### tidb_opt_limit_push_down_threshold

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[0, 2147483647]`
- この変数は、LimitまたはTopN演算子をTiKVに押し下げるかどうかを決定する閾値を設定するために使用されます。
- LimitまたはTopN演算子の値がこの閾値以下の場合、これらの演算子はTiKVに強制的に押し下げられます。この変数は、誤った推定によるためにLimitまたはTopN演算子をTiKVに部分的に押し下げることができない問題を解決します。

### tidb_opt_memory_factor

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `0.001`
- 1行を格納するためのTiDBのメモリコストを示します。この変数は[コストモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb_opt_mpp_outer_join_fixed_build_side <span class="version-mark">v5.1.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: ブール
- デフォルト値: `OFF`
- 変数の値が`ON`の場合、左結合演算子は常にビルドサイドとして内部テーブルを使用し、右結合演算子は常に外部テーブルをビルドサイドとして使用します。`OFF`に設定すると、外部結合演算子はテーブルのどちらかの側をビルドサイドとして使用できます。

### tidb_opt_network_factor

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `1.0`
- 1バイトのデータをネットワークを介して転送するためのネットコストを示します。この変数は[コストモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb_opt_objective <span class="version-mark">v7.4.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) は有効
- タイプ: 列挙型
- デフォルト値: `moderate`
- 可能な値: `moderate`, `determinate`
- この変数はオプティマイザの目的を制御します。 `moderate` は、TiDB v7.4.0より前のバージョンのデフォルト動作を維持します。つまり、オプティマイザはより多くの情報を使用してより良い実行計画を生成しようとします。 `determinate` モードはより慎重で実行計画をより安定させる傾向があります。
- リアルタイム統計情報は、DMLステートメントに基づいて自動的に更新される総行数および変更された行数です。この変数が `moderate` （デフォルト）に設定されている場合、TiDBはリアルタイム統計情報に基づいて実行計画を生成します。この変数を `determinate` に設定すると、TiDBは実行計画の生成にリアルタイム統計情報を使用しないため、実行計画がより安定します。
- 長期安定型のOLTPワークロード、またはユーザーが既存の実行計画に肯定的である場合、予期せぬ実行計画の変更の可能性を減らすために `determinate` モードを使用することをお勧めします。さらに、[`LOCK STATS`](/sql-statements/sql-statement-lock-stats.md) を使用して統計情報の変更を防止し、実行計画をさらに安定化させることができます。

### tidb_opt_ordering_index_selectivity_threshold <span class="version-mark">v7.0.0で新規</span>

- 対象: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 浮動小数点数
- デフォルト値: `0`
- 範囲: `[0, 1]`
- この変数は、SQLステートメントで `ORDER BY` および `LIMIT` 句にフィルタ条件が存在する場合に、オプティマイザがインデックスを選択する方法を制御するために使用されます。
- このようなクエリの場合、オプティマイザは `ORDER BY` および `LIMIT` 句を満たす対応するインデックスを選択することを検討します（そのインデックスがフィルタ条件を満たさない場合でも）。ただし、データの分布の複雑さにより、このシナリオではオプティマイザが非効率なインデックスを選択する可能性があります。
- この変数は閾値を表します。フィルタ条件を満たすインデックスが存在し、その選択度の推定がこの閾値より低い場合、オプティマイザは `ORDER BY` と `LIMIT` 句の両方を満たすために選択されたインデックスを避けます。代わりにフィルタ条件を満たすインデックスを優先します。
- たとえば、変数が `0` に設定されている場合、オプティマイザはデフォルト動作を維持します。`1` に設定すると、オプティマイザは常にフィルタ条件を満たすインデックスを選択し、`ORDER BY` と `LIMIT` 句を満たすインデックスを選択しないようにします。
- 次の例では、テーブル `t` には合計 1,000,000 行が含まれています。列 `b` のインデックスを使用すると、その推定行数は約 8,748 なので、選択度の推定値は約 0.0087 です。デフォルトでは、オプティマイザは列 `a` のインデックスを選択します。ただし、この変数を 0.01 に設定した後、列 `b` のインデックスの選択度（0.0087）が 0.01 よりも低いため、オプティマイザは列 `b` のインデックスを選択します。

```sql
> EXPLAIN SELECT * FROM t WHERE b <= 9000 ORDER BY a LIMIT 1;
+-----------------------------------+---------+-----------+----------------------+--------------------+
| id                                | estRows | task      | access object        | operator info      |
+-----------------------------------+---------+-----------+----------------------+--------------------+
| Limit_12                          | 1.00    | root      |                      | offset:0, count:1  |
| └─Projection_25                   | 1.00    | root      |                      | test.t.a, test.t.b |
|   └─IndexLookUp_24                | 1.00    | root      |                      |                    |
|     ├─IndexFullScan_21(Build)     | 114.30  | cop[tikv] | table:t, index:ia(a) | keep order:true    |
|     └─Selection_23(Probe)         | 1.00    | cop[tikv] |                      | le(test.t.b, 9000) |
|       └─TableRowIDScan_22         | 114.30  | cop[tikv] | table:t              | keep order:false   |
+-----------------------------------+---------+-----------+----------------------+--------------------+

> SET SESSION tidb_opt_ordering_index_selectivity_threshold = 0.01;

> EXPLAIN SELECT * FROM t WHERE b <= 9000 ORDER BY a LIMIT 1;
+----------------------------------+---------+-----------+----------------------+-------------------------------------+
| id                               | estRows | task      | access object        | operator info                       |
+----------------------------------+---------+-----------+----------------------+-------------------------------------+
| TopN_9                           | 1.00    | root      |                      | test.t.a, offset:0, count:1         |
| └─IndexLookUp_20                 | 1.00    | root      |                      |                                     |
|   ├─IndexRangeScan_17(Build)     | 8748.62 | cop[tikv] | table:t, index:ib(b) | range:[-inf,9000], keep order:false |
|   └─TopN_19(Probe)               | 1.00    | cop[tikv] |                      | test.t.a, offset:0, count:1         |
|     └─TableRowIDScan_18          | 8748.62 | cop[tikv] | table:t              | keep order:false                    |
+----------------------------------+---------+-----------+----------------------+-------------------------------------+
```

### tidb_opt_prefer_range_scan <span class="version-mark">v5.0で新規</span>

- 対象: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数の値を `ON` に設定すると、オプティマイザは常に完全テーブルスキャンよりも範囲スキャンを優先します。
- 次の例では、`tidb_opt_prefer_range_scan` を有効にする前は、TiDBオプティマイザは完全テーブルスキャンを実行します。 `tidb_opt_prefer_range_scan` を有効にした後は、オプティマイザがインデックス範囲スキャンを選択します。

```sql
explain select * from t where age=5;
+-------------------------+------------+-----------+---------------+-------------------+
| id                      | estRows    | task      | access object | operator info     |
+-------------------------+------------+-----------+---------------+-------------------+
| TableReader_7           | 1048576.00 | root      |               | data:Selection_6  |
| └─Selection_6           | 1048576.00 | cop[tikv] |               | eq(test.t.age, 5) |
|   └─TableFullScan_5     | 1048576.00 | cop[tikv] | table:t       | keep order:false  |
+-------------------------+------------+-----------+---------------+-------------------+
3 rows in set (0.00 sec)

set session tidb_opt_prefer_range_scan = 1;

explain select * from t where age=5;
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
| id                            | estRows    | task      | access object               | operator info                 |
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
| IndexLookUp_7                 | 1048576.00 | root      |                             |                               |
| ├─IndexRangeScan_5(Build)     | 1048576.00 | cop[tikv] | table:t, index:idx_age(age) | range:[5,5], keep order:false |
| └─TableRowIDScan_6(Probe)     | 1048576.00 | cop[tikv] | table:t                     | keep order:false              |
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
3 rows in set (0.00 sec)
```

### tidb_opt_prefix_index_single_scan <span class="version-mark">v6.4.0で新規</span>

- 対象: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `ON`
- この変数は、TiDBオプティマイザがいくつかのフィルタ条件をプレフィックスインデックスにプッシュダウンして不要なテーブル検索を避け、クエリのパフォーマンスを向上させるかどうかを制御します。
- この変数の値が `ON` に設定されている場合、一部のフィルタ条件がプレフィックスインデックスにプッシュダウンされます。`col` 列がテーブルのインデックスプレフィックス列であるとします。クエリ内の `col is null` または `col is not null` 条件は、テーブル検索のフィルタ条件ではなくインデックスのフィルタ条件として処理されるため、不要なテーブル検索が回避されます。

<details>
<summary><code>tidb_opt_prefix_index_single_scan</code> の使用例</summary>

プレフィックスインデックスを持つテーブルを作成します：

```sql
CREATE TABLE t (a INT, b VARCHAR(10), c INT, INDEX idx_a_b(a, b(5)));
```

`tidb_opt_prefix_index_single_scan` を無効にします：

```sql
SET tidb_opt_prefix_index_single_scan = 'OFF';
```

次のクエリに対して、実行計画はプレフィックスインデックス `idx_a_b` を使用しますが、テーブル検索（`IndexLookUp` 演算子が表示されます）が必要です。

```sql
EXPLAIN FORMAT='brief' SELECT COUNT(1) FROM t WHERE a = 1 AND b IS NOT NULL;
```
```markdown
+----------------------------+---------+-------------------+------------------------------------------+-----------------------+
| id                         | estRows | task              | access object                            | operator info         |
+----------------------------+---------+-------------------+------------------------------------------+-----------------------+
| HashAgg                    | 1.00    | root              |                                          | funcs:count(Column#8)->Column#5 |
| └─IndexLookUp              | 1.00    | root              |                                          |                       |
|   ├─IndexRangeScan(Build)  | 99.90   | cop[tikv]         | table:t, index:idx_a_b(a, b)             | range:[1 -inf,1 +inf], keep order:false, stats:pseudo |
|   └─HashAgg(Probe)         | 1.00    | cop[tikv]         |                                          | funcs:count(1)->Column#8 |
|     └─Selection            | 99.90   | cop[tikv]         |                                          | not(isnull(test.t.b))  |
|       └─TableRowIDScan     | 99.90   | cop[tikv]         | table:t                                  | keep order:false, stats:pseudo |
+----------------------------+---------+-------------------+------------------------------------------+-----------------------+
6 rows in set (0.00 sec)
```

`tidb_opt_prefix_index_single_scan`を有効にします：

```sql
SET tidb_opt_prefix_index_single_scan = 'ON';
```

この変数を有効にした後、次のクエリでは実行プランがプレフィックスインデックス `idx_a_b` を使用しますが、表の検索は必要ありません。

```sql
EXPLAIN FORMAT='brief' SELECT COUNT(1) FROM t WHERE a = 1 AND b IS NOT NULL;
+---------------------------------+---------+-----------+--------------------------+-------------------------------------------------------------------+
| id                              | estRows | task      | access object            | operator info                                                     |
+---------------------------------+---------+-----------+--------------------------+-------------------------------------------------------------------+
| StreamAgg                       | 1.00    | root      |                          | funcs:count(Column#7)->Column#5                                   |
| └─IndexReader                   | 1.00    | root      |                          | index:StreamAgg                                                   |
|   └─StreamAgg                   | 1.00    | cop[tikv] |                          | funcs:count(1)->Column#7                                          |
|     └─IndexRangeScan            | 99.90   | cop[tikv] | table:t, index:idx_a_b(a, b) | range:[1 -inf,1 +inf], keep order:false, stats:pseudo             |
+---------------------------------+---------+-----------+--------------------------+-------------------------------------------------------------------+
4 rows in set (0.00 sec)
```

</details>

### tidb_opt_projection_push_down <span class="version-mark">v6.1.0で導入</span>

- スコープ：セッション
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用：可
- タイプ：真偽値
- デフォルト値：`OFF`
- オプティマイザにプロジェクションをTiKVまたはTiFlashコプロセッサにプッシュダウンすることを許可するかどうかを指定します。

### tidb_opt_range_max_size <span class="version-mark">v6.4.0で導入</span>

- スコープ：セッション | グローバル
- クラスターに永続化：可
- ヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用：可
- デフォルト値：`67108864` (64 MiB)
- スコープ: `[0, 9223372036854775807]`
- 単位：バイト
- この変数は、オプティマイザがスキャン範囲を構築するために使用するメモリ使用量の上限を設定するために使用されます。変数値が`0`の場合、スキャン範囲を構築するためのメモリ制限はありません。正確なスキャン範囲の構築に必要なメモリが制限を超える場合、オプティマイザはより緩やかなスキャン範囲（たとえば`[[NULL,+inf]]`など）を使用します。実行プランが正確なスキャン範囲を使用していない場合は、この変数の値を増やしてオプティマイザに正確なスキャン範囲を構築させることができます。

この変数の使用例は次のとおりです。

<details>
<summary><code>tidb_opt_range_max_size</code> 使用例</summary>

この変数のデフォルト値を表示します。結果から、オプティマイザがスキャン範囲を構築するために最大64MiBのメモリを使用していることがわかります。

```sql
SELECT @@tidb_opt_range_max_size;
```

```sql
+----------------------------+
| @@tidb_opt_range_max_size |
+----------------------------+
| 67108864                   |
+----------------------------+
1 row in set (0.01 sec)
```

```sql
EXPLAIN SELECT * FROM t use index (idx) WHERE a IN (10,20,30) AND b IN (40,50,60);
```

64MiBのメモリ上限で、オプティマイザは次の正確なスキャン範囲`[10 40,10 40], [10 50,10 50], [10 60,10 60], [20 40,20 40], [20 50,20 50], [20 60,20 60], [30 40,30 40], [30 50,30 50], [30 60,30 60]`を構築し、次の実行プラン結果に示すように使用します。

```sql
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                            | estRows | task      | access object            | operator info                                                                                                                                                               |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| IndexLookUp_7                 | 0.90    | root      |                          |                                                                                                                                                                             |
| ├─IndexRangeScan_5(Build)     | 0.90    | cop[tikv] | table:t, index:idx(a, b) | range:[10 40,10 40], [10 50,10 50], [10 60,10 60], [20 40,20 40], [20 50,20 50], [20 60,20 60], [30 40,30 40], [30 50,30 50], [30 60,30 60], keep order:false, stats:pseudo |
| └─TableRowIDScan_6(Probe)     | 0.90    | cop[tikv] | table:t                  | keep order:false, stats:pseudo                                                                                                                                              |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set (0.00 sec)
```

次に、オプティマイザが正確なスキャン範囲を構築するために必要なメモリ使用量が`tidb_opt_range_max_size`の制限を超えることを通知する警告を使用して、オプティマイザがより緩やかなスキャン範囲を構築するようにメモリ上限を1500バイトに設定します。

```sql
SET @@tidb_opt_range_max_size = 1500;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
EXPLAIN SELECT * FROM t USE INDEX (idx) WHERE a IN (10,20,30) AND b IN (40,50,60);
```

1500バイトのメモリ上限で、オプティマイザはより緩和されたスキャン範囲`[10,10], [20,20], [30,30]`を構築し、`tidb_opt_range_max_size`の制限を超える正確なスキャン範囲を構築するために必要なメモリ使用量が警告を使用して通知します。

```sql
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------+
| id                            | estRows | task      | access object            | operator info                                                   |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------+
| IndexLookUp_8                 | 0.09    | root      |                          |                                                                 |
| ├─Selection_7(Build)          | 0.09    | cop[tikv] |                          | in(test.t.b, 40, 50, 60)                                        |
| │ └─IndexRangeScan_5          | 30.00   | cop[tikv] | table:t, index:idx(a, b) | range:[10,10], [20,20], [30,30], keep order:false, stats:pseudo |
| └─TableRowIDScan_6(Probe)     | 0.09    | cop[tikv] | table:t                  | keep order:false, stats:pseudo                                  |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

```sql
SHOW WARNINGS;
```

```sql
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                     |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1105 | Memory capacity of 1500 bytes for 'tidb_opt_range_max_size' exceeded when building ranges. Less accurate ranges such as full range are chosen |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

次に、メモリ使用量の上限を100バイトに設定します。

```sql
SET @@tidb_opt_range_max_size = 100;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
EXPLAIN SELECT * FROM t USE INDEX (idx) WHERE a IN (10,20,30) AND b IN (40,50,60);
```

100バイトのメモリ上限で、オプティマイザは`IndexFullScan`を選択し、`tidb_opt_range_max_size`の制限を超える正確なスキャン範囲を構築するために必要なメモリ使用量が警告を使用して通知します。

```sql
```
```markdown
+-------------------------------+----------+-----------+--------------------------+----------------------------------------------------+
| id                            | estRows  | task      | access object            | operator info                                      |
+-------------------------------+----------+-----------+--------------------------+----------------------------------------------------+
| IndexLookUp_8                 | 8000.00  | root      |                          |                                                    |
| ├─Selection_7(Build)          | 8000.00  | cop[tikv] |                          | in(test.t.a, 10, 20, 30), in(test.t.b, 40, 50, 60) |
| │ └─IndexFullScan_5           | 10000.00 | cop[tikv] | table:t, index:idx(a, b) | keep order:false, stats:pseudo                     |
| └─TableRowIDScan_6(Probe)     | 8000.00  | cop[tikv] | table:t                  | keep order:false, stats:pseudo                     |
+-------------------------------+----------+-----------+--------------------------+----------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

```sql
SHOW WARNINGS;
```

```markdown
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                     |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1105 | Memory capacity of 100 bytes for 'tidb_opt_range_max_size' exceeded when building ranges. Less accurate ranges such as full range are chosen |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

</details>

### tidb_opt_scan_factor

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `1.5`
- これはTiKVがディスクからデータを昇順でスキャンする際のコストを示します。この変数は内部でコストモデルで使用され、その値を変更することは**推奨されていません**。

### tidb_opt_seek_factor

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `20`
- これはTiDBがTiKVからデータを要求する際の起動コストを示します。この変数は内部でコストモデルで使用され、その値を変更することは**推奨されていません**。

### tidb_opt_skew_distinct_agg <span class="version-mark">v6.2.0で導入</span>

> **注記:**
>
> この変数を有効にすることによるクエリのパフォーマンス最適化は、**TiFlashのみに有効**です。

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`DISTINCT`を持つ集約関数を2段階の集約関数に書き換えるかどうかを設定します。例えば、`SELECT b, COUNT(DISTINCT a) FROM t GROUP BY b` を `SELECT b, COUNT(a) FROM (SELECT b, a FROM t GROUP BY b, a) t GROUP BY b` に書き換えることで、集約列が深刻なスキューを持ち、`DISTINCT`列に多くの異なる値がある場合、この書き換えによりクエリ実行時のデータスキューを回避し、クエリのパフォーマンスを向上させることができます。

### tidb_opt_three_stage_distinct_agg <span class="version-mark">v6.3.0で導入</span>

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、MPPモードで`COUNT(DISTINCT)`集約を3段階の集約に書き換えるかどうかを指定します。
- この変数は現在、1つの`COUNT(DISTINCT)`のみを含む集約に適用されます。

### tidb_opt_tiflash_concurrency_factor

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `24.0`
- これはTiFlash計算の並列数を示します。この変数は内部でコストモデルで使用され、その値を変更することは**推奨されていません**。

### tidb_opt_write_row_id

> **注記:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: SESSION
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`INSERT`、`REPLACE`、`UPDATE`文が`_tidb_rowid`列で操作を許可するかどうかを制御します。この変数は、TiDBツールを使用してデータをインポートする際にのみ使用できます。

### tidb_optimizer_selectivity_level

- スコープ: SESSION
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数は、最適化の推定ロジックの反復数を制御します。この変数の値を変更すると、最適化の推定ロジックが大幅に変わります。現在は`0`が唯一の有効な値です。他の値に設定することは推奨されません。

### tidb_partition_prune_mode <span class="version-mark">v5.1で導入</span>

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用
- タイプ: 列挙型
- デフォルト値: `dynamic`
- 可能な値: `static`, `dynamic`, `static-only`, `dynamic-only`
- パーティション化されたテーブルに対して`dynamic`または`static`モードを使用するかを指定します。なお、動的パーティショニングは、テーブルレベルの統計情報、またはGlobalStatsが収集された後にのみ有効です。GlobalStatsが収集される前は、TiDBは`static`モードを代わりに使用します。グローバル統計情報について詳しくは、[Collect statistics of partitioned tables in dynamic pruning mode](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode)を、動的プルーニングモードについては、[Dynamic Pruning Mode for Partitioned Tables](/partitioned-table.md#dynamic-pruning-mode)を参照してください。

### tidb_persist_analyze_options <span class="version-mark">v5.4.0で導入</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用されません
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、[ANALYZE構成の永続化](/statistics.md#persist-analyze-configurations)機能を有効にするかどうかを制御します。

### tidb_pessimistic_txn_fair_locking <span class="version-mark">v7.0.0で導入</span>

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) には適用されません
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、悲観的トランザクション向けの待ちロックを公平にするための改善された機構を使用するかどうかを決定します。このモデルは、悲観的ロックのシングルポイントコンフリクトシナリオにおいて悲観的なトランザクションのウェイクアップ順序を厳密に制御し、既存のウェイクアップ機構のランダム性からもたらされる不確実性を大幅に低減します。ビジネスシナリオで頻繁な単一ポイントの悲観的なロックの衝突（例: 同じデータ行の頻繁な更新）に遭遇し、その結果、頻繁なステートメントのリトライ、高いテールレイテンシー、または時折`pessimistic lock retry limit reached`エラーが発生する場合、この変数を有効にして問題を解決することができます。
- この変数は、v7.0.0以降のバージョンへアップグレードされたTiDBクラスタについてはデフォルトで無効になっています。

> **注記:**
>
> - このオプションを有効にすることは、特定のビジネスシナリオによっては、頻繁なロック競合を持つトランザクションのスループットの一定程度の低下（平均レイテンシーの増加）をもたらす可能性があります。
> - このオプションは、単一のキーをロックする必要のあるステートメントにのみ影響を与えます。複数の行を同時にロックする必要があるステートメントにはこのオプションは影響しません。
> - この機能は、v6.6.0で導入された[`tidb_pessimistic_txn_aggressive_locking`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660)変数によって導入され、デフォルトでは無効になっています。

### tidb_placement_mode <span class="version-mark">v6.0.0で導入</span>

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)には読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスタへの永続化: はい
```
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 列挙
- デフォルト値: `STRICT`
- 可能な値: `STRICT`, `IGNORE`
- この変数は、SQLで指定された[配置ルール](/placement-rules-in-sql.md)をDDLステートメントが無視するかどうかを制御します。変数の値が `IGNORE` の場合、すべての配置ルールオプションは無視されます。
- この変数は、論理ダンプ/リストアツールによって使用され、無効な配置ルールが割り当てられていても常にテーブルを作成できるようにします。これは、たとえば、mysqldump がすべてのダンプファイルの先頭に `SET FOREIGN_KEY_CHECKS=0;` を書き込む方法と類似しています。

### `tidb_plan_cache_invalidation_on_fresh_stats` <span class="version-mark">v7.1.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、関連するテーブルの統計情報が更新されたときに、計画キャッシュを自動的に無効化するかどうかを制御します。
- この変数を有効にした後、計画キャッシュは統計情報をより十分に利用して実行計画を生成できます。たとえば：
    - 統計情報が利用可能でない状態で実行計画が生成された場合は、統計情報が利用可能になった後に計画キャッシュが再び実行計画を生成します。
    - テーブルのデータ分布が変更され、以前は最適だった実行計画が非最適になった場合、統計情報が再収集された後に計画キャッシュが実行計画を再生成します。
- この変数は、v7.1.0 以降に v7.1.0 にアップグレードされた TiDB クラスターにはデフォルトで無効になっています。

### `tidb_plan_cache_max_plan_size` <span class="version-mark">v7.1.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `2097152` (つまり 2 MB)
- 範囲: `[0, 9223372036854775807]`、バイト単位。単位を "KB|MB|GB|TB" で指定するメモリ形式もサポートされています。`0` は制限なしを意味します。
- この変数は、準備済みまたは非準備済み計画キャッシュ内にキャッシュできる計画の最大サイズを制御します。計画のサイズがこの値を超えると、計画はキャッシュされません。詳細は、[準備済み計画キャッシュのメモリ管理](/sql-prepared-plan-cache.md#memory-management-of-prepared-plan-cache)および[非準備済み計画キャッシュ](/sql-plan-management.md#usage)を参照してください。

### tidb_pprof_sql_cpu <span class="version-mark">v4.0 で新規</span>

> **注記:**
>
> この TiDB 変数は TiDB Cloud には適用されません。

- スコープ: GLOBAL
- クラスターに永続する: いいえ、現在接続している TiDB インスタンスにのみ適用されます。
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 1]`
- この変数は、プロファイル出力で対応する SQL 文を識別し、パフォーマンスの問題を特定およびトラブleshootするかどうかを制御します。

### tidb_prefer_broadcast_join_by_exchange_data_size <span class="version-mark">v7.1.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- デフォルト値: `OFF`
- この変数は、TiDB が[MPP ハッシュ結合アルゴリズム](/tiflash/use-tiflash-mpp-mode.md#algorithm-support-for-the-mpp-mode)を選択する際に、ネットワーク転送オーバーヘッドが最小のアルゴリズムを使用するかどうかを制御します。この変数が有効にされている場合、TiDB はそれぞれ `Broadcast Hash Join` と `Shuffled Hash Join` を使用してネットワークで交換されるデータのサイズを見積もり、その後より小さいサイズの方を選択します。
- この変数が有効になると、この変数以降に[`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50)と[`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50)は効果を持ちません。

### tidb_prepared_plan_cache_memory_guard_ratio <span class="version-mark">v6.1.0 で新規</span>

- スコープ: GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 浮動小数点数
- デフォルト値: `0.1`
- 範囲: `[0, 1]`
- 準備済み計画キャッシュがメモリ保護メカニズムをトリガするしきい値。詳細については、[準備済み計画キャッシュのメモリ管理](/sql-prepared-plan-cache.md)を参照してください。
- この設定は以前は `tidb.toml` オプション (`prepared-plan-cache.memory-guard-ratio`) でしたが、TiDB v6.1.0 からシステム変数に変更されました。

### tidb_prepared_plan_cache_size <span class="version-mark">v6.1.0 で新規</span>

> **警告:**
>
> v7.1.0 から、この変数は非推奨となります。設定には代わりに[`tidb_session_plan_cache_size`](#tidb_session_plan_cache_size-new-in-v710)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[1, 100000]`
- セッション内にキャッシュできる計画の最大数。詳細については、[準備済み計画キャッシュのメモリ管理](/sql-prepared-plan-cache.md)を参照してください。
- この設定は以前は `tidb.toml` オプション (`prepared-plan-cache.capacity`) でしたが、TiDB v6.1.0 からシステム変数に変更されました。

### tidb_projection_concurrency

> **警告:**
>
> v5.0 から、この変数は非推奨となります。設定には代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 256]`
- 単位: スレッド
- この変数は、`Projection` 演算子の並列実行を設定するために使用されます。
- 値が `-1` の場合、`tidb_executor_concurrency` の値が使用されます。

### tidb_query_log_max_len

- スコープ: GLOBAL
- クラスターに永続する: はい
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `4096` (4 KiB)
- 範囲: `[0, 1073741824]`
- 単位: バイト
- SQL ステートメント出力の最大長。ステートメントの出力長が `tidb_query_log_max_len` の値より大きい場合、ステートメントは切り捨てられて出力されます。
- この設定は以前は `tidb.toml` オプション (`log.query-log-max-len`) でも利用可能でしたが、TiDB v6.1.0 からはシステム変数のみとなりました。

### tidb_rc_read_check_ts <span class="version-mark">v6.0.0 で新規</span>

> **警告:**
>
> - この機能は [`replica-read`](#tidb_replica_read-new-in-v40) と互換性がありません。`tidb_rc_read_check_ts` と `replica-read` を同時に有効にしないでください。
> - クライアントがカーソルを使用している場合、前のバッチの返されたデータがクライアントによって既に使用されている可能性があるため、`tidb_rc_read_check_ts` を有効にすることはお勧めしません。
> - v7.0.0 以降、この変数は既存のカーソルフェッチ読み取りモードにはもう有効ではありません。

- スコープ: GLOBAL
- クラスターに永続する: いいえ、現在接続している TiDB インスタンスにのみ適用されます。
- 適用されるヒント [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、読み取りコミットされた分離レベルのシナリオでタイムスタンプの取得を最適化するために使用され、読み書きの競合がまれな場合に適しています。この変数を有効にすると、グローバルタイムスタンプを取得する遅延とコストを回避し、トランザクションレベルの読み取り待ち時間を最適化できます。
- 読み書きの競合が激しい場合、この機能を有効にするとグローバルタイムスタンプの取得コストと待ち時間が増加し、パフォーマンスの低下を引き起こす可能性があります。詳細については、[コミットされた読み取り分離レベル](/transaction-isolation-levels.md#read-committed-isolation-level)を参照してください。

### tidb_rc_write_check_ts <span class="version-mark">v6.3.0 で新規</span>

> **警告:**
>
> この機能は、[`replica-read`](#tidb_replica_read-new-in-v40)と現在互換性がありません。この変数が有効になった後は、クライアントが送信するすべてのリクエストで`replica-read`を使用することはできません。したがって、`tidb_rc_write_check_ts`と`replica-read`を同時に有効にしないでください。

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、タイムスタンプの取得を最適化するために使用され、悲観的トランザクションの`READ-COMMITTED`分離レベルでポイント書き込みの競合が少ないシナリオに適しています。この変数を有効にすると、ポイント書き込み文の実行中にグローバルタイムスタンプを取得することによってもたらされる遅延とオーバーヘッドを回避できます。現在、この変数は3種類のポイント書き込み文に適用されます:`UPDATE`, `DELETE`, および `SELECT ...... FOR UPDATE`。ポイント書き込み文とは、主キーまたはユニークキーをフィルタ条件として使用し、最終実行演算子に `POINT-GET` を含む書き込み文のことを指します。
- もしポイント書き込みの競合が深刻ならば、この変数を有効にすると追加のオーバーヘッドと遅延が発生し、性能の低下を招く可能性があります。詳細については、「[Read Committed isolation level](/transaction-isolation-levels.md#read-committed-isolation-level)」を参照してください。

### tidb_read_consistency <span class="version-mark">v5.4.0で新規</span>

- スコープ: SESSION
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 文字列
- デフォルト値: `strict`
- この変数は、自動コミットリード文のリード一貫性を制御するために使用されます。
- 変数値が`weak`に設定されている場合、リード文で遭遇するロックは直接スキップされ、リード実行が速くなる可能性があります。これが弱い一貫性リードモードです。ただし、トランザクションの意味論（例:原子性）と分散一貫性（例:線形性）は保証されません。
- 自動コミットリードが迅速に返す必要があるユーザーシナリオでは、弱い一貫性リード結果が許容される場合、弱い一貫性リードモードを使用できます。

### tidb_read_staleness <span class="version-mark">v5.4.0で新規</span>

- スコープ: SESSION
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[-2147483648, 0]`
- この変数は、現在のセッションでTiDBが読み取れる時系列データの時間範囲を設定するために使用されます。この変数が許可する範囲からできるだけ新しいタイムスタンプがTiDBに選択され、その後のすべての読み取り操作はこのタイムスタンプに対して実行されます。たとえば、この変数の値を`-5`に設定すると、TiKVが対応する時系列バージョンのデータを持っている条件下では、TiDBは5秒範囲内でできるだけ新しいタイムスタンプを選択します。

### tidb_record_plan_in_slow_log

> **注記:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターへの永続化: いいえ。接続中の現在のTiDBインスタンスにのみ適用されます。
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、スローログに遅いクエリの実行計画を含めるかどうかを制御するために使用されます。

### tidb_redact_log

> **注記:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、TiDBログおよびスローグに記録されるSQL文のユーザー情報を非表示にするかどうかを制御します。
- 変数を`1`に設定すると、ユーザー情報が非表示になります。たとえば、実行されるSQL文が`insert into t values (1,2)`の場合、ログには`insert into t values (?,?)`と記録されます。

### tidb_regard_null_as_point <span class="version-mark">v5.4.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、オプティマイザがインデックスアクセスの接頭条件としてnull同値性を含むクエリ条件を使用できるかどうかを制御します。
- この変数はデフォルトで有効化されています。有効化されている場合、オプティマイザはアクセスするべきインデックスデータのボリュームを減らすことができ、クエリの実行を早めます。たとえば、複数列インデックス `index(a, b)` が関与するクエリに`a<=>null and b=1`が含まれている場合、オプティマイザは`a<=>null`と`b=1`の両方をクエリ条件でインデックスアクセスに使用できます。変数が無効にされている場合、`a<=>null and b=1`がnull同値条件を含んでいるため、オプティマイザは`b=1`をインデックスアクセスに使用しません。

### tidb_remove_orderby_in_subquery <span class="version-mark">v6.1.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: v7.2.0より前は`OFF`、v7.2.0以降はデフォルト値が`ON`
- サブクエリの`ORDER BY`句を削除するかどうかを指定します。

### tidb_replica_read <span class="version-mark">v4.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 列挙型
- デフォルト値: `leader`
- 可能な値: `leader`, `follower`, `leader-and-follower`, `prefer-leader`, `closest-replicas`, `closest-adaptive`、および`learner`。`learner`の値はv6.6.0で導入されました。
- この変数は、TiDBがデータを読み取る場所を制御するために使用されます。
- 使用法と実装の詳細については、「[Follower read](/follower-read.md)」を参照してください。

### tidb_restricted_read_only <span class="version-mark">v5.2.0で新規</span>

> **注記:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- `tidb_restricted_read_only`と[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)の動作は似ています。ほとんどの場合、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)のみを使用する必要があります。
- `SUPER`または`SYSTEM_VARIABLES_ADMIN`権限を持つユーザーはこの変数を変更できます。ただし、[Security Enhanced Mode](#tidb_enable_enhanced_security)が有効になっている場合は、追加の`RESTRICTED_VARIABLES_ADMIN`権限が必要です。
- `tidb_restricted_read_only`は、次の場合に[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)に影響します。
    - `tidb_restricted_read_only`を`ON`に設定すると、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)も`ON`に更新されます。
    - `tidb_restricted_read_only`を`OFF`に設定すると、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)は変更されません。
    - もし`tidb_restricted_read_only`が`ON`である場合、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)を`OFF`に設定することはできません。
- TiDBのDBaaSプロバイダーが、TiDBクラスターが他のデータベースの下流データベースである場合、TiDBクラスターを読み取り専用にするには、[Security Enhanced Mode](#tidb_enable_enhanced_security)を有効にし、お客様が[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)を使用してクラスターを書き込み可能にすることを防ぐために、`tidb_restricted_read_only`を使用する必要がある場合があります。これには、[Security Enhanced Mode](#tidb_enable_enhanced_security)の有効化、`SYSTEM_VARIABLES_ADMIN`および`RESTRICTED_VARIABLES_ADMIN`権限を持つ管理者ユーザーの使用、およびデータベースユーザーが`SUPER`権限を持つrootユーザーを使用して[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)を制御する必要があります。
- この変数は、クラスタ全体の読み込み専用状態を制御します。変数が「ON」の場合、クラスタ全体のすべてのTiDBサーバーは読み込み専用モードになります。この場合、TiDBは「SELECT」、「USE」、「SHOW」などのデータを変更しないステートメントのみを実行します。また、「INSERT」や「UPDATE」などのステートメントについては、読み込み専用モードでこれらのステートメントを実行することを拒否します。

- この変数を使用して読み込み専用モードを有効にすると、クラスタ全体が最終的に読み込み専用状態に入ります。TiDBクラスタでこの変数の値を変更したが、変更が他のTiDBサーバーにまだ伝播していない場合、未更新のTiDBサーバーは引き続き読み込み専用モードにはなりません。

- この変数が有効になっている場合、実行中のSQLステートメントは影響を受けません。TiDBは、実行されるSQLステートメントに対してのみ読み込み専用チェックを行います。

- この変数が有効になっている場合、TiDBは未コミットのトランザクションを次のように処理します：
    - 未コミットの読み込み専用トランザクションについては、通常通りトランザクションをコミットできます。
    - 読み込み専用でない未コミットのトランザクションについては、これらのトランザクションで書き込み操作を行うSQLステートメントは拒否されます。
    - 変更されたデータを含む未コミットの読み込み専用トランザクションについては、これらのトランザクションのコミットが拒否されます
- 読み込み専用モードが有効になった後、すべてのユーザー（「SUPER」特権を持つユーザーも含む）は、そのユーザーが明示的に「RESTRICTED_REPLICA_WRITER_ADMIN」特権が付与されている場合を除き、データを書き込む可能性があるSQLステートメントを実行できません。

### tidb_retry_limit

- スコープ：SESSION | GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARなし
- タイプ：整数
- デフォルト値：`10`
- 範囲：`[-1, 9223372036854775807]`
- この変数は、楽観的なトランザクションの最大リトライ回数を設定するために使用されます。トランザクションがリトライ可能なエラー（トランザクションの競合、非常に遅いトランザクションのコミット、またはテーブルスキーマの変更など）に遭遇した場合、この変数に従ってトランザクションを再実行します。ただし、`tidb_retry_limit`を`0`に設定すると、自動リトライが無効になります。この変数は楽観的なトランザクションにのみ適用され、悲観的なトランザクションには適用されません。

### tidb_row_format_version

> **注記：**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ：SESSION | GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARなし
- タイプ：整数
- デフォルト値：`2`
- 範囲：`[1, 2]`
- この変数は、テーブルに新しく保存されるデータのフォーマットバージョンを制御します。TiDB v4.0では、[新しいストレージ行フォーマット](https://github.com/pingcap/tidb/blob/master/docs/design/2018-07-19-row-format.md)バージョン`2`がデフォルトで使用されます。
- TiDBのバージョンがv4.0.0以前からv4.0.0以降にアップグレードすると、フォーマットバージョンは変更されず、TiDBは引き続き古いバージョン`1`のフォーマットを使用してデータをテーブルに書き込むため、**デフォルトでは新しく作成されたクラスタのみが新しいデータフォーマットを使用**します。
- この変数を変更しても、すでに保存された古いデータには影響がありませんが、この変数を変更した後に書き込まれる新しいデータにのみ対応するバージョンのフォーマットが適用されます。

### tidb_runtime_filter_mode <span class="version-mark">v7.2.0で新規</span>

- スコープ：SESSION | GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARに適用あり
- タイプ：列挙型
- デフォルト値：`OFF`
- 可能な値：`OFF`、`LOCAL`
- ランタイムフィルタのモード、つまり**フィルタ送信オペレータ**と**フィルタ受信オペレータ**の関係を制御します。`OFF`と`LOCAL`の2つのモードがあります。`OFF`はランタイムフィルタを無効にすることを意味します。`LOCAL`はローカルモードでランタイムフィルタを有効にすることを意味します。詳細については、[ランタイムフィルタモード](/runtime-filter.md#runtime-filter-mode)を参照してください。

### tidb_runtime_filter_type <span class="version-mark">v7.2.0で新規</span>

- スコープ：SESSION | GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARに適用あり
- タイプ：列挙型
- デフォルト値：`IN`
- 可能な値：`IN`
- 生成されたフィルタオペレータが使用する述語のタイプを制御します。現在、`IN`のみのタイプがサポートされています。詳細は、[ランタイムフィルタタイプ](/runtime-filter.md#runtime-filter-type)を参照してください。

### tidb_scatter_region

> **注記：**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ：GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARなし
- タイプ：ブール値
- デフォルト値：`OFF`
- デフォルトでは、新しいテーブルをTiDBで作成する際には、そのテーブルが作成されている間にリージョンが分割されます。この変数を有効にした後、`CREATE TABLE`ステートメントの実行中に、新たに分割されたリージョンがすぐに散在されます。これは、新しく分割されたリージョンがPDによってスケジュールされるのを待つ必要がないため、テーブルが作成された直後にバッチでデータを書き込む場合に適用されます。バッチでデータを書き込む連続した安定性を確保するために、`CREATE TABLE`ステートメントは、リージョンが正常に散在された後にのみ成功を返します。これにより、ステートメントの実行時間がこの変数を無効にする場合よりも何倍も長くなります。
- テーブルの作成時に`SHARD_ROW_ID_BITS`と`PRE_SPLIT_REGIONS`が設定されている場合、指定された数のリージョンがテーブル作成後に均等に分割されます。

### tidb_schema_version_cache_limit <span class="version-mark">v7.4.0で新規</span>

- スコープ：GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARなし
- デフォルト値：`16`
- 範囲：`[2, 255]`
- この変数は、TiDBインスタンスにキャッシュされる履歴的なスキーマバージョンの数を制限します。デフォルト値は`16`で、TiDBはデフォルトで16のスキーマバージョンをキャッシュします。
- 通常、この変数を変更する必要はありません。[Stale Read](/stale-read.md)機能が使用され、DDL操作が非常に頻繁に実行されると、スキーマバージョンが非常に頻繁に変更されることがあります。そのため、Stale Readがスナップショットからスキーマ情報を取得しようとすると、スキーマキャッシュミスにより情報を再構築するのに多くの時間がかかる場合があります。この場合、`tidb_schema_version_cache_limit`の値を増やして（たとえば`32`）、スキーマキャッシュミスの問題を回避することができます。
- この変数を変更すると、TiDBのメモリ使用量がわずかに増加します。OOMの問題を避けるために、TiDBのメモリ使用量を監視してください。

### tidb_server_memory_limit <span class="version-mark">v6.4.0で新規</span>

> **注記：**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ：GLOBAL
- クラスタへの永続化：はい
- ヒントへの適用：SET_VARなし
- デフォルト値：`80%`
- 範囲：
    - パーセンテージ形式で値を設定できます。つまり、メモリ使用量のパーセンテージとしての値です。値の範囲は`[1%, 99%]`です。
    - メモリサイズで値を設定できます。値の範囲は、バイト単位で`0`および`[536870912, 9223372036854775807]`です。単位「KB|MB|GB|TB」を持つメモリ形式がサポートされています。`0`はメモリ制限無しを意味します。
    - この変数が512MB未満のメモリサイズに設定されている場合、TiDBは実際のサイズとして512MBを使用します。
- この変数はTiDBインスタンスのメモリ制限を指定します。TiDBのメモリ使用量が制限に達すると、TiDBは現在実行中のメモリ使用量が最も高いSQLステートメントをキャンセルします。SQLステートメントが正常にキャンセルされた後、TiDBはできるだけ早くメモリを回収するためにGolang GCを呼び出します。
- `tidb_server_memory_limit_sess_min_size`制限よりも多くのメモリを使用するSQLステートメントのみが、最初にキャンセルされるSQLステートメントとして選択されます。
- 現在、TiDBは一度に1つのSQLステートメントしかキャンセルしません。TiDBが完全にSQLステートメントをキャンセルし、リソースを回復した後でも、メモリ使用量がこの変数で設定された制限よりも依然高い場合、TiDBは次のキャンセル操作を開始します。

### tidb_server_memory_limit_gc_trigger <span class="version-mark">v6.4.0で新規</span>

> **注記：**
>
> この変数は、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では読み取り専用です。

- スコープ: GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- デフォルト値: `70%`
- 範囲: `[50%, 99%]`
- TiDBのメモリ使用量が `tidb_server_memory_limit` の値 \* `tidb_server_memory_limit_gc_trigger` の値に達したとき、TiDBは積極的にGolang GC操作をトリガしようとします。1分間に1回のみGC操作がトリガされます。

### tidb_server_memory_limit_sess_min_size <span class="version-mark">v6.4.0で新規</span>

> **注意:**
>
> この変数は、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では読み取り専用です。

- スコープ: GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- デフォルト値: `134217728` (つまり128 MB)
- 範囲: `[128, 9223372036854775807]`（バイト）。単位が「KB|MB|GB|TB」のメモリ形式もサポートされています。
- メモリ制限を有効にした後、TiDBは現在のインスタンスで最も大きなメモリ使用量のSQLステートメントを終了します。この変数は、終了するSQLステートメントの最小メモリ使用量を指定します。制限を超えるTiDBインスタンスのメモリ使用量が、低いメモリ使用量の多くのセッションによって引き起こされる場合、この変数の値を適切に低く設定して、より多くのセッションをキャンセルできます。

### tidb_service_scope <span class="version-mark">v7.4.0 で新規</span>

<CustomContent platform="tidb">

- スコープ: GLOBAL
- クラスタに維持: いいえ
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: String
- デフォルト値: ""
- オプション値: "", `background`
- この変数はインスタンスレベルのシステム変数です。これを使用して、[TiDB分散実行フレームワーク](/tidb-distributed-execution-framework.md)の下でのTiDBノードのサービス範囲を制御できます。`tidb_service_scope` を`background` に設定すると、TiDB分散実行フレームワークは、そのTiDBノードに[`ADD INDEX`](/sql-statements/sql-statement-add-index.md) や[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md) などのバックグラウンドタスクを実行するようスケジュールします。

> **注意:**
>
> - クラスタ内のいかなるTiDBノードに対しても`tidb_service_scope` が設定されていない場合、TiDB分散実行フレームワークはすべてのTiDBノードにバックグラウンドタスクの実行をスケジュールします。現在のビジネスにパフォーマンスへの影響が懸念される場合は、いくつかのTiDBノードに`tidb_service_scope` を`background` に設定できます。その場合、そのノードのみがバックグラウンドタスクを実行します。
> - 新たにスケールされたノードに対しては、デフォルトでTiDB分散実行フレームワークのタスクは実行されません。このスケールされたノードにバックグラウンドタスクの実行を許可する場合は、このノードの`tidb_service_scop` を`background` に手動で設定する必要があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- スコープ: GLOBAL
- クラスタに維持: いいえ
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: String
- デフォルト値: ""
- オプション値: "", `background`
- この変数はインスタンスレベルのシステム変数です。これを使用して、[TiDB分散実行フレームワーク](/tidb-distributed-execution-framework.md)の下でのTiDBノードのサービス範囲を制御できます。`tidb_service_scope` を`background` に設定すると、TiDB分散実行フレームワークは、そのTiDBノードに[`ADD INDEX`](/sql-statements/sql-statement-add-index.md) や[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md) などのバックグラウンドタスクを実行するようスケジュールします。

> **注意:**
>
> - クラスタ内のいかなるTiDBノードに対しても`tidb_service_scope` が設定されていない場合、TiDB分散実行フレームワークはすべてのTiDBノードにバックグラウンドタスクの実行をスケジュールします。現在のビジネスにパフォーマンスへの影響が懸念される場合は、いくつかのTiDBノードに`tidb_service_scope` を`background` に設定できます。その場合、そのノードのみがバックグラウンドタスクを実行します。
> - 新たにスケールされたノードに対しては、デフォルトでTiDB分散実行フレームワークのタスクは実行されません。このスケールされたノードにバックグラウンドタスクの実行を許可する場合は、このノードの`tidb_service_scop` を`background` に手動で設定する必要があります。

</CustomContent>

### tidb_session_alias <span class="version-mark">v7.4.0 で新規</span>

- スコープ: SESSION
- クラスタに維持: いいえ
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) はい
- デフォルト値: ""
- この変数を使用して、トラブルシューティングでセッションを識別するためのログに関連する`session_alias` 列の値をカスタマイズできます。この設定は、ステートメントの実行に関与する複数ノードのログ（TiKVを含む）に影響します。この変数の最大長は64文字までであり、その長さを超える文字は自動的に切り捨てられます。値の末尾の空白も自動的に削除されます。

### tidb_session_plan_cache_size <span class="version-mark">v7.1.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[1, 100000]`
- この変数はキャッシュできるプランの最大数を制御します。[プリペアドプランキャッシュ](/sql-prepared-plan-cache.md) と[プリペアドでないプランキャッシュ](/sql-non-prepared-plan-cache.md) は同じキャッシュを共有します。
- 以前のバージョンからv7.1.0以降にアップグレードすると、この変数は[`tidb_prepared_plan_cache_size`](#tidb_prepared_plan_cache_size-new-in-v610) と同じ値のままです。

### tidb_shard_allocate_step <span class="version-mark">v5.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: 整数
- デフォルト値: `9223372036854775807`
- 範囲: `[1, 9223372036854775807]`
- この変数は、[`AUTO_RANDOM`](/auto-random.md) や[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md) 属性のために割り当てられる連続したIDの最大数を制御します。通常、一つのトランザクション内で`AUTO_RANDOM` IDや注釈のついた行IDは増加して連続しています。この変数を使用して、大規模なトランザクションシナリオのホットスポット問題を解決できます。

### tidb_simplified_metrics

> **注意:**
>
> この変数は、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では読み取り専用です。

- スコープ: GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数が有効になると、Grafanaパネルで使用されていないメトリクをTiDBは収集または記録しません。

### tidb_skip_ascii_check <span class="version-mark">v5.0 で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数はASCII検証をスキップするかどうかを設定するために使用されます。
- ASCII文字の検証はパフォーマンスに影響を与えます。入力文字が有効なASCII文字であることが確実な場合は、変数の値を`ON` に設定できます。

### tidb_skip_isolation_level_check

- スコープ: SESSION | GLOBAL
- クラスタに維持: はい
- ヒントに適用: [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value) いいえ
- タイプ: ブール値
- デフォルト値: `OFF`
- このスイッチを有効にした後、TiDBでサポートされていない分離レベルが`tx_isolation` に割り当てられてもエラーが報告されません。これにより、異なる分離レベルを設定（依存するわけではない）アプリケーションとの互換性が向上します。

```sql
tidb> set tx_isolation='serializable';
ERROR 8048 (HY000): The isolation level 'serializable' is not supported. Set tidb_skip_isolation_level_check=1 to skip this error
tidb> set tidb_skip_isolation_level_check=1;
Query OK, 0 rows affected (0.00 sec)

tidb> set tx_isolation='serializable';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
### tidb_skip_missing_partition_stats <span class="version-mark">v7.3.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- [動的プルーニングモード](/partitioned-table.md#dynamic-pruning-mode)でパーティション分割されたテーブルにアクセスする場合、TiDBは各パーティションの統計を集計してGlobalStatsを生成します。この変数は、パーティションの統計が欠落している場合のGlobalStatsの生成を制御します。

    - この変数が `ON` の場合、TiDBはGlobalStatsの生成時に欠落しているパーティション統計をスキップし、GlobalStatsの生成に影響を与えません。
    - この変数が `OFF` の場合、TiDBは欠落しているパーティション統計を検出するとGlobalStatsの生成を停止します。

### tidb_skip_utf8_check

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数は、UTF-8の検証をスキップするかどうかを設定するために使用されます。
- UTF-8文字の検証はパフォーマンスに影響します。入力文字が有効なUTF-8文字であることを確実に知っている場合は、変数の値を `ON` に設定できます。

> **注意：**
>
> 文字の検査をスキップすると、TiDBはアプリケーションによって書き込まれた不正なUTF-8文字を検出できなくなり、`ANALYZE` が実行されたときにデコードエラーが発生したり、その他の不明なエンコーディングの問題が発生する可能性があります。アプリケーションが書き込まれた文字列の有効性を保証できない場合は、文字の検査をスキップすることは推奨されません。

### tidb_slow_log_threshold

> **注意：**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `300`
- 範囲: `[-1, 9223372036854775807]`
- 単位: ミリ秒
- この変数は、遅いログによって消費される時間の閾値値を出力するために使用されます。クエリの消費時間がこの値よりも大きい場合、このクエリは遅いログと見なされ、そのログは遅いクエリログに出力されます。

### tidb_slow_query_file

> **注意：**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- `INFORMATION_SCHEMA.SLOW_QUERY` がクエリされるとき、構成ファイルで `slow-query-file` によって設定された遅いクエリログ名が解析されます。デフォルトの遅いクエリログ名は "tidb-slow.log" です。他のログを解析するには、`tidb_slow_query_file` セッション変数を特定のファイルパスに設定してから、設定されたファイルパスを使用して `INFORMATION_SCHEMA.SLOW_QUERY` をクエリして遅いクエリログを解析できます。

<CustomContent platform="tidb">

詳細については、[遅いクエリの識別](/identify-slow-queries.md)を参照してください。

</CustomContent>

### tidb_snapshot

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- この変数は、セッションでデータを読み取る時点を設定するために使用されます。例として、変数を "2017-11-11 20:20:20" や "400036290571534337" のような TSO 番号に設定すると、現在のセッションはこの時点のデータを読み取ります。

### tidb_source_id <span class="version-mark">v6.5.0で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[1, 15]`

<CustomContent platform="tidb">

- この変数は、[双方向レプリケーション](/ticdc/ticdc-bidirectional-replication.md) クラスター内の異なるクラスターIDを設定するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[双方向レプリケーション](https://docs.pingcap.com/tidb/stable/ticdc-bidirectional-replication) クラスター内の異なるクラスターIDを設定するために使用されます。

</CustomContent>

### tidb_stats_cache_mem_quota <span class="version-mark">v6.1.0で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `0`（TiDBインスタンスの総メモリサイズの半分に自動設定される）
- 範囲: `[0, 1099511627776]`
- この変数は、TiDB統計キャッシュのメモリクォータを設定します。

### tidb_stats_load_pseudo_timeout <span class="version-mark">v5.4.0で新規追加</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `ON`
- この変数は、SQL最適化の待機時間がタイムアウトに達したときに完全な列統計を同期的にロードする動作をTiDBがどのように制御するかを管理します。デフォルト値 `ON` では、SQL最適化はタイムアウト後に擬似統計を使用するように戻ります。この変数を `OFF` にすると、タイムアウト後にSQL実行が失敗します。

### tidb_stats_load_sync_wait <span class="version-mark">v5.4.0で新規追加</span>

> **注意：**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み込み専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[0, 2147483647]`
- 単位: ミリ秒
- この変数は、統計情報を同期的にロードする機能を有効にするかどうかを制御します。値 `0` は機能が無効であることを意味します。機能を有効にするには、最大で同期的に完全な列統計をロードするためにSQL最適化が待機できるタイムアウト（ミリ秒単位）をこの変数に設定できます。詳細は、[統計情報のロード](/statistics.md#load-statistics)を参照してください。

### tidb_stmt_summary_enable_persistent <span class="version-mark">v6.6.0で新規追加</span>

<CustomContent platform="tidb-cloud">

> **注意：**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **警告：**
>
> ステートメントサマリーの永続化は実験的な機能です。本番環境で使用することはお勧めしません。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 真偽値
- デフォルト値: `OFF`
- この変数は読み取り専用です。[ステートメントサマリーの永続化](/statement-summary-tables.md#persist-statements-summary)を有効にするかどうかを制御します。

<CustomContent platform="tidb">

- この変数の値は、構成項目 [`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660) の値と同じです。

</CustomContent>

### tidb_stmt_summary_filename <span class="version-mark">v6.6.0で新規追加</span>

<CustomContent platform="tidb-cloud">

> **注意：**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **警告：**
>
> ステートメントサマリーの永続化は実験的な機能です。本番環境で使用することはお勧めしません。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 文字列
- デフォルト値: `"tidb-statements.log"`
- この変数は読み取り専用です。[ステートメントサマリーの永続化](/statement-summary-tables.md#persist-statements-summary)が有効になっている場合に永続データが書き込まれるファイルを指定します。

<CustomContent platform="tidb">

- この変数の値は、構成項目 [`tidb_stmt_summary_filename`](/tidb-configuration-file.md#tidb_stmt_summary_filename-new-in-v660) の値と同じです。

</CustomContent>
```markdown
<CustomContent platform="tidb-cloud">

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **警告:**
>
> ステートメントのサマリー永続化は実験的な機能です。これを本番環境で使用することは推奨されません。この機能は予告なく変更されるか削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `0`
- この変数は読み取り専用です。これは[ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary)が有効になっているときに永続化できるデータファイルの最大数を指定します。

<CustomContent platform="tidb">

- この変数の値は、設定項目[`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660)の値と同じです。

</CustomContent>

### tidb_stmt_summary_file_max_days <span class="version-mark">v6.6.0で新規</span>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **警告:**
>
> ステートメントのサマリー永続化は実験的な機能です。これを本番環境で使用することは推奨されません。この機能は予告なく変更されるか削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `3`
- 単位: 日
- この変数は読み取り専用です。これは[ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary)が有効になっているときに永続的データファイルを保持する最大日数を指定します。

<CustomContent platform="tidb">

- この変数の値は、設定項目[`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660)の値と同じです。

</CustomContent>

### tidb_stmt_summary_file_max_size <span class="version-mark">v6.6.0で新規</span>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **警告:**
>
> ステートメントのサマリー永続化は実験的な機能です。これを本番環境で使用することは推奨されません。この機能は予告なく変更されるか削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `64`
- 単位: MiB
- この変数は読み取り専用です。これは[ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary)が有効になっているときに永続的データファイルの最大サイズを指定します。

<CustomContent platform="tidb">

- この変数の値は、設定項目[`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660)の値と同じです。

</CustomContent>

### tidb_stmt_summary_history_size <span class="version-mark">v4.0で新規</span>

> **注:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `24`
- 範囲: `[0, 255]`
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)の履歴容量を設定するために使用されます。

### tidb_stmt_summary_internal_query <span class="version-mark">v4.0で新規</span>

> **注:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は[TiDBのSQL情報を](/statement-summary-tables.md)ステートメントのサマリーテーブルに含めるかどうかを制御するために使用されます。

### tidb_stmt_summary_max_sql_length <span class="version-mark">v4.0で新規</span>

> **注:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `4096`
- 範囲: `[0, 2147483647]`
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)内のSQL文字列の長さを制御するために使用されます。

### tidb_stmt_summary_max_stmt_count <span class="version-mark">v4.0で新規</span>

> **注:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `3000`
- 範囲: `[1, 32767]`
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)にメモリ内に保存するステートメントの最大数を設定するために使用されます。

### tidb_stmt_summary_refresh_interval <span class="version-mark">v4.0で新規</span>

> **注:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `1800`
- 範囲: `[1, 2147483647]`
- 単位: 秒
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)のリフレッシュ時間を設定するために使用されます。

### tidb_store_batch_size

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): はい
- タイプ: 整数
- デフォルト値: `4`
- 範囲: `[0, 25000]`
- この変数は`IndexLookUp`オペレータのコプロセッサータスクのバッチサイズを制御するために使用されます。 `0` はバッチを無効にします。タスクの数が比較的大きく、遅いクエリが発生する場合は、この変数を増やしてクエリの最適化を行うことができます。

### tidb_store_limit <span class="version-mark">v3.0.4およびv4.0で新規</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 9223372036854775807]`
- この変数はTiDBが同時にTiKVに送信できるリクエストの最大数を制限するために使用されます。 0 は制限がありません。

### tidb_streamagg_concurrency

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: 整数
- デフォルト値: `1`
- この変数はクエリの実行時に`StreamAgg`オペレータの並列処理数を設定します。
- この変数の値を変更することは**推奨されません**。変数の値を変更すると、データの正確性の問題が発生する可能性があります。

### tidb_super_read_only <span class="version-mark">v5.3.1で新規</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントへの適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): なし
- タイプ: ブール値
- デフォルト値: `OFF`
- `tidb_super_read_only`はMySQL変数`super_read_only`の代替として実装されることを目指しています。しかし、TiDBは分散データベースなので、`tidb_super_read_only`は実行後すぐにデータベースを読み取り専用にはしませんが、最終的にはします。
- `SUPER`または`SYSTEM_VARIABLES_ADMIN`権限を持つユーザーはこの変数を変更できます。

```
- この変数は、クラスタ全体の読み取り専用状態を制御します。変数が `ON` の場合、クラスタ全体のすべてのTiDBサーバーが読み取り専用モードになります。この場合、TiDBは `SELECT`、`USE`、`SHOW` など、データを変更しないステートメントのみを実行します。`INSERT` や `UPDATE` などの他のステートメントでは、TiDBは読み取り専用モードでこれらのステートメントの実行を拒否します。
- この変数を使用して読み取り専用モードを有効にすると、クラスタ全体が最終的に読み取り専用状態になります。TiDBクラスターでこの変数の値を変更したが、変更が他のTiDBサーバーにまだ伝播していない場合、未更新のTiDBサーバーは引き続き読み取り専用モードでは**ありません**。
- TiDBはSQLステートメントが実行される前に読み取り専用フラグをチェックします。v6.2.0以降、このフラグはSQLステートメントがコミットされる前にもチェックされます。これにより、サーバーが読み取り専用モードに設定された後も、長時間実行される[auto commit](/transaction-overview.md#autocommit)ステートメントによってデータが変更されるケースを防ぎます。
- この変数が有効になると、TiDBは未コミットのトランザクションを以下の方法で処理します：
    - 未コミットの読み取り専用トランザクションでは、通常通りトランザクションをコミットできます。
    - 読み取り専用でない未コミットのトランザクションでは、これらのトランザクションで書き込み操作を行うSQLステートメントは拒否されます。
    - 修正されたデータを持つ未コミットの読み取り専用トランザクションのコミットは拒否されます。
- 読み取り専用モードが有効になった後、すべてのユーザー（`SUPER` 権限を持つユーザーも）は、ユーザーに明示的に `RESTRICTED_REPLICA_WRITER_ADMIN` 権限が付与されるまで、データを書き込む可能性のあるSQLステートメントを実行できません。
- [`tidb_restricted_read_only`](#tidb_restricted_read_only-new-in-v520)システム変数が `ON` に設定されると、`tidb_restricted_read_only`が一部の場合には[`tidb_restricted_read_only`](#tidb_restricted_read_only-new-in-v520)に影響を受けます。詳細な影響については、[`tidb_restricted_read_only`](#tidb_restricted_read_only-new-in-v520)の説明を参照してください。

### tidb_sysdate_is_now <span class="version-mark">v6.0.0で新規</span>

- スコープ：SESSION | GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`OFF`
- この変数は、`SYSDATE` 関数を `NOW` 関数で置き換えることができるかどうかを制御するために使用されます。この構成項目は、MySQLオプション[`sysdate-is-now`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_sysdate-is-now)と同じ効果があります。

### tidb_sysproc_scan_concurrency <span class="version-mark">v6.5.0で新規</span>

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ：GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`1`
- 範囲：`[1, 256]`
- この変数は、TiDBが内部SQLステートメント（統計の自動更新など）を実行する際に行うスキャン操作の並行性を設定するために使用されます。

### tidb_table_cache_lease <span class="version-mark">v6.0.0で新規</span>

- スコープ：GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`3`
- 範囲：`[1, 10]`
- 単位：秒
- この変数は、デフォルト値が `3` の[cached tables](/cached-tables.md)のリース時間を制御するために使用されます。

### tidb_tmp_table_max_size <span class="version-mark">v5.3.0で新規</span>

- スコープ：SESSION | GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`67108864`
- 範囲：`[1048576, 137438953472]`
- 単位：バイト
- この変数は、単一の[一時テーブル](/temporary-tables.md)の最大サイズを設定するために使用されます。この変数値を超えるサイズの一時テーブルはエラーを引き起こします。

### tidb_top_sql_max_meta_count <span class="version-mark">v6.0.0で新規</span>

> **注記:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ：GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`5000`
- 範囲：`[1, 10000]`

<CustomContent platform="tidb">

- この変数は、[Top SQL](/dashboard/top-sql.md)で1分間に集計されるSQLステートメントタイプの最大数を制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[Top SQL](https://docs.pingcap.com/tidb/stable/top-sql)で1分間に集計されるSQLステートメントタイプの最大数を制御するために使用されます。

</CustomContent>

### tidb_top_sql_max_time_series_count <span class="version-mark">v6.0.0で新規</span>

> **注記:**
>
> このTiDB変数はTiDB Cloudには適用されません。

> **注記:**
>
> 現在、TiDB DashboardのTop SQLページでは、負荷に最も貢献するSQLクエリのトップ5種類しか表示されず、`tidb_top_sql_max_time_series_count`の設定とは無関係です。

- スコープ：GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：整数
- デフォルト値：`100`
- 範囲：`[1, 5000]`

<CustomContent platform="tidb">

- この変数は、[Top SQL](/dashboard/top-sql.md)で1分間に負荷に最も貢献するSQLステートメント（つまり、トップN）を記録できる数を制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[Top SQL](https://docs.pingcap.com/tidb/stable/top-sql)で1分間に負荷に最も貢献するSQLステートメント（つまり、トップN）を記録できる数を制御するために使用されます。

</CustomContent>

### tidb_track_aggregate_memory_usage

- スコープ：SESSION | GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：ブール値
- デフォルト値：`ON`
- この変数は、TiDBが集約関数のメモリ使用量を追跡するかどうかを制御します。

> **警告:**
>
> この変数を無効にすると、TiDBはメモリ使用量を正確に追跡できなくなり、対応するSQLステートメントのメモリ使用量を制御できなくなる可能性があります。

### tidb_tso_client_batch_max_wait_time <span class="version-mark">v5.3.0で新規</span>

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ：GLOBAL
- クラスターに永続化：はい
- ヒントに適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)：いいえ
- タイプ：浮動小数点数
- デフォルト値：`0`
- 範囲：`[0, 10]`
- 単位：ミリ秒
- この変数は、TiDBがPDからTSOを要求する際のバッチ操作の最大待機時間を設定するために使用されます。デフォルト値は `0` であり、追加の待機時間はありません。
- TiDBがPDから各TSO要求を取得するたびに、TiDBが使用するPDクライアントはできるだけ多くの同時に受信したTSO要求を収集します。その後、PDクライアントは収集された要求をバッチで1つのRPC要求にマージし、それをPDに送信します。これにより、PDに対するプレッシャーが軽減されます。
- この変数を`0`より大きな値に設定した後、TiDBは各バッチマージの終了まで、この値の最大期間を待ちます。これは、より多くのTSO要求を収集し、バッチ操作の効果を向上させるためです。
- この変数値を増やす場合のシナリオ：
    * TSO要求のプレッシャーが高く、PDリーダーのCPUがボトルネックに達し、TSOのRPC要求の遅延が発生している場合。
    * クラスターにTiDBインスタンスがあまりなく、すべてのTiDBインスタンスが高い同時実行度である場合。
- できるだけ小さい値に設定することをお勧めします。
> **注意:**
>
> PDリーダーのCPU使用率のボトルネック以外の理由により、TSO RPCレイテンシが増加する場合、この場合、`tidb_tso_client_batch_max_wait_time`の値を増やすと、TiDBの実行レイテンシが増加し、クラスタのQPSパフォーマンスに影響を与える可能性があります。

### tidb_ttl_delete_rate_limit <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `0`
- 範囲: `[0, 9223372036854775807]`
- この変数は各TiDBノードのTTLジョブでの`DELETE`ステートメントのレートを制限するために使用されます。この値は、TTLジョブで単一ノードの1秒あたりに許可される`DELETE`ステートメントの最大数を表します。この変数を`0`に設定すると、制限が適用されません。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_delete_batch_size <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `100`
- 範囲: `[1, 10240]`
- この変数はTTLジョブでの単一の`DELETE`トランザクションで削除できる最大行数を設定するために使用されます。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_delete_worker_count <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `4`
- 範囲: `[1, 256]`
- この変数は各TiDBノードでのTTLジョブの最大並行性を設定するために使用されます。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_job_enable <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `ON`
- タイプ: ブール値
- この変数はTTLジョブの有効化を制御するために使用されます。`OFF`に設定すると、TTL属性を持つすべてのテーブルが自動的に期限切れのデータのクリーンアップを停止します。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_scan_batch_size <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `500`
- 範囲: `[1, 10240]`
- この変数はTTLジョブで期限切れのデータをスキャンする際に使用される各`SELECT`ステートメントの`LIMIT`値を設定するために使用されます。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_scan_worker_count <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `4`
- 範囲: `[1, 256]`
- この変数は各TiDBノードでのTTLスキャンジョブの最大並行性を設定するために使用されます。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_job_schedule_window_start_time <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 時刻
- クラスタに永続化: はい
- デフォルト値: `00:00 +0000`
- この変数は、バックグラウンドでのTTLジョブのスケジュールウィンドウの開始時刻を制御するために使用されます。この変数の値を変更する際は慎重に行ってください。ウィンドウが小さいと、期限切れのデータのクリーンアップが失敗する可能性があります。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_job_schedule_window_end_time <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 時刻
- クラスタに永続化: はい
- デフォルト値: `23:59 +0000`
- この変数は、バックグラウンドでのTTLジョブのスケジュールウィンドウの終了時刻を制御するために使用されます。この変数の値を変更する際は慎重に行ってください。ウィンドウが小さいと、期限切れのデータのクリーンアップが失敗する可能性があります。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_ttl_running_tasks <span class="version-mark">v7.0.0で新規</span>

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)にとって読み取り専用です。

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `-1`、`[1, 256]`
- クラスタ全体での実行中のTTLタスクの最大数を指定します。`-1`は、TTLタスクの数がTiKVノードの数と同等であることを意味します。詳細については[Time to Live](/time-to-live.md)を参照してください。

### tidb_txn_assertion_level <span class="version-mark">v6.0.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 列挙型
- デフォルト値: `FAST`
- 可能な値: `OFF`、`FAST`、`STRICT`
- この変数はアサーションレベルを制御するために使用されます。アサーションは、データとインデックスの整合性を確認するもので、書き込まれるキーがトランザクションのコミットプロセスで存在するかどうかを確認します。詳細については[データとインデックスの不整合エラーのトラブルシューティング](/troubleshoot-data-inconsistency-errors.md)を参照してください。

    - `OFF`: このチェックを無効にします。
    - `FAST`: ほとんどパフォーマンスに影響を与えず、ほとんどのチェック項目を有効にします。
    - `STRICT`: 高負荷のシステムワークロードでは、悲観的トランザクションのパフォーマンスにわずかな影響を与えるため、すべてのチェック項目を有効にします。

- v6.0.0以降の新しいクラスタのデフォルト値は`FAST`です。v6.0.0より古いバージョンからアップグレードした既存のクラスタでは、デフォルト値は`OFF`です。

### tidb_txn_commit_batch_size <span class="version-mark">v6.2.0で新規</span>

- スコープ: GLOBAL
- クラスタに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `16384`
- 範囲: `[1, 1073741824]`
- 単位: バイト
- この変数は、TiDBがTiKVに送信するトランザクションのコミットリクエストのバッチサイズを制御するために使用されます。アプリケーションのワークロードのほとんどのトランザクションが大量の書き込み操作を持っている場合、この変数をより大きな値に調整すると、バッチ処理の性能を向上させることができます。ただし、この変数があまりにも大きな値に設定され、TiKVの[`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size)の制限を超えると、コミットは失敗する可能性があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、TiDBがTiKVに送信するトランザクションのコミットリクエストのバッチサイズを制御するために使用されます。アプリケーションのワークロードのほとんどのトランザクションが大量の書き込み操作を持っている場合、この変数をより大きな値に調整すると、バッチ処理の性能を向上させることができます。ただし、この変数があまりにも大きな値に設定され、TiKVの1つのログの最大サイズ（デフォルトは8 MB）を超えると、コミットは失敗する可能性があります。

</CustomContent>

### tidb_txn_mode

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 列挙
- デフォルト値: `pessimistic`
- 可能な値: `pessimistic`, `optimistic`
- この変数はトランザクションモードを設定するために使用されます。TiDB 3.0では、悲観的トランザクションがサポートされています。TiDB 3.0.8以降、[悲観的トランザクションモード](/pessimistic-transaction.md)はデフォルトで有効になっています。
- TiDBをv3.0.7またはそれ以前のバージョンからv3.0.8以降のバージョンにアップグレードする場合、デフォルトのトランザクションモードは変更されません。**デフォルトで悲観的トランザクションモードを使用するのは新たに作成されるクラスタのみです**。
- この変数が "optimistic"または ""に設定された場合、TiDBは[楽観的トランザクションモード](/optimistic-transaction.md)を使用します。

### tidb_use_plan_baselines <span class="version-mark">v4.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は実行計画バインディング機能を有効にするかどうかを制御するために使用されます。デフォルトでは有効になっており、`OFF`値を割り当てて無効にすることができます。実行計画バインディングの使用方法については、[実行計画バインディング](/sql-plan-management.md#create-a-binding)を参照してください。

### tidb_wait_split_region_finish

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- Regionの分散には通常長い時間がかかりますが、これはPDのスケジューリングとTiKVの負荷によって決まります。この変数は、`SPLIT REGION`ステートメントが実行されているときに、全てのRegionが完全に分散された後にクライアントに結果を返すかどうかを設定するために使用されます：
    - `ON`は、`SPLIT REGIONS`ステートメントが全てのRegionが分散されるまで待機することを要求します。
    - `OFF`は、`SPLIT REGIONS`ステートメントが全てのRegionの分散が終わる前に戻ることを許可します。
- Regionの分散時には、分散中のRegionの書き込みおよび読み取りのパフォーマンスに影響を及ぼす場合があります。バッチ書き込みまたはデータのインポートのシナリオでは、Regionの分散が終了した後にデータをインポートすることを推奨します。

### tidb_wait_split_region_timeout

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `300`
- 範囲: `[1, 2147483647]`
- 単位: 秒
- この変数は、`SPLIT REGION`ステートメントの実行タイムアウトを設定するために使用されます。指定された時間内にステートメントが完全に実行されない場合、タイムアウトエラーが返されます。

### tidb_window_concurrency <span class="version-mark">v4.0で新規</span>

> **警告:**
>
> v5.0から、この変数は非推奨です。これに代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、ウィンドウ演算子の並行度を設定するために使用されます。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tiflash_fastscan <span class="version-mark">v6.3.0で新規</span>

- スコープ: SESSION | GLOBAL
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- デフォルト値: `OFF`
- タイプ: ブール値
- [FastScan](/tiflash/use-fastscan.md)が有効になっている場合（`ON`に設定されている場合）、TiFlashはクエリのパフォーマンスを向上させますが、クエリ結果またはデータの一貫性を保証しません。

### tiflash_fine_grained_shuffle_batch_size <span class="version-mark">v6.2.0で新規</span>

- スコープ: SESSION | GLOBAL
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- デフォルト値: `8192`
- 範囲: `[1, 18446744073709551615]`
- Fine Grained Shuffleが有効になっている場合、TiFlashにプッシュダウンされたウィンドウ関数を並列で実行することができます。この変数は、送信者が送信するデータのバッチサイズを制御します。
- パフォーマンスへの影響: ビジネス要件に応じて適切なサイズを設定してください。適切でない設定はパフォーマンスに影響を与えます。値が小さすぎる場合（例: `1`）、ブロックごとに1回のネットワーク転送が発生します。値が大きすぎる場合（例: テーブルの総行数）、受信側は主にデータを待ち続ける時間が長くなり、パイプライン化された計算が機能しなくなります。適切な値を設定するには、TiFlash受信者が受け取る行数の分布を観察することができます。大部分のスレッドが例えば数百行または数百行しか受け取らない場合、この値を増やしてネットワークオーバーヘッドを減らすことができます。

### tiflash_fine_grained_shuffle_stream_count <span class="version-mark">v6.2.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: はい
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[-1, 1024]`
- ウィンドウ関数をTiFlashにプッシュダウンして実行する場合、この変数を使用してウィンドウ関数の実行の並行度を制御できます。次の値が可能です：

    * -1: Fine Grained Shuffle機能が無効になっています。TiFlashにプッシュダウンされたウィンドウ関数は単一のスレッドで実行されます。
    * 0: Fine Grained Shuffle機能が有効になっています。[`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610)が有効な値に設定されている場合（0より大きい値）、`tiflash_fine_grained_shuffle_stream_count`は[`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610)の値に設定されます。それ以外の場合、これは8に設定されます。 TiFlashノード上の物理スレッドの数）。TiFlash上でのウィンドウ関数の実際の並行度は：min（`tiflash_fine_grained_shuffle_stream_count`、TiFlashノード上の物理スレッドの数）。
    * 0より大きい整数：Fine Grained Shuffle機能が有効になっています。TiFlashにプッシュダウンされたウィンドウ関数は複数のスレッドで実行されます。並行度は：min（`tiflash_fine_grained_shuffle_stream_count`、TiFlashノード上の物理スレッドの数）。
- 理論的には、この値の増加とともにウィンドウ関数のパフォーマンスが線形的に向上します。ただし、この値が実際の物理スレッド数を超えると、パフォーマンスが低下する可能性があります。

### tiflash_mem_quota_query_per_node <span class="version-mark">v7.4.0で新規</span>

- スコープ: SESSION | GLOBAL
- クラスタに永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlashノード上のクエリの最大メモリ使用量を制限します。クエリのメモリ使用量がこの制限を超えると、TiFlashはエラーを返し、クエリを終了します。この変数を`-1`または`0`に設定すると、制限はありません。この変数を`0`より大きい値に設定し、[`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740) が有効な値に設定されている場合、TiFlashは[クエリレベルのスピリング](/tiflush/tiflash-spill-disk.md#query-level-spilling)を有効にします。

### tiflash_query_spill_ratio <span class="version-mark">v7.4.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスター全体に永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 浮動小数点
- デフォルト値: `0.7`
- 範囲: `[0, 0.85]`
- この変数はTiFlashの[クエリレベルのスピリング](/tiflash/tiflash-spill-disk.md#query-level-spilling)の閾値を制御します。`0`は自動クエリレベルのスピリングを無効にします。この変数が`0`より大きく、クエリのメモリ使用量が[`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740) * `tiflash_query_spill_ratio`を超えると、TiFlashはクエリレベルのスピリングを開始し、必要に応じてクエリ内のサポートされる演算子のデータをスピルします。

> **注意:**
>
> - この変数は、[`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740) が`0`または`-1`より大きい場合にのみ有効です。言い換えると、[tiflash_mem_quota_query_per_node](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740)が`0`または`-1`の場合でも、`tiflash_query_spill_ratio`が`0`より大きい場合でも、クエリレベルのスピリングは有効になりません。
> - TiFlashのクエリレベルのスピリングが有効になると、個々のTiFlash演算子のスピリングの閾値は自動的に無効になります。つまり、[`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740) と`tiflash_query_spill_ratio`の両方が0より大きい場合、3つの変数[tidb_max_bytes_before_tiflash_external_sort](/system-variables.md#tidb_max_bytes_before_tiflash_external_sort-new-in-v700)、[tidb_max_bytes_before_tiflash_external_group_by](/system-variables.md#tidb_max_bytes_before_tiflash_external_group_by-new-in-v700)、および[tidb_max_bytes_before_tiflash_external_join](/system-variables.md#tidb_max_bytes_before_tiflash_external_join-new-in-v700) は自動的に無効になり、`0`に設定されたのと同等になります。

### tiflash_replica_read <span class="version-mark">v7.3.0で新規追加</span>

> **注意:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: SESSION | GLOBAL
- クラスター全体に永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 列挙型
- デフォルト値: `all_replicas`
- 値のオプション: `all_replicas`、`closest_adaptive`、または`closest_replicas`
- この変数は、クエリがTiFlashエンジンを必要とする場合にTiFlashレプリカを選択するための戦略を設定します。
    - `all_replicas` は、解析計算に使用するすべての利用可能なTiFlashレプリカを意味します。
    - `closest_adaptive` は、クエリを開始するTiDBノードと同じゾーンのTiFlashレプリカを優先的に使用します。このゾーンのレプリカに必要なデータが含まれていない場合、クエリには他のゾーンのTiFlashレプリカとそれに対応するTiFlashノードが含まれます。
    - `closest_replicas` は、クエリを開始するTiDBノードと同じゾーンのTiFlashレプリカのみを使用します。このゾーンのレプリカに必要なデータが含まれていない場合、クエリはエラーを返します。

<CustomContent platform="tidb">

> **注意:**
>
> - TiDBノードには[ゾーン属性](/schedule-replicas-by-topology-labels.md#optional-configure-labels-for-tidb)が設定されておらず、`tiflash_replica_read`が`all_replicas`に設定されていない場合、TiFlashはレプリカ選択戦略を無視します。代わりに、クエリにすべてのTiFlashレプリカを使用し、`The variable tiflash_replica_read is ignored.`の警告を返します。
> - TiFlashノードには[ゾーン属性](/schedule-replicas-by-topology-labels.md#configure-labels-for-tikv-and-tiflash)が設定されていない場合、これらはどのゾーンにも属さないノードとして扱われます。

</CustomContent>

### tikv_client_read_timeout <span class="version-mark">v7.4.0で新規追加</span>

- スコープ: SESSION | GLOBAL
- クラスター全体に永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- 単位: ミリ秒
- `tikv_client_read_timeout`を使用して、クエリ内でTiDBがTiKV RPC読み取りリクエストを送信するタイムアウトを設定できます。TiDBクラスターがネットワークが不安定な環境にあるか、深刻なTiKV I/Oレイテンシの揺れがある場合、およびアプリケーションがSQLクエリのレイテンシに敏感な場合、`tikv_client_read_timeout`を設定して、TiKV RPC読み取りリクエストのタイムアウトを短縮できます。この場合、TiKVノードのI/Oレイテンシに揺れがある場合、TiDBはすぐにタイムアウトし、次のTiKVリージョンピアがあるTiKVノードに再度RPCリクエストを送信できます。すべてのTiKVリージョンピアのリクエストがタイムアウトした場合、TiDBは通常のタイムアウト（通常40秒）でリトライします。
- クエリ内でオプティマイザヒント`/*+ SET_VAR(TIKV_CLIENT_READ_TIMEOUT=N) */`を使用して、TiDBがTiKV RPC読み取りリクエストのタイムアウトを設定することもできます。オプティマイザヒントとこのシステム変数の両方が設定されている場合、オプティマイザヒントが優先されます。
- デフォルト値の`0`は、通常のタイムアウト（通常40秒）が使用されることを示します。

> **注意:**
>
> - 通常、通常のクエリは数ミリ秒かかりますが、TiKVノードが不安定なネットワークにあるかI/Oが揺れている場合は、クエリに1秒以上、10秒以上かかることがあります。この場合、オプティマイザヒント`/*+ SET_VAR(TIKV_CLIENT_READ_TIMEOUT=100) */`を使用して、特定のクエリのTiKV RPC読み取りリクエストタイムアウトを100ミリ秒に設定できます。このようにすると、TiKVノードの応答が遅くても、TiDBはすぐにタイムアウトし、次のTiKVリージョンピアがあるTiKVノードにRPCリクエストを再送信できます。2つのTiKVノードが同時にI/O揺れを起こす可能性は低いため、クエリは通常数ミリ秒から110ミリ秒以内に完了します。
> - `tikv_client_read_timeout`に対して小さすぎる値（たとえば1ミリ秒）を設定しないでください。そうすると、TiDBクラスターのワークロードが高い場合、リクエストが簡単にタイムアウトし、その後のリトライによりTiDBクラスターの負荷がさらに増加します。
> - 異なる種類のクエリに対して異なるタイムアウト値を設定する必要がある場合は、オプティマイザヒントを使用することをお勧めします。

### time_zone

- スコープ: SESSION | GLOBAL
- クラスター全体に永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- デフォルト値: `SYSTEM`
- この変数は現在のタイムゾーンを返します。値は`-8:00`などのオフセット、または`America/Los_Angeles`などの名前付きゾーンとして指定できます。
- 値`SYSTEM`は、タイムゾーンがシステムホストと同じであることを意味し、これは[`system_time_zone`](#system_time_zone)変数を介して利用できます。

### timestamp

- スコープ: SESSION
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 浮動小数点
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数の空でない値は、`CURRENT_TIMESTAMP()`や`NOW()`などの関数のタイムスタンプとして使用されるUNIXエポックを示します。この変数はデータの復元やレプリケーションで使用される場合があります。

### transaction_isolation

- スコープ: SESSION | GLOBAL
- クラスター全体に永続化: はい
- ヒント[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value)に適用: いいえ
- タイプ: 列挙型
- デフォルト値: `REPEATABLE-READ`
- 可能な値: `READ-UNCOMMITTED`、`READ-COMMITTED`、`REPEATABLE-READ`、`SERIALIZABLE`
- この変数はトランザクションの分離レベルを設定します。TiDBはMySQLとの互換性のために`REPEATABLE-READ`を宣伝していますが、実際の分離レベルはスナップショット分離です。詳細については、[トランザクションの分離レベル](/transaction-isolation-levels.md)を参照してください。

### tx_isolation

この変数は`transaction_isolation`のエイリアスです。

### tx_isolation_one_shot

> **注意:**
> この変数はTiDB内部で使用されます。使用することは想定されていません。

TiDBパーサーは、`SET TRANSACTION ISOLATION LEVEL [READ COMMITTED| REPEATABLE READ | ...]` ステートメントを `SET @@SESSION.TX_ISOLATION_ONE_SHOT = [READ COMMITTED| REPEATABLE READ | ...]` に変換します。

### tx_read_ts

- スコープ: SESSION
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: ""
- 古いリードシナリオで、このセッション変数はステーブルリードタイムスタンプの値を記録するのに使用されます。
- この変数はTiDBの内部操作に使用されます。この変数を設定することは**推奨されていません**。

### txn_scope

> **注意:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- スコープ: SESSION
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `global`
- 値のオプション: `global` および `local`
- この変数は、現在のセッションのトランザクションがグローバルトランザクションかローカルトランザクションかを設定するために使用されます。
- この変数はTiDBの内部操作に使用されます。この変数を設定することは**推奨されていません**。

### validate_password.check_user_name <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `ON`
- タイプ: ブール
- この変数はパスワード複雑性のチェックアイテムです。パスワードがユーザー名と一致するかどうかをチェックします。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650) が有効になっている場合のみ有効です。
- この変数が有効で`ON`に設定されている場合、パスワードを設定すると、TiDBはパスワードをユーザー名（ホスト名を除く）と比較します。パスワードがユーザー名と一致した場合は、パスワードが拒否されます。
- この変数は[`validate_password.policy`](#validate_passwordpolicy-new-in-v650)に独立しており、パスワード複雑度のチェックレベルに影響されません。

### validate_password.dictionary <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `""`
- タイプ: 文字列
- この変数はパスワード複雑性のチェックアイテムです。パスワードが辞書と一致するかどうかをチェックします。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650) が有効になっており、[`validate_password.policy`](#validate_passwordpolicy-new-in-v650) が `2` (STRONG) に設定されている場合にのみ有効です。
- この変数は1024文字より長くない文字列です。パスワードに存在してはならない単語のリストを含みます。各単語はセミコロン (`;`) で区切られています。
- この変数はデフォルトで空の文字列に設定されており、辞書チェックが実行されないことを意味します。辞書チェックを実行するには、一致させる単語を文字列に含める必要があります。この変数が構成されている場合、パスワードを設定すると、TiDBはパスワードの部分文字列 (長さが4～100文字) を辞書の単語と比較します。パスワードの部分文字列が辞書の単語と一致する場合、パスワードは拒否されます。比較は大文字と小文字を区別しません。

### validate_password.enable <span class="version-mark">v6.5.0で新規</span>

> **注意:**
>
> この変数はいつも[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して有効です。

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `OFF`
- タイプ: ブール
- この変数はパスワード複雑性のチェックを実行するかどうかを制御します。この変数が `ON` に設定されている場合、TiDBはパスワードを設定するときにパスワード複雑性のチェックを実行します。

### validate_password.length <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `8`
- 範囲: `0, 2147483647` (TiDB Self-Hosted および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) の場合)、`8, 2147483647` ([TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) の場合)
- この変数はパスワード複雑性のチェックアイテムです。パスワードの長さが十分かどうかをチェックします。デフォルトでは、最小のパスワードの長さは `8` です。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650) が有効になっている場合にのみ有効です。
- この変数の値は次の式より小さくてはいけません: `validate_password.number_count + validate_password.special_char_count + (2 * validate_password.mixed_case_count)`。
- `validate_password.number_count`、`validate_password.special_char_count`、または` validate_password.mixed_case_count`の値を変更して、式の値が`validate_password.length`より大きくなるようにした場合、`validate_password.length`の値は自動的に式の値と一致するように変更されます。

### validate_password.mixed_case_count <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `0, 2147483647` (TiDB Self-Hosted および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) の場合)、`1, 2147483647` ([TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) の場合)
- この変数はパスワード複雑性のチェックアイテムです。パスワードに十分な大文字と小文字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650) が有効になっており、 [`validate_password.policy`](#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) またはそれ以上に設定されている場合にのみ有効です。
- パスワードに含まれる大文字の数と小文字の数のどちらも、`validate_password.mixed_case_count`の値より少なくしてはいけません。例として、変数が `1` に設定されている場合、パスワードには少なくとも1つの大文字の文字と1つの小文字の文字とを含める必要があります。

### validate_password.number_count <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `0, 2147483647` (TiDB Self-Hosted および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) の場合)、`1, 2147483647` ([TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) の場合)
- この変数はパスワード複雑性のチェックアイテムです。パスワードに十分な数字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](#password_reuse_interval-new-in-v650) が有効になっている場合、 かつ [`validate_password.policy`](#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) またはそれ以上に設定されている場合にのみ有効です。

### validate_password.policy <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスタへの永続化: はい
- ヒントへの適用[SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 列挙型
- デフォルト値: `1`
- 値のオプション: `0`、`1` (TiDB Self-Hosted および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) の場合)、 `1` および `2` ([TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) の場合)
- この変数はパスワード複雑性のチェックのポリシーを制御します。この変数は、[`validate_password.enable`](#password_reuse_interval-new-in-v650) が有効になっている場合のみ有効です。この変数の値は、他の`validate-password`変数が、`validate_password.check_user_name`を除いて、パスワード複雑性のチェックでどのように影響を与えるかを決定します。
- この変数の値は`0`、`1`、または`2` (LOW、MEDIUM、STRONGに対応) になります。異なるポリシーレベルには異なるチェックがあります:
    - 0 または LOW: パスワードの長さ。
    - 1 または MEDIUM: パスワードの長さ、大文字と小文字、数字、特殊文字。
```markdown
    - 2またはSTRONG: パスワードの長さ、大文字と小文字の英字、数字、特殊文字、および辞書との一致。

### validate_password.special_char_count <span class="version-mark">v6.5.0で新規</span>

- スコープ: GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `1`
- 範囲: TiDB Self-Hosted および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) の場合 `[0, 2147483647]`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) の場合 `[1, 2147483647]`
- この変数はパスワードの複雑さをチェックするチェック項目です。パスワードに十分な特殊文字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](#password_reuse_interval-new-in-v650) が有効であり、[`validate_password.policy`](#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) またはそれ以上に設定されている場合にのみ効果があります。

### version

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `8.0.11-TiDB-`（tidb version）
- この変数は、MySQLのバージョンの後にTiDBのバージョンが続きます。例: '8.0.11-TiDB-v7.4.0'。

### version_comment

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: (string)
- この変数は、TiDBバージョンに関する追加の詳細を返します。たとえば、'TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible'。

### version_compile_machine

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: (string)
- この変数は、TiDBが実行されているCPUアーキテクチャの名前を返します。

### version_compile_os

- スコープ: NONE
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: (string)
- この変数は、TiDBが実行されているOSの名前を返します。

### wait_timeout

> **注記:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: 整数
- デフォルト値: `28800`
- 範囲: `[0, 31536000]`
- 単位: 秒
- この変数はユーザーセッションのアイドルタイムアウトを制御します。ゼロ値は無制限を意味します。

### warning_count

- スコープ: SESSION
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- デフォルト値: `0`
- この読み取り専用変数は、以前に実行されたステートメントで発生した警告の数を示します。

### windowing_use_high_precision

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- ヒントに適用 [SET_VAR](/optimizer-hints.md#set_varvar_namevar_value): いいえ
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、ウィンドウ関数を計算する際に高精度モードを使用するかどうかを制御します。
```