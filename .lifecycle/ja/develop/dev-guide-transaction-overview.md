---
title: トランザクションの概要
summary: TiDBのトランザクションについての簡単な紹介
---

# トランザクションの概要

TiDBは完全な分散トランザクションをサポートし、[楽観的トランザクション](/optimistic-transaction.md)および[悲観的トランザクション](/pessimistic-transaction.md)（TiDB 3.0で導入）を提供しています。この記事では、主にトランザクションの文、楽観的トランザクションおよび悲観的トランザクション、トランザクションの分離レベル、楽観的トランザクションにおけるアプリケーション側のリトライとエラーハンドリングを紹介します。

## 一般的な文

このチャプターでは、TiDBでのトランザクションの使用方法について紹介します。次の例では、シンプルなトランザクションのプロセスを示しています：

BobはAliceに$20を送金したいと考えています。このトランザクションには2つの操作が含まれます:

- Bobの口座から$20を引き落とす。
- Aliceの口座に$20を入金する。

トランザクションは、上記の両方の操作が正常に実行されるか、どちらも失敗するかを保証できます。

[bookshop](/develop/dev-guide-bookshop-schema-design.md)データベースの`users`テーブルにサンプルデータを挿入します：

```sql
INSERT INTO users (id, nickname, balance)
  VALUES (2, 'Bob', 200);
INSERT INTO users (id, nickname, balance)
  VALUES (1, 'Alice', 100);
```

次のトランザクションを実行し、各文が何を意味するかを説明します：

```sql
BEGIN;
  UPDATE users SET balance = balance - 20 WHERE nickname = 'Bob';
  UPDATE users SET balance = balance + 20 WHERE nickname= 'Alice';
COMMIT;
```

上記のトランザクションが正常に実行された後、テーブルは次のように見えるはずです：

```
+----+--------------+---------+
| id | account_name | balance |
+----+--------------+---------+
|  1 | Alice        |  120.00 |
|  2 | Bob          |  180.00 |
+----+--------------+---------+
```

### トランザクションを開始

新しいトランザクションを明示的に開始するには、`BEGIN`または`START TRANSACTION`を使用できます。

```sql
BEGIN;
```

```sql
START TRANSACTION;
```

TiDBのデフォルトのトランザクションモードは悲観的です。また、[楽観的トランザクションモデル]
(/develop/dev-guide-optimistic-and-pessimistic-transaction.md)を明示的に指定することもできます。

```sql
BEGIN OPTIMISTIC;
```

[悲観的トランザクションモード](/develop/dev-guide-optimistic-and-pessimistic-transaction.md)を有効にします。

```sql
BEGIN PESSIMISTIC;
```

上記の文が実行される際に現在のセッションがトランザクションの途中にある場合、TiDBはまず現在のトランザクションをコミットし、その後新しいトランザクションを開始します。

### トランザクションのコミット

`COMMIT`文を使用して、TiDBが現在のトランザクションで行ったすべての変更をコミットできます。

```sql
COMMIT;
```

楽観的トランザクションを有効にする前に、`COMMIT`文で返される可能性のあるエラーをアプリケーションが適切に処理できることを確認してください。アプリケーションの処理方法がわからない場合は、代わりに悲観的トランザクションモードを使用することをお勧めします。

### トランザクションのロールバック

`ROLLBACK`文を使用して、現在のトランザクションの変更を元に戻すことができます。

```sql
ROLLBACK;
```

前述の送金の例では、トランザクション全体をロールバックすると、AliceとBobの残高は変わらず、現在のトランザクションのすべての変更がキャンセルされます。

```sql
TRUNCATE TABLE `users`;

INSERT INTO `users` (`id`, `nickname`, `balance`) VALUES (1, 'Alice', 100), (2, 'Bob', 200);

SELECT * FROM `users`;
+----+--------------+---------+
| id | nickname     | balance |
+----+--------------+---------+
|  1 | Alice        |  100.00 |
|  2 | Bob          |  200.00 |
+----+--------------+---------+

BEGIN;
  UPDATE `users` SET `balance` = `balance` - 20 WHERE `nickname`='Bob';
  UPDATE `users` SET `balance` = `balance` + 20 WHERE `nickname`='Alice';
ROLLBACK;

SELECT * FROM `users`;
+----+--------------+---------+
| id | nickname     | balance |
+----+--------------+---------+
|  1 | Alice        |  100.00 |
|  2 | Bob          |  200.00 |
+----+--------------+---------+
```

トランザクションは、クライアント接続が停止または閉じられた場合にも自動的にロールバックされます。

## トランザクション分離レベル

トランザクション分離レベルは、データベーストランザクション処理の基盤です。 **ACID**の「I」（分離）は、トランザクションの分離を指します。

SQL-92標準では、以下の4つの分離レベルが定義されています:

- 読み取り未確定（`READ UNCOMMITTED`）
- 読み取り確定（`READ COMMITTED`）
- 可変読み取り（`REPEATABLE READ`）
- 直列化可能（`SERIALIZABLE`）。

詳細については次の表を参照してください：

| 分離レベル        | ダーティライト | ダーティリード | ファジーリード | ファントム     |
| ------------------| ------------ | ------------ | ------------ | ------------ |
| 読み取り未確定   | 不可能       | 可能         | 可能         | 可能         |
| 読み取り確定     | 不可能       | 不可能      | 可能         | 可能         |
| 可変読み取り     | 不可能       | 不可能      | 不可能       | 可能         |
| 直列化可能       | 不可能       | 不可能      | 不可能       | 不可能       |

TiDBは、次の分離レベルをサポートしています: `READ COMMITTED`および`REPEATABLE READ`:

```sql
mysql> SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
ERROR 8048 (HY000): The isolation level 'READ-UNCOMMITTED' is not supported. Set tidb_skip_isolation_level_check=1 to skip this error
mysql> SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
Query OK, 0 rows affected (0.00 sec)

mysql> SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
ERROR 8048 (HY000): The isolation level 'SERIALIZABLE' is not supported. Set tidb_skip_isolation_level_check=1 to skip this error
```

TiDBは、MySQLとの整合性のために、スナップショット分離（SI）レベルの整合性、別名"repeatable read"を実装しています。この分離レベルは、[ANSI Repeatable Read分離レベル](/transaction-isolation-levels.md#difference-between-tidb-and-ansi-repeatable-read)および[MySQL Repeatable Read分離レベル](/transaction-isolation-levels.md#difference-between-tidb-and-mysql-repeatable-read)とは異なります。詳細については、[TiDBトランザクション分離レベル](/transaction-isolation-levels.md)を参照してください。