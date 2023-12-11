---
title: TiDB パスワード管理
summary: TiDB におけるユーザーパスワード管理のメカニズムを学びます。
---

# TiDB パスワード管理

ユーザーパスワードのセキュリティを保護するために、TiDBは v6.5.0 から以下のパスワード管理ポリシーをサポートしています。

- パスワード複雑性ポリシー: 空のパスワードや弱いパスワードを防ぐために、ユーザーに強力なパスワードの設定を要求します。
- パスワード有効期限ポリシー: ユーザーに定期的なパスワード変更を要求します。
- パスワード再利用ポリシー: 古いパスワードの再利用を防ぎます。
- 失敗したログインの追跡および一時的なアカウントロックポリシー: 誤ったパスワードによる複数回のログイン失敗後、同じユーザーがログインしようとするのを一時的に防止するため、ユーザーアカウントを一時的にロックします。

## TiDB 認証資格情報の格納

ユーザーアイデンティティの真正性を確保するために、TiDBはユーザーのログイン時に認証資格情報としてパスワードを使用します。

このドキュメントで説明されている「パスワード」は、TiDB が生成、格納、検証する内部資格情報を指します。TiDB はユーザーパスワードを `mysql.user` システムテーブルに格納します。

以下の認証プラグインは TiDB パスワード管理に関連しています。

- `mysql_native_password`
- `caching_sha2_password`
- `tidb_sm3_password`

