---
title: コネクションプールとコネクションパラメータ
---

# コネクションプールとコネクションパラメータ

この文書では、TiDBに接続する際にドライバやORMフレームワークを使用する際のコネクションプールとコネクションパラメータの設定方法について説明します。

<CustomContent platform="tidb">

Javaアプリケーション開発に関するさらなるヒントに興味がある場合は、[TiDBを使用したJavaアプリケーションのベストプラクティス](/best-practices/java-app-best-practices.md#connection-pool)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

Javaアプリケーション開発に関するさらなるヒントに興味がある場合は、[TiDBを使用したJavaアプリケーションのベストプラクティス](https://docs.pingcap.com/tidb/stable/java-app-best-practices)を参照してください。

</CustomContent>

## コネクションプール

TiDB（MySQL）の接続の構築は比較的高コストです（少なくともOLTPシナリオでは）。なぜなら、TCP接続の構築に加えて、接続認証も必要とされるためです。そのため、クライアントは通常、TiDB（MySQL）の接続をコネクションプールに保存して再利用します。

Javaには、[HikariCP](https://github.com/brettwooldridge/HikariCP)、[tomcat-jdbc](https://tomcat.apache.org/tomcat-10.1-doc/jdbc-pool.html)、[druid](https://github.com/alibaba/druid)、[c3p0](https://www.mchange.com/projects/c3p0/)、[dbcp](https://commons.apache.org/proper/commons-dbcp/)など、多くのコネクションプールの実装があります。TiDBは使用するコネクションプールを制限していませんので、アプリケーションに合わせて好きなものを選択することができます。

### 接続数の設定

コネクションプールのサイズをアプリケーションのニーズに適切に調整するのは一般的な慣習です。例としてHikariCPを取り上げます：

- **maximumPoolSize**: コネクションプール内の最大接続数です。この値が大きすぎると、TiDBは無駄な接続を維持するためのリソースを消費します。この値が小さすぎると、アプリケーションは遅い接続を取得します。そのため、アプリケーションの特性に応じてこの値を設定する必要があります。詳細については、[About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)を参照してください。
- **minimumIdle**: コネクションプール内の最小アイドル接続数です。アプリケーションがアイドル状態で突然のリクエストに応答するために使用されます。これもアプリケーションの特性に応じて設定する必要があります。

アプリケーションは使用後に接続を返す必要があります。接続プールの監視（たとえば**metricRegistry**）を使用して、接続プールの問題を迅速に特定することをお勧めします。

### 接続プールの設定

コネクションプールはTiDBとの持続的な接続を維持します。通常、TiDBはクライアント接続を積極的に閉じません（エラーが報告されない限り）。しかし一般的に、クライアントとTiDBの間に[LVS](https://en.wikipedia.org/wiki/Linux_Virtual_Server)や[HAProxy](https://en.wikipedia.org/wiki/HAProxy)などのネットワークプロキシが存在します。通常、これらのプロキシは一定期間アイドル状態の接続を積極的にクリーンアップします。プロキシのアイドル設定に注意するだけでなく、コネクションプールは接続をアクティブに保つか、または接続のプローブを行う必要があります。

Javaアプリケーションで次のエラーが頻繁に発生する場合は、

```
The last packet sent successfully to the server was 3600000 milliseconds ago. The driver has not received any packets from the server. com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

`n milliseconds ago`の`n`が`0`または非常に小さい値の場合、通常は実行されたSQL操作がTiDBを異常終了させたためです。原因を見つけるには、TiDBの標準エラーログを確認することをお勧めします。

`n`が非常に大きな値（上記の例では`3600000`など）である場合、この接続が長時間アイドル状態であり、その後にプロキシによって閉じられた可能性が高いです。一般的な解決策は、プロキシのアイドル設定の値を増やし、接続プールに対して次のことを行うことです。

- 接続を使用する前に接続の可用性を確認する。
- 別のスレッドを使用して定期的に接続の可用性をチェックする。
- 定期的にテストクエリを送信して接続をアクティブに保つ。

異なるコネクションプールの実装では、上記のいずれか以上のメソッドをサポートしている場合があります。対応する設定を見つけるためにコネクションプールのドキュメントを確認できます。

### 経験に基づく公式

HikariCPの[About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)記事によると、データベースコネクションプールの適切なサイズ設定方法がわからない場合は、[経験に基づく公式](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing#connections--core_count--2--effective_spindle_count)を使用して始めることをお勧めします。その後、この公式から計算されたプールサイズのパフォーマンス結果に基づいて、最適なパフォーマンスを得るためにさらにサイズを調整することができます。

経験に基づく公式は次のようになります：

```
connections = ((core_count * 2) + effective_spindle_count)
```

公式の各パラメータの説明は次のとおりです：

- **connections**: 取得した接続のサイズ。
- **core_count**: CPUコアの数。
- **effective_spindle_count**: ハードドライブの数（[SSD](https://en.wikipedia.org/wiki/Solid-state_drive)ではない）。回転するハードディスク1つが1つのスピンドルとみなされます。たとえば、16台のハードディスクのRAIDを使用している場合、**effective_spindle_count**は16になります。なぜなら、**HDD**は通常一度に1つのリクエストしか処理できないため、この公式は実際にはサーバーが処理できる同時I/Oリクエストの数を測定しています。

特に、[公式](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing#the-formula)の下に以下のような注意事項に注意してください。

> ```
> この公式は長年にわたる多くのベンチマークでかなりよく機能していることが確認されています。最適なスループットを得るには、アクティブな接続の数はおおよそ((core_count * 2) + effective_spindle_count)に近い必要があります。Hyper-Threadingが有効であっても、コア数にはHTスレッドは含めないでください。データセットが完全にキャッシュされている場合は、**effective_spindle_count**は0に設定する必要があり、キャッシュヒット率が低下すると、この数は実際の**HDD**の数に近づきます。...SSDでこの公式がどの程度機能するかについては、これまでに分析されていません。
> ```

この注意事項には次のように記載されています：

- **core_count** は物理コアの数であり、Hyper-Threadingを有効にしていても[HTスレッド](https://en.wikipedia.org/wiki/Hyper-threading)は含まれません。
- データが完全にキャッシュされている場合は**effective_spindle_count**を`0`に設定する必要があります。キャッシュのヒット率が低下すると、このカウントは実際の`HDD`の数に近づいてきます。
- **SSD**の場合、この公式がどのように機能するかについてはテストされておらず不明です。

SSDを使用する場合は、代わりに次の経験に基づく公式を使用することがお勧めされます：

```
connections = (コア数 * 4)
```

したがって、SSDの場合、初期のコネクションプールの最大接続サイズを`コア数 * 4`に設定し、さらにサイズを調整してパフォーマンスを調整することができます。

### 調整方向

[経験に基づく公式](#formulas-based-on-experience)から計算されたサイズは、推奨される基本値に過ぎません。特定のマシンで最適なサイズを得るには、基本値を中心とした周辺の値を試してパフォーマンスをテストする必要があります。

最適なサイズを取得するための基本的なルールを以下に示します：

- ネットワークまたはストレージの待ち時間が長い場合は、最大接続数を増やして待ち時間を減らします。スレッドが待ち時間によってブロックされると、他のスレッドが引き継いで処理を続行できます。
- サーバーに複数のサービスを展開し、それぞれのサービスに別々のコネクションプールがある場合は、すべてのコネクションプールの最大接続数の合計を考慮してください。

## コネクションパラメータ

Javaアプリケーションはさまざまなフレームワークでカプセル化されることがあります。ほとんどのフレームワークでは、データベースサーバーとやり取りするために最下層でJDBC APIが呼び出されます。JDBCの場合、以下の点について焦点を当てることをお勧めします。

- JDBC APIの使用方法の選択
- API実装者のパラメーター設定

### JDBC API

JDBC APIの使用方法については、[JDBC公式チュートリアル](https://docs.oracle.com/javase/tutorial/jdbc/)を参照してください。このセクションでは、いくつかの重要なAPIの使用方法について説明します。

#### Prepare APIの使用

OLTP（オンライントランザクション処理）シナリオでは、プログラムからデータベースサーバーに送信されるSQLステートメントは、パラメータの変更を取り除いても使い果たされる可能性のある複数のタイプです。そのため、[Prepared Statements](https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html)を使用して、テキストファイルからの通常の実行の代わりに事前に準備されたステートメントを再利用して直接実行することをお勧めします。これにより、TiDBでのSQLの繰り返しの解析と実行計画の生成のオーバーヘッドを回避できます。

現在、ほとんどの上位レベルのフレームワークは、SQLの実行にPrepare APIを呼び出しています。開発にJDBC APIを直接使用する場合は、Prepare APIの選択に注意してください。
また、MySQL Connector/Jのデフォルトの実装では、クライアント側のステートメントのみが事前処理され、ステートメントはクライアントで`?`が置換された後にサーバーにテキストファイルで送信されます。したがって、Prepare APIを使用するだけでなく、TiDBサーバーでステートメントの事前処理を実行する前に、JDBC接続パラメータで`useServerPrepStmts = true`を設定する必要があります。詳細なパラメータ構成方法については、「[MySQL JDBCパラメータ](#mysql-jdbc-parameters)」を参照してください。

#### バッチAPIの使用

複数の挿入を行う場合、[`addBatch`/`executeBatch` API](https://www.tutorialspoint.com/jdbc/jdbc-batch-processing)を使用できます。 `addBatch()`メソッドは、まず複数のSQLステートメントをクライアントでキャッシュし、`executeBatch`メソッドを呼び出すときにそれらをまとめてデータベースサーバーに送信します。

> **メモ:**
>
> デフォルトのMySQL Connector/Jの実装では、`addBatch()`でバッチに追加されたSQLステートメントの送信は`executeBatch()`が呼び出される時まで遅延されますが、実際のネットワーク転送中にステートメントはまだ1つずつ送信されます。したがって、通常は通信オーバーヘッドが削減されないことが多いです。
>
> ネットワーク転送をバッチ処理するためには、JDBC接続パラメータで`rewriteBatchedStatements = true`を構成する必要があります。詳細なパラメータ構成方法については、「[バッチ関連のパラメータ](#batch-related-parameters)」を参照してください。

#### 実行結果を取得するために`StreamingResult`を使用する

ほとんどのシナリオでは、JDBCはデフォルトで事前にクエリ結果を取得し、それらをクライアントメモリに保存します。ただし、クエリが超大きな結果セットを返す場合、クライアントは通常、データベースサーバーに対して一度に返却されるレコード数を減らし、クライアントのメモリが準備できるまで待機して、次のバッチのリクエストを行います。

通常、JDBCには2種類の処理方法があります。

- [**FetchSize**を`Integer.MIN_VALUE`に設定](https://dev.mysql.com/doc/connector-j/ja/connector-j-reference-implementation-notes.html#ResultSet)して、クライアントがキャッシュしないようにします。クライアントは`StreamingResult`を介してネットワーク接続から実行結果を読み取ります。

    クライアントがストリーミング読み取りメソッドを使用する場合は、クエリを続ける前に`resultset`の読み取りを終了するか`resultset`をクローズする必要があります。そうでない場合は、エラー`No statements may be issued when any streaming result sets are open and in use on a given connection. Ensure that you have called .close() on any active streaming result sets before attempting more queries.`が返されます。

    クライアントが`resultset`の読み取りを終了する前や`resultset`をクローズする前にこのようなエラーが発生しないようにするには、URLに`clobberStreamingResults=true`パラメータを追加できます。すると、`resultset`は自動的にクローズされますが、以前のストリーミングクエリの結果セットは失われます。

- カーソルフェッチを使用する場合、最初に`FetchSize`を正の整数に[設定](http://makejavafaster.blogspot.com/2015/06/jdbc-fetch-size-performance.html)し、JDBC URLで`useCursorFetch=true`を構成します。

TiDBは両方の方法をサポートしていますが、より簡単な実装で実行効率が向上するため、最初の方法を使用することが推奨されています。

### MySQL JDBCパラメータ

JDBCは通常、JDBC URLパラメータの形式で実装に関連する構成を提供します。このセクションでは、[MySQL Connector/Jのパラメータ構成](https://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html)を紹介しています（MariaDBを使用する場合は、[MariaDBのパラメータ構成](https://mariadb.com/kb/en/library/about-mariadb-connector-j/#optional-url-parameters)を参照してください）。このドキュメントではすべての構成項目を網羅することはできませんが、パフォーマンスに影響を与える可能性のあるいくつかのパラメータに主に焦点を当てています。

#### Prepare関連のパラメータ

このセクションでは、「Prepare」に関連するパラメータを紹介します。

- **useServerPrepStmts**

    **useServerPrepStmts**はデフォルトで`false`に設定されており、つまり、Prepare APIを使用しても、「prepare」操作はクライアントでのみ実行されます。サーバーのパースオーバーヘッドを避けるために、同じSQLステートメントが複数回Prepare APIを使用する場合は、この構成を`true`に設定することを推奨します。

    この設定が既に有効であることを確認するには、次の手順を実行できます:

    - TiDB監視ダッシュボードにアクセスし、「クエリサマリ」>「インスタンス別コマンドタイプ」を介してリクエストのコマンドタイプを表示します。
    - リクエスト内の`COM_QUERY`が、`COM_STMT_EXECUTE`または`COM_STMT_PREPARE`に置換されている場合、この設定が既に有効であることを示します。

- **cachePrepStmts**

    `useServerPrepStmts=true`に設定しても、デフォルトではクライアントはPrepared Statementsを実行後に閉じて再利用しません。つまり、"prepare"操作さえもテキストファイルの実行ほど効率的ではありません。そのため、`useServerPrepStmts=true`を設定した後に、`cachePrepStmts=true`を構成することを推奨します。

    この設定が既に有効であることを確認するには、次の手順を実行できます:

    - TiDB監視ダッシュボードにアクセスし、「クエリサマリ」>「インスタンス別コマンドタイプ」を介してリクエストのコマンドタイプを表示します。
    - リクエスト内の`COM_STMT_EXECUTE`の数が`COM_STMT_PREPARE`の数よりもかなり多い場合、この設定が既に有効であることを示します。

    また、`useConfigs=maxPerformance`を構成すると、`cachePrepStmts=true`を含む複数のパラメータが同時に構成されます。

- **prepStmtCacheSqlLimit**

    `cachePrepStmts`を構成した後は、`prepStmtCacheSqlLimit`構成（デフォルト値は`256`）にも注意する必要があります。この設定はクライアントでキャッシュされるPrepared Statementsの最大長を制御します。

    この最大長を超えるPrepared Statementsはキャッシュされず、再利用できません。このような場合は、アプリケーションの実際のSQLの長さに応じてこの構成の値を増やすことを検討する必要があります。

    この設定が小さすぎるかどうかを確認するには、次の手順を実行できます:

    - TiDB監視ダッシュボードにアクセスし、「クエリサマリ」>「インスタンス別コマンドタイプ」を介してリクエストのコマンドタイプを表示します。
    - `cachePrepStmts=true`が構成されていても、`COM_STMT_PREPARE`がほとんど等しいままである場合、また`COM_STMT_CLOSE`が存在する場合は、この設定が小さい可能性があります。

- **prepStmtCacheSize**

    **prepStmtCacheSize**はキャッシュされるPrepared Statementsの数を制御します（デフォルト値は`25`）。アプリケーションが多くの種類のSQLステートメントを「準備」し、準備したステートメントを再利用したい場合は、この値を増やすことができます。

    この設定が既に有効であることを確認するには、次の手順を実行できます:

    - TiDB監視ダッシュボードにアクセスし、「クエリサマリ」>「インスタンス別コマンドタイプ」を介してリクエストのコマンドタイプを表示します。
    - リクエスト内の`COM_STMT_EXECUTE`の数が`COM_STMT_PREPARE`の数よりもかなり多い場合、この設定が既に有効であることを示します。

#### バッチ関連のパラメータ

バッチ書き込みを処理する際には、`rewriteBatchedStatements=true`を構成することを推奨します。`addBatch()`または`executeBatch()`を使用しても、JDBCはデフォルトで1つずつSQLを送信します。

```java
pstmt = prepare("INSERT INTO `t` (a) values(?)");
pstmt.setInt(1, 10);
pstmt.addBatch();
pstmt.setInt(1, 11);
pstmt.addBatch();
pstmt.setInt(1, 12);
pstmt.executeBatch();
```

しかし、`rewriteBatchedStatements=true`を設定すると、TiDBに送信されるSQLステートメントは1つの`INSERT`ステートメントになります。
バッチ更新中に3回以上の更新がある場合、SQLステートメントは書き換えられ、複数のクエリとして送信されます。これによりクライアントからサーバーへのリクエストオーバーヘッドが効果的に削減されますが、副作用としてより大きなSQLステートメントが生成されます。例えば：

```sql
UPDATE `t` SET `a` = 10 WHERE `id` = 1; UPDATE `t` SET `a` = 11 WHERE `id` = 2; UPDATE `t` SET `a` = 12 WHERE `id` = 3;
```

また、[クライアントのバグ](https://bugs.mysql.com/bug.php?id=96623)のため、バッチ更新中に`rewriteBatchedStatements=true`および`useServerPrepStmts=true`を構成しようとする場合、このバグを回避するために`allowMultiQueries=true`パラメータを構成することを推奨します。

#### パラメータの統合

モニタリングを通じて、アプリケーションがTiDBクラスターに対して`INSERT`操作のみを実行しているにもかかわらず、冗長な`SELECT`ステートメントが多く存在することに気付くかもしれません。通常、これはJDBCがいくつかのSQLステートメントを送信して設定をクエリするために発生します。たとえば、`select @@session.transaction_read_only`などです。これらのSQLステートメントはTiDBにとって無用なので、余分なオーバーヘッドを避けるために`useConfigs=maxPerformance`を構成することを推奨します。

`useConfigs=maxPerformance`には、一連の構成が含まれます。MySQL Connector/J 8.0およびMySQL Connector/J 5.1の詳細な構成を取得するには、[mysql-connector-j 8.0](https://github.com/mysql/mysql-connector-j/blob/release/8.0/src/main/resources/com/mysql/cj/configurations/maxPerformance.properties)および[mysql-connector-j 5.1](https://github.com/mysql/mysql-connector-j/blob/release/5.1/src/com/mysql/jdbc/configs/maxPerformance.properties)を参照してください。

構成したら、モニタリングを確認して、`SELECT`ステートメントの数が減少したことを確認できます。

#### タイムアウト関連のパラメータ

TiDBでは、タイムアウトを制御するための2つのMySQL互換のパラメータが提供されます：[`wait_timeout`](/system-variables.md#wait_timeout)と[`max_execution_time`](/system-variables.md#max_execution_time)。これらの2つのパラメータは、それぞれJavaアプリケーションとの接続のアイドルタイムアウトと接続間のSQL実行の最大タイムアウトを制御します。つまり、これらのパラメータはTiDBとJavaアプリケーション間の接続の最も長いアイドル時間および最も長いビジー時間を制御します。TiDB v5.4から、`wait_timeout`のデフォルト値は`28800`秒、つまり8時間です。v5.4より前のTiDBバージョンでは、デフォルト値は`0`で、タイムアウトが無制限であることを意味します。`max_execution_time`のデフォルト値は`0`で、これはSQLステートメントの最大実行時間が無制限であることを意味します。

ただし、実際の本番環境では、アイドル状態の接続や実行時間が極端に長いSQLステートメントがデータベースやアプリケーションに否定的な影響を与えます。アイドル状態の接続や長時間実行されたSQLステートメントを回避するために、これらの2つのパラメータをアプリケーションの接続文字列で構成できます。例えば、`sessionVariables=wait_timeout=3600`（1時間）および`sessionVariables=max_execution_time=300000`（5分）を設定してください。