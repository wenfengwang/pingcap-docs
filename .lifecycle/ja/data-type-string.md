---
title: 文字列の種類
summary: TiDBでサポートされている文字列の種類について学びます。
aliases: ['/docs/dev/data-type-string/','/docs/dev/reference/sql/data-types/string/']
---

# 文字列の種類

TiDBは、`CHAR`、`VARCHAR`、`BINARY`、`VARBINARY`、`BLOB`、`TEXT`、`ENUM`、および`SET`を含むすべてのMySQLの文字列型をサポートしています。詳細については、[MySQLの文字列の種類](https://dev.mysql.com/doc/refman/8.0/en/string-types.html)を参照してください。

## サポートされている種類

### `CHAR`型

`CHAR`は固定長の文字列です。Mは文字数（バイトではなく）での列長を表します。Mの範囲は0から255です。`CHAR`列にデータを挿入する際、末尾のスペースは切り捨てられます。`VARCHAR`型と異なります。

```sql
[NATIONAL] CHAR[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]
```

### `VARCHAR`型

`VARCHAR`は可変長の文字列です。Mは文字数（バイトではなく）での最大列長を表します。`VARCHAR`の最大サイズは65,535バイトを超えることはできません。最大行長と使用されている文字セットによって`VARCHAR`の長さが決まります。

1文字あたりの占有するスペースは、異なる文字セットについて異なる場合があります。次の表には、1文字あたりの消費バイト数と各文字セットにおける`VARCHAR`列長の範囲が示されています。

| 文字セット | 1文字あたりのバイト数 | 最大`VARCHAR`列長の範囲 |
| ----- | ---- | ---- |
| ascii | 1 | (0, 65535] |
| latin1 | 1 | (0, 65535] |
| binary | 1 | (0, 65535] |
| utf8 | 3 | (0, 21845] |
| utf8mb4 | 4 | (0, 16383] |

```sql
[NATIONAL] VARCHAR(M) [CHARACTER SET charset_name] [COLLATE collation_name]
```

### `TEXT`型

`TEXT`は可変長の文字列です。最大列長は65,535バイトです。オプションのM引数は文字数であり、`TEXT`列の最適な型を自動的に選択するために使用されます。たとえば、`TEXT(60)`は、最大255バイトまで保持できる`TINYTEXT`データ型を生成し、これは最大60文字のUTF-8文字列に適合し、1文字あたり最大4バイト（4×60=240）を持ちます。M引数の使用は推奨されません。

```sql
TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]
```

### `TINYTEXT`型

`TINYTEXT`型は[`TEXT`型](#text-type)に類似しています。違いは、`TINYTEXT`の最大列長が255であることです。

```sql
TINYTEXT [CHARACTER SET charset_name] [COLLATE collation_name]
```

### `MEDIUMTEXT`型

<CustomContent platform="tidb">

`MEDIUMTEXT`型は[`TEXT`型](#text-type)に類似しています。違いは、`MEDIUMTEXT`の最大列長が16,777,215であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>
<CustomContent platform="tidb-cloud">

`MEDIUMTEXT`型は[`TEXT`型](#text-type)に類似しています。違いは、`MEDIUMTEXT`の最大列長が16,777,215であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>

```sql
MEDIUMTEXT [CHARACTER SET charset_name] [COLLATE collation_name]
```

### `LONGTEXT`型

<CustomContent platform="tidb">

`LONGTEXT`型は[`TEXT`型](#text-type)に類似しています。違いは、`LONGTEXT`の最大列長が4,294,967,295であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>
<CustomContent platform="tidb-cloud">

`LONGTEXT`型は[`TEXT`型](#text-type)に類似しています。違いは、`LONGTEXT`の最大列長が4,294,967,295であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>

```sql
LONGTEXT [CHARACTER SET charset_name] [COLLATE collation_name]
```

### `BINARY`型

`BINARY`型は[`CHAR`型](#char-type)に類似しています。違いは、`BINARY`はバイナリバイトストリングを格納することです。

```sql
BINARY(M)
```

### `VARBINARY`型

`VARBINARY`型は[`VARCHAR`型](#varchar-type)に類似しています。違いは、`VARBINARY`はバイナリバイトストリングを格納することです。

```sql
VARBINARY(M)
```

### `BLOB`型

`BLOB`は大きなバイナリファイルです。Mはバイトでの最大列長を表し、0から65,535までの範囲です。

```sql
BLOB[(M)]
```

### `TINYBLOB`型

`TINYBLOB`型は[`BLOB`型](#blob-type)に類似しています。違いは、`TINYBLOB`の最大列長が255であることです。

```sql
TINYBLOB
```

### `MEDIUMBLOB`型

<CustomContent platform="tidb">

`MEDIUMBLOB`型は[`BLOB`型](#blob-type)に類似しています。違いは、`MEDIUMBLOB`の最大列長が16,777,215であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>
<CustomContent platform="tidb-cloud">

`MEDIUMBLOB`型は[`BLOB`型](#blob-type)に類似しています。違いは、`MEDIUMBLOB`の最大列長が16,777,215であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>

```sql
MEDIUMBLOB
```

### `LONGBLOB`型

<CustomContent platform="tidb">

`LONGBLOB`型は[`BLOB`型](#blob-type)に類似しています。違いは、`LONGBLOB`の最大列長が4,294,967,295であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>
<CustomContent platform="tidb-cloud">

`LONGBLOB`型は[`BLOB`型](#blob-type)に類似しています。違いは、`LONGBLOB`の最大列長が4,294,967,295であることです。ただし、デフォルトではTiDBの1行における最大ストレージサイズは6 MiBであり、設定を変更することで最大120 MiBまで増やすことができます。

</CustomContent>

```sql
LONGBLOB
```

### `ENUM`型

`ENUM`は、テーブルが作成された際に列の定義で明示的に列挙された許可される値リストから選択される値を持つ文字列オブジェクトです。構文は以下の通りです。

```sql
ENUM('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

たとえば：
ENUM（「りんご」、「オレンジ」、「梨」）

「ENUM」データ型の値は数字として格納されます。 各値は定義順に従って番号に変換されます。前述の例では、それぞれの文字列が番号にマップされます。

| 値 | 番号 |
| ---- | ---- |
| NULL | NULL |
| '' | 0 |
| 'りんご' | 1 |
| 'オレンジ' | 2 |
| '梨' | 3 |

詳細については、[MySQLのENUM型](https://dev.mysql.com/doc/refman/8.0/en/enum.html)を参照してください。

### `SET` タイプ

`SET` は、テーブルが作成されるときに指定された許可された値のリストから選択する必要がある、ゼロ個以上の値を持つ文字列オブジェクトです。 構文は次のとおりです。

```sql
SET('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

たとえば：
SET（'1'、 '2'） NOT NULL

この例では、次のいずれかの値が有効です。

```
''
'1'
'2'
'1,2'
```

TiDBでは、`SET` タイプの値は内部的に `Int64` に変換されます。各要素の存在は、バイナリを使用して表されます: 0または1。 `SET（'a'、'b'、'c'、'd'）` と指定された列の場合、メンバーには次の10進およびバイナリ値があります。

| メンバー | 10進数の値 |  2進数の値 |
| ---- | ---- | ------ |
| 'a' | 1 | 0001 |
| 'b' | 2 | 0010 |
| 'c' | 4 | 0100 |
| 'd' | 8 | 1000 |

この場合、 `（'a'、 'c'）` の要素は、バイナリで `0101` です。

詳細については、[MySQLのSET型](https://dev.mysql.com/doc/refman/8.0/en/set.html)を参照してください。