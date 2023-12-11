---
title: TiDBを使用したJavaアプリケーションの開発のベストプラクティス
summary: TiDBを使用したJavaアプリケーションの開発のベストプラクティスを学びます。
aliases: ['/docs/dev/best-practices/java-app-best-practices/','/docs/dev/reference/best-practices/java-app/']
---

# TiDBを使用したJavaアプリケーションの開発のベストプラクティス

このドキュメントでは、Javaアプリケーションを開発するためのベストプラクティスを紹介します。一般的なJavaアプリケーションのコンポーネントとTiDBデータベースとのやり取りに基づいて、開発中によく遭遇する問題の解決策も提供します。

## Javaアプリケーションにおけるデータベース関連のコンポーネント

JavaアプリケーションにおけるTiDBデータベースとのやり取りに関連する一般的なコンポーネントには、以下があります。

- ネットワークプロトコル: クライアントは標準の[MySQLプロトコル](https://dev.mysql.com/doc/dev/mysql-server/latest/PAGE_PROTOCOL.html)経由でTiDBサーバとやり取りします。
- JDBC APIとJDBCドライバ: Javaアプリケーションでは通常、標準の[JDBC（Java Database Connectivity）](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/) APIを使用してデータベースにアクセスします。TiDBに接続するには、JDBC APIを介してMySQLプロトコルを実装したJDBCドライバを使用できます。MySQLの一般的なJDBCドライバには、[MySQL Connector/J](https://github.com/mysql/mysql-connector-j)や[MariaDB Connector/J](https://mariadb.com/kb/en/library/about-mariadb-connector-j/#about-mariadb-connectorj)などがあります。
- データベースコネクションプール: 接続を再利用してキャッシュするため、アプリケーションは通常、コネクションプールを使用します。JDBCの[DataSource](https://docs.oracle.com/javase/8/docs/api/javax/sql/DataSource.html)はコネクションプールのAPIを定義しています。必要に応じて異なるオープンソースのコネクションプールの実装を選択できます。
- データアクセスフレームワーク: アプリケーションでは、[MyBatis](https://mybatis.org/mybatis-3/index.html)や[Hibernate](https://hibernate.org/)などのデータアクセスフレームワークを使用して、データベースアクセス操作をさらに簡略化して管理することが一般的です。
- アプリケーション実装: アプリケーションロジックは、どのタイミングでデータベースにどのようなコマンドを送信するかを制御します。一部のアプリケーションでは、[Spring Transaction](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html)を使用してトランザクションの開始とコミットロジックを管理します。 

![Javaアプリケーションのコンポーネント](/media/best-practices/java-practice-1.png)

上記の図からわかるように、Javaアプリケーションは次のことを行う可能性があります。

- JDBC APIを使用してTiDBとやり取りするためのMySQLプロトコルを実装します。
- 接続プールから永続的な接続を取得します。
- MyBatisなどのデータアクセスフレームワークを使用してSQLステートメントを生成し実行します。
- Spring Transactionを使用してトランザクションの自動開始または停止を実行します。

このドキュメントの残りの部分では、上記のコンポーネントを使用してJavaアプリケーションを開発する際に遭遇する問題とその解決策について説明します。

## JDBC

Javaアプリケーションはさまざまなフレームワークでカプセル化することができます。ほとんどのフレームワークでは、JDBC APIが最下層で呼び出されてデータベースサーバとやり取りします。JDBCでは、以下の点に焦点を当てることをお勧めします。

- JDBC APIの使用方法の選択
- API実装者のパラメーター設定

### JDBC API

JDBC APIの使用方法については、[JDBC公式チュートリアル](https://docs.oracle.com/javase/tutorial/jdbc/)を参照してください。このセクションでは、いくつかの重要なAPIの使用方法について説明します。

#### Prepare APIの使用

OLTP（オンライントランザクション処理）シナリオでは、プログラムがデータベースに送信するSQLステートメントは、パラメーターの変更を取り除いた後で使い尽くされるタイプのものであるため、通常、[プリペアドステートメント](https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html)を使用し、テキストファイルからの通常の実行ではなく、プリペアドステートメントを再利用して直接実行することをお勧めします。これにより、TiDBで繰り返しSQLの解析と実行計画の生成のオーバーヘッドを回避できます。

現在、ほとんどの上位フレームワークは、SQLの実行にPrepare APIを呼び出します。開発時にJDBC APIを直接使用する場合は、Prepare APIを選択することに注意してください。

また、MySQL Connector/Jのデフォルト実装では、クライアント側のステートメントのみが事前処理され、ステートメントは`?`がクライアントで置換された後、テキストファイルでサーバに送信されます。したがって、Prepare APIを使用するだけでなく、ステートメントの事前処理を行う前に、JDBC接続パラメータで`useServerPrepStmts = true`を設定する必要があります。詳細なパラメータ構成については、[MySQL JDBCパラメータ](#mysql-jdbc-parameters)を参照してください。

#### Batch APIの使用

バッチ挿入では、[`addBatch`/`executeBatch` API](https://www.tutorialspoint.com/jdbc/jdbc-batch-processing)を使用できます。`addBatch()`メソッドは、まず複数のSQLステートメントをクライアントでキャッシュし、その後、`executeBatch`メソッドを呼び出すときにそれらをまとめてデータベースサーバに送信します。

> **注意:**
>
> デフォルトのMySQL Connector/J実装では、`addBatch()`でバッチ処理に追加されたSQLステートメントの送信は`executeBatch()`が呼び出されるまで遅延されますが、実際のネットワーク転送中もステートメントは実際には1つずつ送信されます。そのため、この方法では通常、通信のオーバーヘッドを削減することができません。
>
> ネットワーク転送をバッチ処理したい場合は、JDBC接続パラメータで`rewriteBatchedStatements = true`を設定する必要があります。詳細なパラメータ構成については、[バッチ関連のパラメータ](#batch-related-parameters)を参照してください。

#### 実行結果を取得するために`StreamingResult`を使用

ほとんどのシナリオでは、実行効率を向上させるために、JDBCはデフォルトでクエリ結果を事前に取得し、クライアントメモリに保存します。ただし、クエリが非常に大きな結果セットを返すと、クライアントはデータベースサーバに対して一度に返されるレコードの数を減らし、クライアントのメモリが準備完了し、次のバッチをリクエストするまで待機したい場合があります。

通常、JDBCには2種類の処理方法があります。

- `FetchSize`を`Integer.MIN_VALUE`に設定して、クライアントがキャッシュしないようにします。クライアントは`StreamingResult`を介してネットワーク接続から実行結果を読み込みます。

    クライアントがストリーミング読み込み方式を使用する場合、この後、クエリを続ける前に`resultset`の読み取りを完了するか`resultset`を閉じる必要があります。それ以外の場合、エラー`No statements may be issued when any streaming result sets are open and in use on a given connection. Ensure that you have called .close() on any active streaming result sets before attempting more queries.`が返されます。

    クライアントが`resultset`の読み取りを完了するかまたは`resultset`を閉じる前にクエリを続けることを避けるために、URLに`clobberStreamingResults=true`パラメータを追加できます。その後、`resultset`は自動的に閉じられますが、前のストリーミングクエリで読み取る結果セットは失われます。

- カーソルフェッチを使用するには、まず`FetchSize`を正の整数に設定し、JDBC URLで`useCursorFetch=true`を設定します。

TiDBは両方の方法をサポートしていますが、より単純な実装で実行効率が向上するため、第1の方法を使用することをお勧めします。

### MySQL JDBCパラメータ

JDBCは通常、JDBC URLパラメータの形で実装に関連する構成を提供します。このセクションでは、[MySQL Connector/Jのパラメータ構成](https://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html)（MariaDBを使用する場合は[MariaDBのパラメータ構成](https://mariadb.com/kb/en/library/about-mariadb-connector-j/#optional-url-parameters)を参照してください）を紹介します。このドキュメントではすべての構成項目を網羅することができないため、パフォーマンスに影響を与える可能性のあるいくつかのパラメータに焦点を当てて説明します。

#### Prepare関連のパラメータ

このセクションでは、`Prepare`に関連するパラメータについて紹介します。

##### `useServerPrepStmts`

`useServerPrepStmts`はデフォルトで`false`に設定されており、つまりPrepare APIを使用しても、"prepare"操作はクライアント側でのみ行われます。サーバの解析オーバーヘッドを回避するために、同じSQLステートメントを複数回Prepare APIを使用する場合は、この構成を`true`に設定することをお勧めします。

この設定がすでに有効になっているかどうかを確認するには、次のように行います。

- TiDBのモニタリングダッシュボードに移動し、**Query Summary** > **CPS By Instance**でリクエストコマンドタイプを表示します。
- リクエスト内の`COM_QUERY`が`COM_STMT_EXECUTE`または`COM_STMT_PREPARE`に置き換わっている場合、この設定がすでに有効になっています。

##### `cachePrepStmts`

`useServerPrepStmts=true`を設定しても、デフォルトではクライアントは各実行後にPrepareステートメントを閉じ、再利用しません。つまり、"prepare"操作はテキストファイルの実行よりも効率的ではありません。この問題を解決するために、`useServerPrepStmts=true`を設定した後に、`cachePrepStmts=true`を構成することをお勧めします。これにより、クライアントはPrepareステートメントをキャッシュできます。

この設定がすでに有効になっているかどうかを確認するには、次のように行います。

- TiDBのモニタリングダッシュボードに移動し、**Query Summary** > **CPS By Instance**でリクエストコマンドタイプを表示します。
- リクエスト内の`COM_STMT_EXECUTE`の数が`COM_STMT_PREPARE`の数よりもはるかに多い場合、この設定がすでに有効になっています。

また、`useConfigs=maxPerformance`を構成すると、`cachePrepStmts=true`を含む複数のパラメータが同時に構成されます。

##### `prepStmtCacheSqlLimit`

`cachePrepStmts`を設定した後は、`prepStmtCacheSqlLimit`の設定にも注意する必要があります（デフォルト値は`256`です）。この設定は、クライアントでキャッシュされるPrepared Statementsの最大長を制御します。

この最大長を超えるPrepared Statementsはキャッシュされないため、再利用することはできません。この場合、アプリケーションの実際のSQLの長さに応じて、この設定の値を増やすことを検討する必要があります。

以下の点についてこの設定が小さすぎるかどうかを確認する必要があります:
- TiDB監視ダッシュボードにアクセスし、**Query Summary** > **CPS By Instance**を通じてリクエストのコマンドタイプを表示する。
- `cachePrepStmts=true`が設定されているが、`COM_STMT_PREPARE`がほとんど`COM_STMT_EXECUTE`に等しく、`COM_STMT_CLOSE`が存在することを確認する。

##### `prepStmtCacheSize`

`prepStmtCacheSize`は、キャッシュされるPrepared Statementsの数を制御します（デフォルト値は`25`です）。アプリケーションが多くの種類のSQL文を"準備"し、Prepared Statementsを再利用したい場合は、この値を増やすことができます。

この設定が既に有効になっているかどうかを確認するためには、以下の作業を行うことができます:
- TiDB監視ダッシュボードにアクセスし、**Query Summary** > **CPS By Instance**を通じてリクエストのコマンドタイプを表示する。
- リクエスト内の`COM_STMT_EXECUTE`の数が`COM_STMT_PREPARE`の数よりもはるかに多い場合、この設定が既に有効になっていることを意味します。

#### バッチ関連のパラメータ

バッチ書き込みを処理する際には、`rewriteBatchedStatements=true`を設定することをお勧めします。`addBatch()`または`executeBatch()`を使用した後も、JDBCはデフォルトでSQLを個別に送信します。例えば:

```java
pstmt = prepare("insert into t (a) values(?)");
pstmt.setInt(1, 10);
pstmt.addBatch();
pstmt.setInt(1, 11);
pstmt.addBatch();
pstmt.setInt(1, 12);
pstmt.executeBatch();
```

`Batch`メソッドを使用していても、TiDBに送信されるSQL文は個々の`INSERT`文です:

{{< copyable "sql" >}}

```sql
insert into t(a) values(10);
insert into t(a) values(11);
insert into t(a) values(12);
```

しかし、`rewriteBatchedStatements=true`を設定すると、TiDBに送信されるSQL文は単一の`INSERT`文になります:

{{< copyable "sql" >}}

```sql
insert into t(a) values(10),(11),(12);
```

`INSERT`文の書き換えは、複数の "values" キーワードの後の値を1つのSQL文に連結することです。`INSERT`文に他の違いがある場合は、書き換えることができません。例えば:

{{< copyable "sql" >}}

```sql
insert into t (a) values (10) on duplicate key update a = 10;
insert into t (a) values (11) on duplicate key update a = 11;
insert into t (a) values (12) on duplicate key update a = 12;
```

上記の`INSERT`文は1つの文に書き換えることができません。しかし、3つの文を以下のように変更すると:

{{< copyable "sql" >}}

```sql
insert into t (a) values (10) on duplicate key update a = values(a);
insert into t (a) values (11) on duplicate key update a = values(a);
insert into t (a) values (12) on duplicate key update a = values(a);
```

それによって、上記の`INSERT`文は以下の1つの文に書き換えられます:

{{< copyable "sql" >}}

```sql
insert into t (a) values (10), (11), (12) on duplicate key update a = values(a);
```

バッチ更新時に3つ以上の更新がある場合、SQL文は書き換えられて複数のクエリとして送信されます。これにより、クライアントからサーバへのリクエストのオーバーヘッドが効果的に削減されますが、副作用としてより大きなSQL文が生成されます。例えば:

{{< copyable "sql" >}}

```sql
update t set a = 10 where id = 1; update t set a = 11 where id = 2; update t set a = 12 where id = 3;
```

また、[クライアントのバグ](https://bugs.mysql.com/bug.php?id=96623)により、バッチ更新中に`rewriteBatchedStatements=true`と`useServerPrepStmts=true`を設定する場合は、このバグを避けるために`allowMultiQueries=true`パラメータも設定することをお勧めします。

#### 統合パラメータ

監視を通じて、TiDBクラスタに対してアプリケーションが`INSERT`操作のみを実行しているにもかかわらず、余分な`SELECT`文が多く存在することがあるかもしれません。通常、これはJDBCが設定をクエリするためにいくつかのSQL文をTiDBに送信することによるものです。例えば、`select @@session.transaction_read_only`などです。これらのSQL文はTiDBにとって無用なので、余分なオーバーヘッドを避けるために`useConfigs=maxPerformance`を設定することをお勧めします。

`useConfigs=maxPerformance`には、一連の設定が含まれます。MySQL Connector/J 8.0の詳細な設定とMySQL Connector/J 5.1の詳細な設定については、それぞれ[mysql-connector-j 8.0](https://github.com/mysql/mysql-connector-j/blob/release/8.0/src/main/resources/com/mysql/cj/configurations/maxPerformance.properties)および[mysql-connector-j 5.1](https://github.com/mysql/mysql-connector-j/blob/release/5.1/src/com/mysql/jdbc/configs/maxPerformance.properties)を参照してください。

これが設定されると、監視を確認して`SELECT`文の数が減少していることを確認できます。

#### タイムアウト関連のパラメータ

TiDBは、タイムアウトを制御する2つのMySQL互換パラメータ、`wait_timeout`と`max_execution_time`を提供しています。これらの2つのパラメータは、それぞれJavaアプリケーションとの接続のアイドルタイムアウトと接続内のSQL実行時間のタイムアウトを制御します。つまり、これらのパラメータは接続間の最長アイドル時間と最長ビジー時間を制御します。デフォルト値は`0`であり、これにより接続が無限にアイドル状態になり、1つのSQL文の実行のための無制限の時間が許可されます。

ただし、実際のプロダクション環境では、アイドル接続と過度に長い実行時間を持つSQL文はデータベースとアプリケーションに否定的な影響を与えます。アイドル接続と長時間実行されるSQL文を避けるためには、アプリケーションの接続文字列でこれらの2つのパラメータを設定することができます。例えば、`sessionVariables=wait_timeout=3600`（1時間）および`sessionVariables=max_execution_time=300000`（5分）と設定します。

## コネクションプール

TiDB（MySQL）のコネクションを構築することは比較的高価です（少なくともOLTPシナリオでは）。なぜなら、TCP接続を構築するだけでなく、接続認証も必要だからです。そのため、クライアントは通常、TiDB（MySQL）のコネクションをコネクションプールに保存して再利用します。

Javaには、[HikariCP](https://github.com/brettwooldridge/HikariCP)、[tomcat-jdbc](https://tomcat.apache.org/tomcat-10.1-doc/jdbc-pool.html)、[druid](https://github.com/alibaba/druid)、[c3p0](https://www.mchange.com/projects/c3p0/)、[dbcp](https://commons.apache.org/proper/commons-dbcp/)など、多くのコネクションプールの実装があります。TiDBはどの接続プールを使用するかに制限を加えていないため、アプリケーションで好きなものを選択することができます。

### コネクション数の設定

接続プールのサイズは、アプリケーションの独自のニーズに応じて適切に調整されるのが一般的です。HikariCPを例として取り上げます:

- `maximumPoolSize`: コネクションプール内の最大接続数です。この値が大きすぎると、TiDBは無用な接続を維持するためにリソースを消費します。逆に、この値が小さすぎると、アプリケーションは遅い接続を取得します。したがって、この値は適切に設定してください。詳細については、[About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)を参照してください。
- `minimumIdle`: コネクションプール内の最小アイドル接続数です。アプリケーションがアイドル状態のときに突発的なリクエストに応答するために、いくつかの接続を予約するために使用されます。アプリケーションのニーズに合わせて設定してください。

アプリケーションは使用後にコネクションを返す必要があります。また、アプリケーションは対応するコネクションプールの監視（`metricRegistry`など）を使用して、適切に接続プールの問題を特定することをお勧めします。

### プローブの構成

コネクションプールはTiDBとの持続的な接続を維持します。TiDBは通常、クライアントの接続を積極的に閉じることはしません（エラーが報告されない限り）、ただし一般的にクライアントとTiDBの間にLVSやHAProxyなどのネットワークプロキシが存在します。通常、これらのプロキシは一定期間アイドル状態にある接続を積極的にクリーンアップします（プロキシのアイドル構成によって制御されます）。プロキシのアイドル構成に注意するだけでなく、コネクションプールは接続を維持またはプローブする必要があります。

Javaアプリケーションで次のようなエラーが頻繁に発生する場合は、次のような解決策があります。

```
The last packet sent successfully to the server was 3600000 milliseconds ago. The driver has not received any packets from the server. com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

`n milliseconds ago`の`n`が`0`または非常に小さい値の場合、通常、実行されたSQL操作がTiDBを異常終了させたためです。原因を特定するには、TiDBのstderrログをチェックすることをお勧めします。

```yaml
      - もし `n` が非常に大きな値（上記の例では `3600000` など）である場合、この接続は長い時間アイドル状態であり、その後中間プロキシによって閉じられた可能性があります。通常の解決策は、プロキシのアイドル構成の値を増やし、接続プールに以下のことを許可することです。

  - 接続を使用する前に接続が利用可能かどうかを常に確認する
  - 別のスレッドを使用して定期的に接続が利用可能かどうかをチェックする
  - 定期的にテストクエリを送信し、接続をアクティブに保つ

  異なる接続プールの実装は、上記のいずれかを1つ以上サポートしている場合があります。対応する構成を見つけるために、接続プールのドキュメントを確認できます。

## データアクセスフレームワーク

アプリケーションは、通常、データアクセスを簡素化するためにデータアクセスフレームワークを使用します。

### MyBatis

[MyBatis](http://www.mybatis.org/mybatis-3/) は人気のあるJavaデータアクセスフレームワークです。主にSQLクエリの管理と結果セットとJavaオブジェクトのマッピングの完了に使用されます。MyBatisはTiDBと高い互換性があります。MyBatisにはその歴史的な問題に基づいて問題が発生することはほとんどありません。

ここでは、このドキュメントは主に以下の設定に焦点を当てています。

#### マッパーパラメータ

MyBatisマッパーは2つのパラメータをサポートしています。

- `select 1 from t where id = #{param1}` は、準備済みステートメントとして `select 1 from t where id =?` に変換され、「準備」として使用された実際のパラメータが再利用されます。このパラメータは、以前に説明した「プリペア接続パラメータ」と一緒に使用すると、最高のパフォーマンスを得ることができます。
- `select 1 from t where id = ${param2}` は、テキストファイルとして `select 1 from t where id = 1` に置き換えられ、実行されます。この文が異なるパラメータで置き換えられて実行されると、MyBatisはTiDBに対して「準備」ステートメントの異なるリクエストを送信する可能性があります。これにより、TiDBは大量の準備済みステートメントをキャッシュし、この方法でのSQL操作を実行すると、インジェクションセキュリティのリスクがあります。

#### ダイナミックSQLバッチ

[ダイナミックSQL - foreach](http://www.mybatis.org/mybatis-3/dynamic-sql.html#foreach)

`INSERT`ステートメントを複数の `...の形式に挿入するための動的なSQL`の再構築をサポートするために、JDBCで `rewriteBatchedStatements=true` を設定する必要があります。また、MyBatisはダイナミックSQLを使用してバッチ挿入を自動的に生成することもできます。次のマッパーを例に挙げます。

```xml
<insert id="insertTestBatch" parameterType="java.util.List" fetchSize="1">
  insert into test
   (id, v1, v2)
  values
  <foreach item="item" index="index" collection="list" separator=",">
  (
   #{item.id}, #{item.v1}, #{item.v2}
  )
  </foreach>
  on duplicate key update v2 = v1 + values(v1)
</insert>
```

このマッパーは `insert on duplicate key update` ステートメントを生成します。「values」の後に続く `(?,?,?)` の数は、渡されたリストの数によって決まります。その最終的な効果は、`rewriteBatchStatements=true` を使用するのと同様で、クライアントとTiDB間の通信オーバーヘッドも効果的に削減されます。

前述のように、`prepStmtCacheSqlLimit`の値を超えた場合、準備済みステートメントはキャッシュされないことにも注意する必要があります。

#### ストリーミング結果

[JDBCでストリーミングリザルトを使用して実行結果を取得するための前のセクション](#use-streamingresult-to-get-the-execution-result)は紹介しています。JDBCの対応する構成に加えて、MyBatisで非常に大きな結果セットを読み込む場合は、以下のことに注意する必要があります。

- マッパー構成の単一のSQLステートメントに `fetchSize` を設定できます（前述のコードブロックを参照）。その効果は、JDBCで `setFetchSize` を呼び出すことと等価です。
- `ResultHandler` を使用したクエリインターフェースを使用して、結果セット全体を一度に取得するのを避けることができます。
- `Cursor` クラスを使用してストリーム読み取りを行うことができます。

XMLを使用してマッピングを構成する場合、`fetchSize="-2147483648"`（`Integer.MIN_VALUE`）をマッピングの `<select>` セクションに設定することで、結果をストリーミング読み取りできます。

```xml
<select id="getAll" resultMap="postResultMap" fetchSize="-2147483648">
  select * from post;
</select>
```

コードを使用してマッピングを構成する場合、`@Options(fetchSize = Integer.MIN_VALUE)` アノテーションを追加し、結果のタイプを `Cursor` に保ち、そのSQL結果をストリーミング読み取りできます。

```java
@Select("select * from post")
@Options(fetchSize = Integer.MIN_VALUE)
Cursor<Post> queryAllPost();
```

### `ExecutorType`

`openSession`中に `ExecutorType` を選択することができます。MyBatisは3種類のエグゼキューターをサポートしています。

- Simple：それぞれの実行ごとに準備済みステートメントをJDBCに呼び出します（JDBCの構成項目`cachePrepStmts`が有効になっている場合、繰り返された準備済みステートメントが再利用されます）
- Reuse：準備済みステートメントは`executor`にキャッシュされるため、JDBCの `cachePrepStmts` を使用せずに準備済みステートメントの重複呼び出しを減らすことができます
- 実行される毎にバッチ：各更新操作（`INSERT` / `DELETE` / `UPDATE`）は最初にバッチに追加され、トランザクションがコミットされるか`SELECT`クエリが実行されるまで実行されます。JDBCレイヤーで`rewriteBatchStatements`が有効になっている場合、ステートメントを再構築しようとします。そうでない場合、ステートメントは1つずつ送信されます。

通常、`ExecutorType`のデフォルト値は `Simple` です。`openSession`を呼び出す際に `ExecutorType` を変更する必要があります。バッチ実行されている場合、トランザクション内での `UPDATE` や `INSERT` ステートメントの実行は非常に高速である一方、データの読み取りやトランザクションのコミットが遅くなることがあります。これは実際には正常な動作なので、遅いSQLクエリのトラブルシューティング時にはこれに注意する必要があります。

## Springトランザクション

実際の世界では、アプリケーションは[Springトランザクション](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html)とAOPアスペクトを使用してトランザクションの開始と終了を行うことがあります。

メソッドの定義に`@Transactional`アノテーションを追加することで、AOPはメソッドが呼び出される前にトランザクションを開始し、メソッドが結果を返す前にトランザクションをコミットします。類似した必要がある場合、コード内で `@Transactional` を見つけて、いつトランザクションが開始され、終了されるかを確認できます。

特殊な埋め込みの場合に注意してください。それが発生した場合、Springは[Propagation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html)構成に基づいて異なる動作をします。

## その他

このセクションでは、Java向けの問題トラブルシューティングを支援するいくつかの有用なツールを紹介します。

### トラブルシューティングツール

Javaアプリケーションで問題が発生し、アプリケーションのロジックがわからない場合は、JVMの強力なトラブルシューティングツールを使用することをお勧めします。ここには、いくつかの一般的なツールがあります。

#### jstack

[jstack](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstack.html) は、Goのpprof/goroutineに似ており、プロセスがスタックした問題を簡単にトラブルシューティングできます。

`jstack pid` を実行することで、対象プロセスのすべてのスレッドのIDとスタック情報を出力できます。デフォルトでは、Javaスタックのみが出力されます。同時にJVM内のC++スタックも出力したい場合は、`-m`オプションを追加します。

jstackを複数回使用することで、スタックした問題（たとえば、MybatisでBatch ExecutorTypeを使用するため、アプリケーションビューからの遅いクエリが発生する問題）やアプリケーションのデッドロック問題（たとえば、送信する前にロックが先に獲得されるためアプリケーションがどのようなSQLステートメントも送信しない場合）を簡単に特定できます。

その他にも、`top -p $ PID -H` やJavaスイスナイフなどがスレッドIDを表示する一般的な方法があります。また、"スレッドが多くのCPUリソースを占有し、それが何を実行しているのかわからない"という問題を特定するには、以下の手順があります。

- `printf "%x\n" pid` を使用して、スレッドIDを16進数に変換します。
- 対応するスレッドのスタック情報をjstackの出力から見つけます。

#### jmap & mat

Goのpprof/heapとは異なり、[jmap](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jmap.html) はプロセス全体のメモリスナップショットをダンプし、そのスナップショットは別のツールである[mat](https://www.eclipse.org/mat/)で解析することができます。

matを使用すると、プロセスのすべてのオブジェクトの関連情報や属性を見ることができ、スレッドの実行状況を観察することができます。たとえば、matを使用して、現在のアプリケーションに存在するMySQL接続オブジェクトの数や、各接続オブジェクトのアドレスや状態情報を見つけることができます。

matはデフォルトでは到達可能なオブジェクトのみを扱います。若いGCの問題をトラブルシューティングする場合は、matの構成を調整して到達不能なオブジェクトを表示することができます。また、若いGCのメモリ割り当ての調査（または多数の短命であるオブジェクト）については、Javaフライトレコーダーを使用する方が便利です。

#### トレース

オンラインアプリケーションでは通常コードを変更できませんが、Javaでダイナミックインストルメンテーションを実行して問題を特定することがよく求められます。そのため、btraceやarthas traceを使用することをお勧めします。これらのツールを使用すると、アプリケーションプロセスを再起動することなく動的にトレースコードを挿入することができます。

#### フレームグラフ
```
```
Obtaining flame graphs in Java applications is tedious. For details, see [Java Flame Graphs Introduction: Fire For Everyone!](http://psy-lob-saw.blogspot.com/2017/02/flamegraphs-intro-fire-for-everyone.html).

## Conclusion

Based on commonly used Java components that interact with databases, this document describes the common problems and solutions for developing Java applications with TiDB. TiDB is highly compatible with the MySQL protocol, so most of the best practices for MySQL-based Java applications also apply to TiDB.

Join us at [TiDB Community slack channel](https://tidbcommunity.slack.com/archives/CH7TTLL7P), and share with broad TiDB user group about your experience or problems when you develop Java applications with TiDB.
```

```
Javaアプリケーションでフレームグラフを取得するのは手間がかかります。詳細については、[Java Flame Graphs Introduction: Fire For Everyone!](http://psy-lob-saw.blogspot.com/2017/02/flamegraphs-intro-fire-for-everyone.html)を参照してください。

## 結論

データベースとやりとりする一般的に使用されるJavaコンポーネントに基づき、このドキュメントはTiDBを使用したJavaアプリケーションの開発における一般的な問題と解決策について説明しています。TiDBはMySQLプロトコルと高度に互換性があるため、MySQLベースのJavaアプリケーションのほとんどのベストプラクティスはTiDBにも適用されます。

[TiDB Community slack channel](https://tidbcommunity.slack.com/archives/CH7TTLL7P)で私たちに参加し、TiDBを使用したJavaアプリケーションの開発時における経験や問題について幅広いTiDBユーザーグループと共有してください。
```