TiDB 認証プラグインについての詳細は、[認証プラグインステータス](/security-compatibility-with-mysql.md#authentication-plugin-status)を参照してください。

## パスワード複雑性ポリシー

デフォルトでは TiDB ではパスワード複雑性チェックが無効になっています。パスワード複雑性に関連するシステム変数を構成することで、パスワード複雑性チェックを有効にし、ユーザーパスワードがポリシーに準拠するようにします。

パスワード複雑性ポリシーには次の特徴があります。

- ユーザーが平文でパスワードを設定する SQL ステートメント（`CREATE USER`、`ALTER USER`、`SET PASSWORD` を含む）に対して、TiDB はパスワードをパスワード複雑性ポリシーに対してチェックします。パスワードが要件を満たさない場合、パスワードは拒否されます。
- SQL 関数 [`VALIDATE_PASSWORD_STRENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_validate-password-strength) を使用して、パスワードの強度を検証できます。

> **注意:**
>
> - `CREATE USER` ステートメントにおいて、アカウントを作成時にはアカウントをロックできる場合でも、受け入れ可能なパスワードを設定する必要があります。そうしないと、アカウントがアンロックされた際に、このアカウントはパスワード複雑性ポリシーに準拠していないパスワードを使用して TiDB にログインできる可能性があります。
> - パスワード複雑性ポリシーの変更は既存のパスワードには影響せず、新たに設定されたパスワードのみに影響します。

以下の SQL ステートメントを実行して、パスワード複雑性ポリシーに関連するすべてのシステム変数を表示できます。

```sql
mysql> SHOW VARIABLES LIKE 'validate_password.%';

+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary         |        |
| validate_password.enable             | OFF    |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
8 rows in set (0.00 sec)
```

各システム変数の詳しい説明については、[システム変数](/system-variables.md#validate_passwordcheck_user_name-new-in-v650)を参照してください。

### パスワード複雑性ポリシーの構成

このセクションでは、パスワード複雑性ポリシーに関連するシステム変数の構成の例を示します。

パスワード複雑性チェックを有効にする:

```sql
SET GLOBAL validate_password.enable = ON;
```

ユーザーがユーザー名と同じパスワードを使用しないようにする:

```sql
SET GLOBAL validate_password.check_user_name = ON;
```

パスワード複雑性レベルを `LOW` に設定する:

```sql
SET GLOBAL validate_password.policy = LOW;
```

パスワードの最小長を `10` に設定する:

```sql
SET GLOBAL validate_password.length = 10;
```

パスワードに少なくとも2つの数字、1つの大文字、1つの小文字、1つの特殊文字が含まれるようにする:

```sql
SET GLOBAL validate_password.number_count = 2;
SET GLOBAL validate_password.mixed_case_count = 1;
SET GLOBAL validate_password.special_char_count = 1;
```

`mysql` や `abcd` のような単語を含むパスワードを防ぐ辞書チェックを有効にする:

```sql
SET GLOBAL validate_password.dictionary = 'mysql;abcd';
```

> **注意:**
>
> - `validate_password.dictionary` の値は、1024 文字以内の文字列です。パスワードに存在してはならない単語のリストを含みます。各単語はセミコロン (`;`) で区切られています。
> - 辞書チェックは大文字と小文字を区別しません。

### パスワード複雑性チェックの例

システム変数 `validate_password.enable` が `ON` に設定されている場合、TiDB はパスワード複雑性チェックを有効にします。次の例は、このチェック結果の例です。

TiDB はユーザーの平文パスワードをデフォルトのパスワード複雑性ポリシーに対してチェックします。設定したパスワードがポリシーを満たさない場合は、パスワードが拒否されます。

```sql
mysql> ALTER USER 'test'@'localhost' IDENTIFIED BY 'abc';
ERROR 1819 (HY000): Require Password Length: 8
```

TiDB はハッシュ化されたパスワードをパスワード複雑性ポリシーに対してチェックしません。

```sql
mysql> ALTER USER 'test'@'localhost' IDENTIFIED WITH mysql_native_password AS '*0D3CED9BEC10A777AEC23CCC353A8C08A633045E';
Query OK, 0 rows affected (0.01 sec)
```

最初にロックされたアカウントを作成する際は、パスワード複雑性ポリシーに準拠したパスワードを設定する必要があります。そうしないと、作成は失敗します。

```sql
mysql> CREATE USER 'user02'@'localhost' ACCOUNT LOCK;
ERROR 1819 (HY000): Require Password Length: 8
```

### パスワード強度検証関数

パスワードの強度をチェックするために、`VALIDATE_PASSWORD_STRENGTH()` 関数を使用できます。この関数はパスワードの引数を受け入れ、0（弱い）から100（強い）までの整数を返します。

> **注意:**
>
> この関数は、現在のパスワード複雑性ポリシーに基づいてパスワードの強度を評価します。パスワード複雑性ポリシーが変更されると、同じパスワードでも異なる評価結果が得られる可能性があります。

以下の例は、`VALIDATE_PASSWORD_STRENGTH()` 関数の使用例です。

```sql
mysql> SELECT VALIDATE_PASSWORD_STRENGTH('weak');
+------------------------------------+
| VALIDATE_PASSWORD_STRENGTH('weak') |
+------------------------------------+
|                                 25 |
+------------------------------------+
1 row in set (0.01 sec)

mysql> SELECT VALIDATE_PASSWORD_STRENGTH('lessweak$_@123');
+----------------------------------------------+
| VALIDATE_PASSWORD_STRENGTH('lessweak$_@123') |
+----------------------------------------------+
|                                           50 |
+----------------------------------------------+
1 row in set (0.01 sec)

mysql> SELECT VALIDATE_PASSWORD_STRENGTH('N0Tweak$_@123!');
+----------------------------------------------+
| VALIDATE_PASSWORD_STRENGTH('N0Tweak$_@123!') |
+----------------------------------------------+
|                                          100 |
+----------------------------------------------+
1 row in set (0.01 sec)
```

## パスワード有効期限ポリシー

TiDB は、パスワードセキュリティを向上させるために、パスワードを定期的に変更する必要があるように、パスワード有効期限ポリシーの構成をサポートしています。アカウントパスワードを手動で期限が切れるようにすることもできますし、自動パスワード有効期限ポリシーを確立することもできます。

自動パスワード有効期限ポリシーは、グローバルレベルとアカウントレベルで設定できます。データベース管理者は、グローバルレベルで自動パスワード有効期限ポリシーを確立することができる他、アカウントレベルのポリシーを使用してグローバルポリシーを上書きできます。

パスワード有効期限ポリシーの設定権限は次のとおりです。

- `SUPER` または `CREATE USER` 権限を持つデータベース管理者は、パスワードの手動期限切れを設定できます。
- `SUPER` または `CREATE USER` 権限を持つデータベース管理者は、アカウントレベルのパスワード有効期限ポリシーを設定できます。
- `SUPER` または `SYSTEM_VARIABLES_ADMINR` 権限を持つデータベース管理者は、グローバルレベルのパスワード有効期限ポリシーを設定できます。

### 手動期限切れ

アカウントパスワードを手動で期限切れにするには、`CREATE USER` ステートメントまたは `ALTER USER` ステートメントを使用します。

```sql
ALTER USER 'test'@'localhost' PASSWORD EXPIRE;
```

データベース管理者がアカウントパスワードを期限切れにすると、TiDB にログインする前にパスワードを変更する必要があります。手動期限切れは取り消すことはできません。

`CREATE ROLE` ステートメントを使用して作成されたロールの場合、ロールにはパスワードが必要ないため、ロールのパスワードフィールドは空です。その場合、TiDBは `password_expired` 属性を `'Y'` に設定します。これはロールのパスワードが手動的に期限切れになっていることを意味します。この設計の目的は、ロールがアンロックされ、空のパスワードで TiDB にログインされるのを防ぐことです。`ALTER USER ... ACCOUNT UNLOCK` ステートメントでロールをアンロックすると、このアカウントで有効なパスワードを設定する必要があります。

```sql
mysql> CREATE ROLE testrole;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT user,password_expired,Account_locked FROM mysql.user WHERE user = 'testrole';
+----------+------------------+----------------+
| user     | password_expired | Account_locked |
+----------+------------------+----------------+
| testrole | Y                | Y              |
+----------+------------------+----------------+
1 row in set (0.02 sec)
```

### 自動期限切れ

自動パスワード有効期限は、**パスワードの年齢** と **パスワードの有効期限** に基づいています。
- パスワードの有効期限: パスワードの最終変更日から現在日までの時間間隔。最終パスワード変更時刻は `mysql.user` システムテーブルに記録されます。
- パスワードのライフタイム: パスワードを用いて TiDB にログインできる日数。

許可されたライフタイムを超えてパスワードが使用されている場合、サーバーは自動的にそのパスワードを期限切れとみなします。

TiDB はグローバルレベルとアカウントレベルの自動パスワード期限切れをサポートしています。

- グローバルレベル

    システム変数 [`default_password_lifetime`](/system-variables.md#default_password_lifetime-new-in-v650) を設定することで、パスワードのライフタイムを制御できます。デフォルト値の `0` は、パスワードの期限切れがないことを示します。このシステム変数を正の整数 `N` に設定した場合、パスワードのライフタイムは `N` 日であり、`N` 日ごとにパスワードを変更する必要があります。

    グローバル自動パスワード期限切れポリシーは、アカウントレベルのオーバーライドを持たないすべてのアカウントに適用されます。

    次の例では、パスワードのライフタイムを 180 日に設定したグローバル自動パスワード期限切れポリシーを確立します:

    ```sql
    SET GLOBAL default_password_lifetime = 180;
    ```

- アカウントレベル

    個々のアカウントに自動パスワード期限切れポリシーを確立するには、`CREATE USER` や `ALTER USER` ステートメントで `PASSWORD EXPIRE` オプションを使用します。

    次の例では、ユーザーパスワードを 90 日ごとに変更する必要があります:

    ```sql
    CREATE USER 'test'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;
    ALTER USER 'test'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;
    ```

    次の例では、個別のアカウントに対する自動パスワード期限切れポリシーを無効にします:

    ```sql
    CREATE USER 'test'@'localhost' PASSWORD EXPIRE NEVER;
    ALTER USER 'test'@'localhost' PASSWORD EXPIRE NEVER;
    ```

    指定されたアカウントのアカウントレベルの自動パスワード期限切れポリシーを削除し、グローバル自動パスワード期限切れポリシーに従わせるためには:

    ```sql
    CREATE USER 'test'@'localhost' PASSWORD EXPIRE DEFAULT;
    ALTER USER 'test'@'localhost' PASSWORD EXPIRE DEFAULT;
    ```

### パスワード期限切れのチェックメカニズム

クライアントが TiDB サーバーに接続する際、サーバーはパスワードが期限切れかどうかを次の順序で確認します:

1. サーバーはパスワードが手動で期限切れに設定されているかどうかを確認します。
2. パスワードが手動で期限切れにされていない場合、サーバーはパスワードの有効期間が設定された期間よりも長いかどうかを確認します。もし長ければ、サーバーはそのパスワードを期限切れとみなします。

### 期限切れたパスワードの処理

TiDB サーバーのパスワード期限切れに対する挙動は制御できます。パスワードが期限切れの場合、サーバーはクライアントを切断するか、クライアントを "sandbox モード" に制限できます。"sandbox モード" では、TiDB サーバーは期限切れアカウントからの接続を許可しますが、その接続ではユーザーはパスワードのリセットのみが許可されます。

TiDB サーバーは、パスワードが期限切れの場合の挙動を制御できます。パスワードが期限切れの場合の TiDB サーバーの挙動を制御するには、TiDB 構成ファイルの [`security.disconnect-on-expired-password`](/tidb-configuration-file.md#disconnect-on-expired-password-new-in-v650) パラメーターを構成します。

```toml
[security]
disconnect-on-expired-password = true
```

- `disconnect-on-expired-password` が `true` に設定されている場合（デフォルト値）、サーバーはパスワードが期限切れの場合にクライアントを切断します。
- `disconnect-on-expired-password` が `false` に設定されている場合、サーバーは "sandbox モード" を有効にしてユーザーがサーバーに接続できるようにします。しかし、クライアントはパスワードのリセットのみが許可されます。パスワードがリセットされると、ユーザーは通常の SQL ステートメントを実行できます。

`disconnect-on-expired-password` が有効な場合、アカウントのパスワードが期限切れの場合、TiDB はそのアカウントからの接続を拒否します。そのような場合、次のようにパスワードを変更できます:

- 通常のアカウントのパスワードが期限切れの場合、管理者は SQL ステートメントを使用してそのアカウントのパスワードを変更することができます。
- 管理者アカウントのパスワードが期限切れの場合、別の管理者は SQL ステートメントを使用してそのアカウントのパスワードを変更することができます。
- 管理者アカウントのパスワードが期限切れであり、他に管理者がパスワードを変更するのを手助けできる者がいない場合、`skip-grant-table` メカニズムを使用してそのアカウントのパスワードを変更できます。詳細については、「パスワードを忘れた場合の処理」を参照してください [/user-account-management.md#forgot-the-root-password](/user-account-management.md#forgot-the-root-password)。

## パスワード再利用ポリシー

TiDB は以前のパスワードの再利用を制限することができます。パスワード再利用ポリシーは、パスワードの変更回数または経過時間に基づくことができ、また両方に基づくことができます。

パスワード再利用ポリシーはグローバルレベルおよびアカウントレベルで設定できます。グローバルレベルでパスワード再利用ポリシーを確立し、グローバルポリシーを上書きするためにアカウントレベルのポリシーを使用することができます。

TiDB はアカウントのパスワード履歴を記録し、新しいパスワードの選択を履歴から制限します:

- パスワード再利用ポリシーがパスワード変更回数に基づいている場合、新しいパスワードは直近の指定された回数のパスワードと同じであってはなりません。例えば、最小のパスワード変更回数を `3` に設定した場合、新しいパスワードは直前の 3 回のパスワードのいずれとも同じであってはなりません。
- パスワード再利用ポリシーが経過時間に基づいている場合、新しいパスワードは指定された日数内で使用されたパスワードのいずれとも同じであってはなりません。例えば、パスワード再利用間隔が `60` に設定されている場合、新しいパスワードは過去 60 日間で使用されたパスワードのいずれとも同じであってはなりません。

> **注記:**
>
> 空のパスワードはパスワード履歴に記録されず、いつでも再利用できます。

### グローバルレベルのパスワード再利用ポリシー

グローバルパスワード再利用ポリシーを確立するには、[`password_history`](/system-variables.md#password_history-new-in-v650) および [`password_reuse_interval`](/system-variables.md#password_reuse_interval-new-in-v650) システム変数を使用します。

例えば、最新の 6 回のパスワードと過去 365 日間に使用されたパスワードの再利用を禁止するグローバルパスワード再利用ポリシーを確立するには:

```sql
SET GLOBAL password_history = 6;
SET GLOBAL password_reuse_interval = 365;
```

グローバルパスワード再利用ポリシーは、アカウントレベルの上書きを持たないすべてのアカウントに適用されます。

### アカウントレベルのパスワード再利用ポリシー

アカウントレベルのパスワード再利用ポリシーを確立するには、`CREATE USER` または `ALTER USER` ステートメントで `PASSWORD HISTORY` および `PASSWORD REUSE INTERVAL` オプションを使用します。

例:

直近の 5 回のパスワードの再利用を禁止する場合:

```sql
CREATE USER 'test'@'localhost' PASSWORD HISTORY 5;
ALTER USER 'test'@'localhost' PASSWORD HISTORY 5;
```

過去の 365 日間に使用されたパスワードの再利用を禁止する場合:

```sql
CREATE USER 'test'@'localhost' PASSWORD REUSE INTERVAL 365 DAY;
ALTER USER 'test'@'localhost' PASSWORD REUSE INTERVAL 365 DAY;
```

両方の再利用ポリシーを組み合わせる場合、`PASSWORD HISTORY` および `PASSWORD REUSE INTERVAL` の両方を使用します:

```sql
CREATE USER 'test'@'localhost'
  PASSWORD HISTORY 5
  PASSWORD REUSE INTERVAL 365 DAY;
ALTER USER 'test'@'localhost'
  PASSWORD HISTORY 5
  PASSWORD REUSE INTERVAL 365 DAY;
```

アカウントレベルのパスワード再利用ポリシーを指定されたアカウントから削除し、グローバルパスワード再利用ポリシーに従うようにするには:

```sql
CREATE USER 'test'@'localhost'
  PASSWORD HISTORY DEFAULT
  PASSWORD REUSE INTERVAL DEFAULT;
ALTER USER 'test'@'localhost'
  PASSWORD HISTORY DEFAULT
  PASSWORD REUSE INTERVAL DEFAULT;
```

> **注記:**
>
> - パスワード再利用ポリシーを複数回設定すると、最後に設定された値が適用されます。
> - `PASSWORD HISTORY` および `PASSWORD REUSE INTERVAL` のデフォルト値は 0 で、再利用ポリシーが無効であることを示します。
> - ユーザー名を変更すると、TiDB は `mysql.password_history` システムテーブル内の対応するパスワード履歴を元のユーザー名から新しいユーザー名に移行します。

## ログイン失敗トラッキングと一時アカウントロックポリシー

TiDB はアカウントのログイン失敗試行回数を追跡することができます。パスワードが総当たり攻撃でクラックされるのを防ぐため、指定されたログイン失敗試行回数後にアカウントをロックすることができます。

> **注記:**
>
> - TiDB はアカウントレベルでのみ、ログイン失敗トラッキングや一時アカウントロックをサポートしており、グローバルレベルではサポートしていません。
> - ログイン失敗とは、クライアントが接続試行中に正しいパスワードを提供できず、未知のユーザーやネットワークの問題による接続失敗は含まれません。
> - アカウントのログインに失敗した場合、アカウントがログイン操作のパフォーマンスに影響を与える追加のチェックを受ける場合があり、特に高並列のログインシナリオでの影響があります。

### ログイン失敗追跡ポリシーを構成する

`CREATE USER` または `ALTER USER` ステートメントで `FAILED_LOGIN_ATTEMPTS` および `PASSWORD_LOCK_TIME` オプションを使用して、各アカウントのログイン失敗試行回数とロック時間を構成できます。使用可能な値オプションは次のとおりです:

- `FAILED_LOGIN_ATTEMPTS`: N. `N` 回の連続したログイン失敗後、アカウントが一時的にロックされます。N の値は 0 から 32767 の範囲です。
- `PASSWORD_LOCK_TIME`: N | UNBOUNDED.
    - Nは、連続した失敗したログイン試行後にアカウントが一時的に`N`日間ロックされることを意味します。Nの値は0から32767までの範囲です。
    - `UNBOUNDED`はロック時間が無制限であり、アカウントは手動でアンロックする必要があります。Nの値は0から32767までの範囲です。

> **注意:**
>
> - SQL文1つで`FAILED_LOGIN_ATTEMPTS`または`PASSWORD_LOCK_TIME`のいずれかを構成できます。この場合、アカウントのロックは有効になりません。
> - アカウントのロックは、`FAILED_LOGIN_ATTEMPTS`と`PASSWORD_LOCK_TIME`の両方が0でないときにのみ有効になります。

アカウントのロックポリシーは次のように設定できます:

新しいユーザを作成し、アカウントのロックポリシーを設定します。パスワードが3回連続で間違えられた場合、アカウントは3日間一時的にロックされます:

```sql
CREATE USER 'test1'@'localhost' IDENTIFIED BY 'password' FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 3;
```

既存のユーザのアカウントロックポリシーを変更します。パスワードが4回連続で間違えられた場合、アカウントは手動でアンロックされるまで無期限にロックされます:

```sql
ALTER USER 'test2'@'localhost' FAILED_LOGIN_ATTEMPTS 4 PASSWORD_LOCK_TIME UNBOUNDED;
```

既存のユーザのアカウントロックポリシーを無効にします:

```sql
ALTER USER 'test3'@'localhost' FAILED_LOGIN_ATTEMPTS 0 PASSWORD_LOCK_TIME 0;
```

### ロックされたアカウントのアンロック

次のシナリオで、連続したパスワードエラーのカウントをリセットできます:

- `ALTER USER ... ACCOUNT UNLOCK` 文を実行したとき。
- 正常にログインしたとき。

また、次のシナリオで、ロックされたアカウントをアンロックできます:

- ロック時間が終了すると、アカウントの自動ロックフラグは次回のログイン試行時にリセットされます。
- `ALTER USER ... ACCOUNT UNLOCK` 文を実行したとき。

> **注意:**
>
> アカウントが連続したログイン失敗のためにロックされた場合、アカウントロックポリシーを変更すると以下の影響があります:
>
> - `FAILED_LOGIN_ATTEMPTS`を変更した場合、アカウントのロック状態は変わりません。変更された`FAILED_LOGIN_ATTEMPTS`は、アカウントがアンロックされ、再度ログインを試行した後に有効になります。
> - `PASSWORD_LOCK_TIME`を変更した場合、アカウントのロック状態は変わりません。変更された`PASSWORD_LOCK_TIME`は、アカウントが再度ログインを試行したときに有効になります。その時、TiDBは新しいロック時間に到達しているかどうかをチェックします。到達していれば、TiDBはユーザをアンロックします。