---
title: sync-diff-inspectorユーザーガイド
summary: sync-diff-inspectorを使用してデータを比較し、不整合なデータを修復します。
aliases: ['/docs/dev/sync-diff-inspector/sync-diff-inspector-overview/','/docs/dev/reference/tools/sync-diff-inspector/overview/']
---

# sync-diff-inspectorユーザーガイド

[sync-diff-inspector](https://github.com/pingcap/tidb-tools/tree/master/sync_diff_inspector)は、データベースに格納されているデータをMySQLプロトコルで比較するためのツールです。たとえば、MySQLのデータとTiDBのデータ、MySQLのデータとMySQLのデータ、またはTiDBのデータとTiDBのデータを比較することができます。さらに、わずかなデータの不整合が発生した場合に、このツールを使用してデータを修復することもできます。

このガイドでは、sync-diff-inspectorの主な機能を紹介し、このツールの構成と使用方法について説明します。sync-diff-inspectorをダウンロードする方法は次のとおりです。

+ バイナリパッケージ。sync-diff-inspectorのバイナリパッケージはTiDB Toolkitに含まれています。TiDB Toolkitをダウンロードするには、[TiDBツールのダウンロード](/download-ecosystem-tools.md)を参照してください。
+ Dockerイメージ。次のコマンドを実行してダウンロードします。

    {{< copyable "shell-regular" >}}

    ```shell
    docker pull pingcap/tidb-tools:latest
    ```

## 主な機能

* テーブルのスキーマとデータを比較する
* データの不整合がある場合に修復に使用されるSQLステートメントを生成する
* [異なるスキーマやテーブル名を持つテーブルのデータチェックをサポート](/sync-diff-inspector/route-diff.md)
* [シャーディングシナリオでのデータチェックをサポート](/sync-diff-inspector/shard-diff.md)
* [TiDBの上流と下流クラスタのデータチェックをサポート](/ticdc/ticdc-upstream-downstream-check.md)
* [DMレプリケーションシナリオでのデータチェックをサポート](/sync-diff-inspector/dm-diff.md)

## sync-diff-inspectorの制限事項

* MySQLとTiDB間のデータ移行に対するオンラインチェックはサポートされていません。上流と下流のチェックリストにデータが書き込まれないようにし、ある範囲内のデータが変更されないようにします。`range`を設定してこの範囲内のデータをチェックできます。

* TiDBおよびMySQLでは、`FLOAT`、`DOUBLE`などの浮動小数点型が異なる実装されています。`FLOAT`および`DOUBLE`はそれぞれ計算のチェックサムに6桁と15桁を使用します。この機能を使用したくない場合は、`ignore-columns`を設定してこれらの列のチェックをスキップします。

* 主キーまたは一意のインデックスを含まないテーブルのチェックをサポートしています。ただし、データが不整合な場合、生成されたSQLステートメントでデータを正しく修復できないことがあります。

## sync-diff-inspectorのデータベース権限

sync-diff-inspectorはテーブルのスキーマ情報を取得し、データをクエリする必要があります。必要なデータベース権限は次のとおりです。

* 上流データベース
    - `SELECT` (比較のためのデータのチェック)
    - `SHOW_DATABASES` (データベース名の表示)
    - `RELOAD` (テーブルスキーマの表示)
* 下流データベース
    - `SELECT` (比較のためのデータのチェック)
    - `SHOW_DATABASES` (データベース名の表示)
    - `RELOAD` (テーブルスキーマの表示)

## 構成ファイルの説明

sync-diff-inspectorの構成は以下の部分で構成されています。

- `グローバル構成`: スレッド数の設定、不整合なテーブルを修復するためのSQLステートメントをエクスポートするかどうか、データの比較を行うかどうか、上流または下流に存在しないテーブルのチェックをスキップするかどうかなどの一般的な構成です。
- `データベース構成`: 上流および下流データベースのインスタンスを構成します。
- `ルート`: 上流の複数のスキーマ名を、下流の単一のスキーマ名に一致させるためのルール **(オプション)**。
- `タスク構成`: チェックするテーブルを構成します。上流と下流のデータベース間に特定のマッピング関係がある場合や、特定の要件がある場合は、これらのテーブルを構成する必要があります。
- `テーブル構成`: 特定のテーブルに対する特別な構成です。たとえば、特定の範囲や無視する列などの特別な要件を設定します。**(オプション)**。

以下は完全な構成ファイルの説明です。

- 注: `s`で名前の後に構成が複数ある場合、その構成値を含むために角括弧`[]`を使用する必要があります。

``` toml
# Diff Configuration.

######################### グローバル構成 #########################
# データをチェックするために作成されたゴールーチンの数。sync-diff-inspectorと上流/下流データベース間の接続数は、わずかにこの値よりも大きくなります。
check-thread-count = 4

# 有効になっている場合、SQLステートメントは不整合なテーブルを修復するためにエクスポートされます。
export-fix-sql = true

# データの比較のみを行い、テーブルの構造のみをチェックします。
check-struct-only = false

# 有効になっている場合、sync-diff-inspectorは上流または下流に存在しないテーブルのチェックをスキップします。
skip-non-existing-table = false

######################### データソース構成 #########################
[data-sources]
[data-sources.mysql1] # mysql1はデータベースインスタンスの唯一のカスタムIDです。次の`task.source-instances/task.target-instance`構成に使用されます。
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""  # 上流データベースに接続するためのパスワード。プレインテキストまたはBase64でエンコードされていることがあります。

    # (オプション)複数の上流シャードテーブルをマッチングするためのマッピングルールを使用します。Rule1およびrule2は次のルートセクションで構成されています。
    route-rules = ["rule1", "rule2"]

[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = ""  # 下流データベースに接続するためのパスワード。プレインテキストまたはBase64でエンコードされていることがあります。

    # (オプション)TLSを使用してTiDBに接続します。
    # security.ca-path = ".../ca.crt"
    # security.cert-path = ".../cert.crt"
    # security.key-path = ".../key.crt"

    # (オプション)スナップショット機能を使用します。有効になっている場合、履歴データが比較に使用されます。
    # snapshot = "386902609362944000"
    # "snapshot"を"auto"に設定すると、上流および下流でTiCDCによって生成された最後のシンクポイントが比較に使用されます。詳細については、<https://github.com/pingcap/tidb-tools/issues/663>を参照してください。
    # snapshot = "auto"

########################### ルート ##############################
# 多数の異なるスキーマ名やテーブル名を持つテーブルのデータを比較したり、複数の上流のシャードテーブルと下流のテーブルファミリーのデータをチェックしたりする場合は、table-ruleを使用してマッピング関係を構成します。スキーマまたはテーブルのいずれかのマッピングルールのみを構成することもできます。また、スキーマとテーブルの両方のマッピングルールを構成することもできます。
[routes]
[routes.rule1] # rule1は構成のための唯一のカスタムIDです。上記の`data-sources.route-rules`構成に使用されます。
schema-pattern = "test_*"      # データソースのスキーマ名に一致します。ワイルドカード"*"と"?"がサポートされています。
table-pattern = "t_*"          # データソースのテーブル名に一致します。ワイルドカード"*"と"?"がサポートされています。
target-schema = "test"         # ターゲットデータベース内のスキーマ名
target-table = "t"             # ターゲットテーブルの名前
[routes.rule2]
schema-pattern = "test2_*"      # データソースのスキーマ名に一致します。ワイルドカード"*"と"?"がサポートされています。
table-pattern = "t2_*"          # データソースのテーブル名に一致します。ワイルドカード"*"と"?"がサポートされています。
target-schema = "test2"         # ターゲットデータベース内のスキーマ名
target-table = "t2"             # ターゲットテーブルの名前

######################### タスク構成 #########################
# 比較する必要がある下流データベースのテーブルを構成します。
[task]
    # output-dirは次の情報を保存します：
    # 1 sql: エラーが検出された後に生成されたテーブルを修復するためのSQLファイル。1つのチャンクに1つのSQLファイルが対応します。
    # 2 log: sync-diff.log
    # 3 summary: summary.txt
    # 4 checkpoint: ディレクトリ
    output-dir = "./output"
    # 上流データベース。値はdata-sourcesで宣言された一意のIDです。
    source-instances = ["mysql1"]
    # 下流データベース。値はdata-sourcesで宣言された一意のIDです。
    target-instance = "tidb0"
    # 比較する下流データベースのテーブル。各テーブルには、スキーマ名とテーブル名が"."で区切られている必要があります。
    # "?"を使用して任意の文字と"*"を使用して任意の長さの文字をマッチさせます。
    # 詳細なマッチルールについては、golang regexp pkg: https://github.com/google/re2/wiki/Syntax を参照してください。
    target-check-tables = ["schema*.table*", "!c.*", "test2.t2"]
    # (オプション)一部のテーブルの追加構成、Config1は次のテーブル構成の例で定義されています。
    target-configs = ["config1"]

######################### テーブル構成 #########################
# 特定のテーブルの特別な構成。構成するテーブルは`task.target-check-tables`に含まれている必要があります。
[table-configs.config1] # config1はこの構成のための唯一のカスタムIDです。上記の`task.target-configs`構成に使用されます。
# ターゲットテーブルの名前、複数のテーブルを一致させるために正規表現を使用できますが、一つのテーブルが複数の特別な構成で一度に一致することはできません。
target-tables = ["schema*.test*", "test2.t2"]
# (任意) チェックするデータの範囲を指定します
# SQLのWHERE句の構文に準拠する必要があります。
range = "age > 10 AND age < 20"
# (任意) データをチャンクに分割するために使用される列を指定します。構成しない場合、
# sync-diff-inspectorは適切な列（プライマリキー、ユニークキー、またはインデックス付きのフィールド）を選択します。
index-fields = ["col1","col2"]
# (任意) 現在sync-diff-inspectorがサポートしていないいくつかの列（json、bit、blobなど）のようないくつかの列のチェックを無視します。
# 浮動小数点データ型はTiDBとMySQLで異なる動作を示します。これらの列のチェックをスキップするために`ignore-columns`を使用できます。
ignore-columns = ["",""]
# (任意) テーブルを分割するためのチャンクのサイズを指定します。指定されていない場合、この構成は削除するか、0で設定できます。
chunk-size = 0
# (任意) テーブルの「照合（collation）」を指定します。指定されていない場合、この構成は削除するか、空の文字列に設定できます。
collation = ""
```

## sync-diff-inspectorの実行

以下のコマンドを実行します：

{{< copyable "shell-regular" >}}

```bash
./sync_diff_inspector --config=./config.toml
```

このコマンドは`config.toml`の`output-dir`に`summary.txt`とログ`sync_diff.log`を出力し、`output-dir`内に`config.toml`ファイルのハッシュ値で名前付けられたフォルダーが生成されます。このフォルダには、チェックポイントノード情報とデータが不整合の時に生成されるSQLファイルが含まれます。

### 進捗情報

sync-diff-inspectorは実行時に`stdout`に進捗情報を送信します。進捗情報には、テーブル構造の比較結果、テーブルデータの比較結果、および進捗バーが含まれます。

> **注意:**
>
> 表示ウィンドウの幅を80文字以上に保つようにしてください。

```progress
比較するテーブルの合計は2です

「sbtest」「sbtest96」のテーブル構造を比較中...等しい
「sbtest」「sbtest99」のテーブル構造を比較中...等しい
「sbtest」「sbtest96」のテーブルデータを比較中...失敗
「sbtest」「sbtest99」のテーブルデータを比較中...
_____________________________________________________________________________
進捗 [==========================================================>--] 98% 193/200
```

```progress
比較するテーブルの合計は2です

「sbtest」「sbtest96」のテーブル構造を比較中...等しい
「sbtest」「sbtest99」のテーブル構造を比較中...等しい
「sbtest」「sbtest96」のテーブルデータを比較中...失敗
「sbtest」「sbtest99」のテーブルデータを比較中...失敗
_____________________________________________________________________________
進捗 [============================================================>] 100% 0/0
`sbtest`.`sbtest99`のデータは一致していません
`sbtest`.`sbtest96`のデータは一致していません

残りのテーブルは全て一致しています。

合計2つのテーブルが比較され、0つのテーブルが完了し、2つのテーブルが失敗し、0つのテーブルがスキップされました。
パッチファイルは次の場所に生成されます
        'output/fix-on-tidb2/'
比較の詳細は 'output/sync_diff.log' で確認できます
```

### 出力ファイル

出力ファイルのディレクトリ構造は以下のとおりです：

```
output/
|-- checkpoint # ブレークポイント情報を保存
| |-- bbfec8cc8d1f58a5800e63aa73e5 # 構成のハッシュ値。出力ディレクトリ（output/）に対応する構成ファイルを識別するプレースホルダーファイル
、│ │ DO_NOT_EDIT_THIS_DIR
│ └-- sync_diff_checkpoints.pb # ブレークポイント情報
|
|-- fix-on-target # データ不整合を修正するためのSQLファイルを保存
| |-- xxx.sql
| |-- xxx.sql
| └-- xxx.sql
|
|-- summary.txt # チェック結果のサマリーを保存
└-- sync_diff.log # sync-diff-inspectorの実行時の出力ログ情報を保存
```

### ログ

sync-diff-inspectorのログは`${output}/sync_diff.log`に保存されます。ここで`${output}`は`config.toml`ファイルの`output-dir`の値です。

### 進捗

実行中のsync-diff-inspectorは、定期的に（10秒ごとに）チェックポイントで進捗を表示し、 `${output}/checkpoint/sync_diff_checkpoints.pb`にあり、`${output}`は`config.toml`ファイルの`output-dir`の値です。

### 結果

チェックが完了すると、sync-diff-inspectorはレポートを出力します。レポートは `${output}/summary.txt`にあり、`${output}`は`config.toml`ファイルの`output-dir`の値です。

```summary
+---------------------+--------------------+----------------+---------+-----------+
|        TABLE        | STRUCTURE EQUALITY | DATA DIFF ROWS | UPCOUNT | DOWNCOUNT |
+---------------------+--------------------+----------------+---------+-----------+
| `sbtest`.`sbtest99` | true               | +97/-97        |  999999 |    999999 |
| `sbtest`.`sbtest96` | true               | +0/-101        |  999999 |   1000100 |
+---------------------+--------------------+----------------+---------+-----------+
Time Cost: 16.75370462s
Average Speed: 113.277149MB/s
```

- `TABLE`: 対応するデータベース名とテーブル名
- `RESULT`: チェックが完了したかどうか。`skip-non-existing-table = true`を構成した場合、この列の値は、上流または下流に存在しないテーブルに対して`skipped`になります。
- `STRUCTURE EQUALITY`: テーブル構造が同じかどうかをチェックします
- `DATA DIFF ROWS`: `rowAdd`/`rowDelete`。テーブルを修正するために追加/削除する行の数を示します

### データの不整合を修正するSQLステートメント

データのチェック中に異なる行が存在する場合、それらを修正するためのSQLステートメントが生成されます。データの不整合がチャンクに存在する場合、`chunk.Index`という名前のSQLファイルが生成されます。SQLファイルの場所は`${output}/fix-on-${instance}`であり、`${instance}`は`config.toml`ファイルの`task.target-instance`の値です。

SQLファイルには、チャンクが属するテーブルと範囲情報が含まれています。SQLファイルでは、次の3つの状況を考慮する必要があります。

- 下流のデータベースの行が欠落している場合、置換（REPLACE）ステートメントが適用されます
- 下流のデータベースの行が余分な場合、削除（DELETE）ステートメントが適用されます
- 下流のデータベースの行の一部のデータが不整合の場合、置換（REPLACE）ステートメントが適用され、不整合の列はSQLファイルに注釈付きでマークされます

```sql
-- テーブル: sbtest.sbtest99
-- 範囲: (3690708) < (id) <= (3720581)
/*
  不整合カラム ╏   `K`   ╏                `C`                 ╏               `PAD`
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
```sql
      + {R}
      + {R}
    + {R}
  + {R}
```

```sql
      + {T}
      + {T}
    + {T}
  + {T}
```