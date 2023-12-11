---
title: インデックスのベストプラクティス
summary: TiDBでのインデックスの作成と使用のためのベストプラクティスについて学びます。
---

<!-- markdownlint-disable MD029 -->

# インデックスのベストプラクティス

このドキュメントでは、TiDBでのインデックスの作成と使用に関するいくつかのベストプラクティスについて紹介します。

## 開始前の注意事項

このセクションでは、[書店](/develop/dev-guide-bookshop-schema-design.md)データベースの`books`テーブルを例に取り上げます。

```sql
CREATE TABLE `books` (
  `id` bigint(20) AUTO_RANDOM NOT NULL,
  `title` varchar(100) NOT NULL,
  `type` enum('Magazine', 'Novel', 'Life', 'Arts', 'Comics', 'Education & Reference', 'Humanities & Social Sciences', 'Science & Technology', 'Kids', 'Sports') NOT NULL,
  `published_at` datetime NOT NULL,
  `stock` int(11) DEFAULT '0',
  `price` decimal(15,2) DEFAULT '0.0',
  PRIMARY KEY (`id`) CLUSTERED
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

## インデックスの作成のベストプラクティス

- 複数のカラムでの結合インデックスを作成することで、[カバリングインデックスの最適化](/explain-indexes.md#indexreader)と呼ばれる最適化を実施します。**カバリングインデックス最適化**により、TiDBはインデックス上でデータを直接問い合わせることができ、パフォーマンスを向上させることができます。
- リードされる頻度の低いカラムにはセカンダリインデックスを作成しないでください。有用なセカンダリインデックスはクエリの高速化に役立ちますが、追加でキーと値が行挿入時に追加されるため、インデックスが多いほど書き込みが遅く、スペースを消費します。さらに、多すぎるインデックスはオプティマイザのランタイムに影響を与え、適切でないインデックスはオプティマイザを誤導する可能性があります。そのため、インデックスが多いからといって必ずしもパフォーマンスが向上するわけではありません。
- アプリケーションに基づいて適切なインデックスを作成してください。基本的には、クエリで使用されるカラムにのみインデックスを作成してパフォーマンスを向上させるべきです。次のケースでは、インデックスの作成が適しています：

    - 大きな差異を持つカラムは、フィルタリングされる行の数を大幅に減らすことができます。たとえば、個人のID番号にはインデックスを作成することをお勧めしますが、性別には作成しないでください。
    - 複数の条件でクエリを実行する場合は、結合インデックスを使用してください。同等条件のカラムは結合インデックスの先頭に配置する必要があります。次の例を参照してください：`select* from t where c1 = 10 and c2 = 100 and c3 > 10`のクエリが頻繁に使用される場合は、結合インデックス`Index cidx (c1, c2, c3)`を作成してください。このようにすることで、クエリ条件によってスキャンするためのインデックスプレフィックスが構築されます。

- 意味のある名前でセカンダリインデックスを命名し、会社や組織のテーブル命名規則に従うことをお勧めします。このような命名規則が存在しない場合は、[インデックスの命名仕様](/develop/dev-guide-object-naming-guidelines.md)のルールに従ってください。

## インデックスの使用のベストプラクティス

- クエリの高速化を目的としてインデックスを作成するため、既存のインデックスが実際にいくつかのクエリで使用されていることを確認してください。インデックスが1つもクエリで使用されていない場合は、そのインデックスは意味をなさず、削除する必要があります。
- 結合インデックスを使用する場合は、左プレフィックスのルールに従ってください。

    例えば、`title`および`published_at`の新しい結合インデックスを作成するとします：

    {{< copyable "sql" >}}

    ```sql
    CREATE INDEX title_published_at_idx ON books (title, published_at);
    ```

    以下のクエリは引き続き結合インデックスを使用することができます：

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE title = 'database';
    ```

    ただし、次のクエリは結合インデックスを使用することができません。なぜなら、インデックスの最初の左側の条件が指定されていないからです：

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE published_at = '2018-08-18 21:42:08';
    ```

- クエリ条件でインデックスカラムを使用する際には、そのカラムに対して計算、関数、または型変換を使用しないでください。これにより、TiDBオプティマイザがインデックスを使用できなくなります。

    たとえば、`published_at`というタイムタイプのカラムに新しいインデックスを作成するとします：

    {{< copyable "sql" >}}

    ```sql
    CREATE INDEX published_at_idx ON books (published_at);
    ```

    ただし、次のクエリでは`published_at`のインデックスを使用することができません：

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE YEAR(published_at)=2022;
    ```

    `published_at`のインデックスを使用するには、次のようにクエリを書き直してください。これにより、インデックスカラムに対して関数を使用しないようになります。

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE published_at >= '2022-01-01' AND published_at < '2023-01-01';
    ```

    また、クエリ条件で`YEAR(Published at)`に対して式インデックスを作成することもできます：

    {{< copyable "sql" >}}

    ```sql
    CREATE INDEX published_year_idx ON books ((YEAR(published_at)));
    ```

    これで`SELECT * FROM books WHERE YEAR(published_at)=2022;`クエリを実行すると、クエリは`published_year_idx`インデックスを使用して実行を高速化することができます。

    > **注意：**
    >
    > 現在、式インデックスは実験的な機能であり、TiDB構成ファイルで有効にする必要があります。詳細については、[式インデックス](/sql-statements/sql-statement-create-index.md#expression-index)を参照してください。

- カバリングインデックスを使用することを検討し、インデックス内のカラムがクエリされるカラムを含むようにしてください。また、`SELECT *`文ですべてのカラムをクエリしないようにしてください。

    次のクエリは、データを取得するためにインデックス`title_published_at_idx`のみをスキャンする必要があります：

    {{< copyable "sql" >}}

    ```sql
    SELECT title, published_at FROM books WHERE title = 'database';
    ```

    次のクエリ文は、`(title, published_at)`の結合インデックスを使用できますが、インデックスされていないカラムのクエリに対して追加コストが発生するため、通常は避けるべきです。これにより、通常は主キー情報に従ってインデックスデータに格納された参照に基づいて行データをクエリする必要があります。

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE title = 'database';
    ```

- クエリ条件に`!=`または`NOT IN`が含まれている場合、クエリはインデックスを使用できません。たとえば、次のクエリはいかなるインデックスも使用できません：

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE title != 'database';
    ```

- クエリ条件で`LIKE`条件が`%`で始まる場合、クエリはインデックスを使用できません。たとえば、次のクエリはいかなるインデックスも使用できません：

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM books WHERE title LIKE '%database';
    ```

- クエリ条件に複数のインデックスが利用可能な場合、実用的なベストなインデックスを事前に知っている場合は、[オプティマイザヒント](/optimizer-hints.md)を使用してTiDBオプティマイザにこのインデックスを使用するよう指示することをお勧めします。これにより、統計情報の不正確さやその他の問題によりTiDBオプティマイザが誤ったインデックスを選択することを防ぐことができます。

    次のクエリで、`id`と`title`の列に対してインデックス`id_idx`および`title_idx`が利用可能な場合、`id_idx`を使用するようにTiDBオプティマイザに強制するためにSQLで`USE INDEX`ヒントを使用してください。

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM t USE INDEX(id_idx) WHERE id = 1 and title = 'database';
    ```

- クエリ条件で`IN`式を使用する場合、その後に一致する値の数が300を超えないようにすることをお勧めします。それ以外の場合、実行効率が低下する可能性があります。