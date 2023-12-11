---
title: SET [GLOBAL|SESSION] <variable> | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSET [GLOBAL|SESSION] <variable>の使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-set-variable/','/docs/dev/reference/sql/statements/set-variable/']
---

# `SET [GLOBAL|SESSION] <variable>`

`SET [GLOBAL|SESSION]` ステートメントは、`SESSION`または`GLOBAL`スコープのTiDBの組み込み変数の1つを変更します。

> **注意:**
>
> MySQLと同様に、`GLOBAL`変数への変更は既存の接続またはローカル接続には適用されません。新しいセッションのみが値の変更を反映します。

## 概要

**SetStmt:**

![SetStmt](/media/sqlgram/SetStmt.png)

**VariableAssignment:**

![VariableAssignment](/media/sqlgram/VariableAssignment.png)

## 例

`sql_mode`の値を取得します。

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'sql_mode';
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| Variable_name | Value                                                                                                                                     |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| sql_mode      | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW SESSION VARIABLES LIKE 'sql_mode';
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| Variable_name | Value                                                                                                                                     |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| sql_mode      | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

`sql_mode`の値をグローバルに更新します。更新後に`SQL_mode`の値を確認すると、`SESSION`レベルの値が更新されていないことがわかります:

```sql
mysql> SET GLOBAL sql_mode = 'STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER';
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'sql_mode';
+---------------+-----------------------------------------+
| Variable_name | Value                                   |
+---------------+-----------------------------------------+
| sql_mode      | STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER |
+---------------+-----------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW SESSION VARIABLES LIKE 'sql_mode';
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| Variable_name | Value                                                                                                                                     |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| sql_mode      | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

`SET SESSION`を使用すると、すぐに効果があります:

```sql
mysql> SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER';
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW SESSION VARIABLES LIKE 'sql_mode';
+---------------+-----------------------------------------+
| Variable_name | Value                                   |
+---------------+-----------------------------------------+
| sql_mode      | STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER |
+---------------+-----------------------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

次の動作上の違いが適用されます:

* `SET GLOBAL`で行われた変更は、クラスタ内のすべてのTiDBインスタンスに伝播します。これはMySQLとは異なり、変更はレプリカに伝播しません。
* TiDBはいくつかの変数を読み取り可能および設定可能として表示します。これはMySQLの互換性のために必要であり、アプリケーションやコネクタがMySQL変数を読み取ることが一般的です。例: JDBCコネクタはクエリキャッシュ設定を読み取り可能であり設定可能であり、その動作には依存していません。
* `SET GLOBAL`で行われた変更は、TiDBサーバの再起動を介して永続化されます。つまり、TiDBの`SET GLOBAL`は、MySQL 8.0以降で利用可能な`SET PERSIST`によりさらに似た動作をします。

## 関連情報

* [SHOW \[GLOBAL|SESSION\] VARIABLES](/sql-statements/sql-statement-show-variables.md)