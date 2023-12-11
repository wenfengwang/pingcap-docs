---
title: テーブルフィルター
summary: TiDBツールでのテーブルフィルター機能の使用方法
aliases: ['/docs/dev/tidb-lightning/tidb-lightning-table-filter/', '/docs/dev/reference/tools/tidb-lightning/table-filter/', '/tidb/dev/tidb-lightning-table-filter/']
---

# テーブルフィルター

TiDB移行ツールは通常、すべてのデータベースで動作しますが、多くの場合、サブセットのみが必要です。たとえば、`foo*`と`bar*`の形式のスキーマのみを操作したい場合があります。

TiDB 4.0以降、すべてのTiDB移行ツールはサブセットを定義するための共通のフィルター構文を共有します。このドキュメントでは、テーブルフィルター機能の使用方法について説明します。

## 使用法

### CLI

テーブルフィルターは、複数の`-f`または`--filter`コマンドラインパラメータを使用してツールに適用できます。各フィルターは`db.table`の形式であり、各部分はワイルドカードである場合があります（[次のセクション](#ワイルドカード)で詳しく説明します）。以下に例を示します。

<CustomContent platform="tidb">

* [BR](/br/backup-and-restore-overview.md):

    ```shell
    ./br backup full -f 'foo*.*' -f 'bar*.*' -s 'local:///tmp/backup'
    ```

    ```shell
    ./br restore full -f 'foo*.*' -f 'bar*.*' -s 'local:///tmp/backup'
    ```

</CustomContent>

* [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview):

    ```shell
    ./dumpling -f 'foo*.*' -f 'bar*.*' -P 3306 -o /tmp/data/
    ```

<CustomContent platform="tidb">

* [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md):

    ```shell
    ./tidb-lightning -f 'foo*.*' -f 'bar*.*' -d /tmp/data/ --backend tidb
    ```

</CustomContent>

<CustomContent platform="tidb-cloud">

* [TiDB Lightning](https://docs.pingcap.com/tidb/stable/tidb-lightning-overview):

    ```shell
    ./tidb-lightning -f 'foo*.*' -f 'bar*.*' -d /tmp/data/ --backend tidb
    ```

</CustomContent>

### TOML構成ファイル

TOMLファイルのテーブルフィルターは、[文字列の配列](https://toml.io/en/v1.0.0-rc.1#section-15)として指定されます。以下に例を示します。

* TiDB Lightning:

    ```toml
    [mydumper]
    filter = ['foo*.*', 'bar*.*']
    ```

<CustomContent platform="tidb">

* [TiCDC](/ticdc/ticdc-overview.md):

    ```toml
    [filter]
    rules = ['foo*.*', 'bar*.*']

    [[sink.dispatchers]]
    matcher = ['db1.*', 'db2.*', 'db3.*']
    dispatcher = 'ts'
    ```

</CustomContent>

## 構文

### 単純なテーブル名

各テーブルフィルター規則は、「スキーマパターン」と「テーブルパターン」で構成され、ドット（`.`）で区切られます。規則に一致する完全修飾名のテーブルが受け入れられます。

```
db1.tbl1
db2.tbl2
db3.tbl3
```

単純な名前は、[識別子の文字](/schema-object-names.md)で構成されている必要があります。たとえば:

* 数字（`0`から`9`）
* 文字（`a`から`z`、`A`から`Z`）
* `$`
* `_`
* ASCII以外の文字（U+0080からU+10FFFF）

その他のASCII文字は予約されています。一部の句読点には特別な意味がありますが、次のセクションで説明します。

### ワイルドカード

名前の各部分は、[fnmatch(3)](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_13)で説明されているワイルドカード記号である場合があります:

* `*` — 0文字以上に一致
* `?` — 1文字に一致
* `[a-z]` — "a"から "z"の間の1文字に一致
* `[!a-z]` — "a"から "z"以外の1文字に一致。

```
db[0-9].tbl[0-9a-f][0-9a-f]
data.*
*.backup_*
```

ここでの「文字」はUnicodeのコードポイントを意味し、たとえば:

* U+00E9 (é)は1文字です。
* U+0065 U+0301 (é)は2文字です。
* U+1F926 U+1F3FF U+200D U+2640 U+FE0F (🤦🏿‍♀️)は5文字です。

### ファイルインポート

フィルタールールとしてファイルをインポートするには、ルールの先頭に`@`を付けてファイル名を指定します。テーブルフィルターパーサーは、インポートされたファイルの各行を追加のフィルタールールとして扱います。

たとえば、ファイル`config/filter.txt`に次の内容が含まれている場合:

```
employees.*
*.WorkOrder
```

次の2つの呼び出しは等価です:

```bash
./dumpling -f '@config/filter.txt'
./dumpling -f 'employees.*' -f '*.WorkOrder'
```

フィルターファイルは他のファイルをさらにインポートすることはできません。

### コメントと空行

フィルターファイル内では、各行の先頭および末尾の空白はトリミングされます。さらに、空行（空の文字列）は無視されます。

先頭に`#`が付いている行はコメントとして扱われ、無視されます。行の先頭に `#` がない場合は、構文エラーとみなされます。

```
# この行はコメントです
db.table   # ただし、この部分はコメントではなく、エラーの原因になる可能性があります
```

### 除外

規則の先頭に`!`を付けると、それ以降のパターンは処理対象から除外されます。これにより、フィルターがブロックリストに変わります。

```
*.*
#^ 注意: まずすべてのテーブルを含めるために *.* を追加する必要があります
!*.Password
!employees.salaries
```

### エスケープ文字

特殊な文字を識別子文字に変えるには、バックスラッシュ（`\`）でそれに続く文字をエスケープします。

```
db\.with\.dots.*
```

簡単さと将来の互換性のため、次のシーケンスは禁止されています:

* 行末の空白がトリミングされた後の `\`（リテラルの空白に一致させるには`[ ]`を使用します）。
* `\`の後にASCII英数字文字（`[0-9a-zA-Z]`）が続く。特に、現在は意味を持たないCライクのエスケープシーケンス（`\0`、`\r`、`\n`、`\t`）。

### クォート識別子

`\`の代わりに、特殊文字は`"`または `` ` ``を使用してクォートすることもできます。

```
"db.with.dots"."tbl\1"
`db.with.dots`.`tbl\2`
```

クォーテーションマークは、ダブルにすることで識別子内に含めることができます。

```
"foo""bar".`foo``bar`
# 以下と同等です:
foo\"bar.foo\`bar
```

クォートされた識別子は複数行にまたがることはできません。

識別子を部分的にクォートすることは無効です:

```
"this is "invalid*.*
```

### 正規表現

非常に複雑なルールが必要な場合は、各パターンを`/`で囲まれた正規表現として記述できます:

```
/^db\d{2,}$/./^tbl\d{2,}$/
```

これらの正規表現は、[Goの方言](https://pkg.go.dev/regexp/syntax?tab=doc)を使用します。パターンは、識別子に正規表現に一致する部分文字列が含まれている場合に一致します。たとえば、`/b/`は `db01` に一致します。

> **注記:**
>
> 正規表現内の各`/`は`\/`とエスケープする必要があり、`[…]`内でもエスケープする必要があります。`\Q…\E`の間にエスケープされていない`/`を置くことはできません。

## 複数のルール

<CustomContent platform="tidb-cloud">

> **注記:**
>
> このセクションはTiDB Cloudには適用されません。現在、TiDB Cloudは1つのテーブルフィルタールールのみをサポートしています。

</CustomContent>

フィルターリスト内のテーブル名がいずれのルールにも一致しない場合、デフォルトの動作はそのような不一致のテーブルを無視することです。

ブロックリストを構築するには、最初のルールとして明示的に`*.*`を使用する必要があります。そうしないと、すべてのテーブルが除外されます。

```bash
# すべてのテーブルがフィルタリングされます
./dumpling -f '!*.Password'

# "Password"テーブルのみがフィルタリングされます。それ以外のテーブルは含まれます。
./dumpling -f '*.*' -f '!*.Password'
```

フィルターリスト内でテーブル名が複数のパターンに一致する場合、最後の一致が結果を決定します。たとえば:

```
# ルール1
employees.*
# ルール2
!*.dep*
# ルール3
*.departments
```

フィルターリスト内でテーブル名が複数のパターンに一致する場合、最後の一致が結果を決定します。たとえば:

| テーブル名            | ルール1 | ルール2 | ルール3 | 結果          |
|-----------------------|--------|--------|--------|------------------|
| irrelevant.table      |        |        |        | デフォルト（拒否） |
| employees.employees   | ✓      |        |        | ルール1(許可)  |
| employees.dept_emp    | ✓      | ✓      |        | ルール2(拒否)  |
| employees.departments | ✓      | ✓      | ✓      | ルール3(許可)  |
| else.departments      |        | ✓      | ✓      | ルール3(許可)  |

> **注意:**
>
> TiDBツールでは、システムスキーマは常にデフォルト構成で除外されます。システムスキーマは以下の通りです:
>
> * `INFORMATION_SCHEMA`
> * `PERFORMANCE_SCHEMA`
> * `METRICS_SCHEMA`
> * `INSPECTION_SCHEMA`
> * `mysql`
> * `sys`