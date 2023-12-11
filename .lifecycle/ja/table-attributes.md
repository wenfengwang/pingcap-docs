---
title: テーブル属性
summary: TiDBのテーブル属性機能の使用方法を学びます。

# テーブル属性

TiDB v5.3.0 でテーブル属性機能が導入されました。この機能を使用すると、テーブルやパーティションに特定の属性を追加して、それに対応する操作を行うことができます。たとえば、テーブル属性を使用して、Regionのマージ動作を制御することができます。

<CustomContent platform="tidb">

現在、TiDBはテーブルやパーティションにのみ `merge_option` 属性を追加して、Regionのマージ動作を制御することができます。`merge_option` 属性はホットスポットの対応の一部です。詳細については、[ホットスポットの問題のトラブルシューティング](/troubleshoot-hot-spot-issues.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

現在、TiDBはテーブルやパーティションにのみ `merge_option` 属性を追加して、Regionのマージ動作を制御することができます。`merge_option` 属性はホットスポットの対応の一部です。

</CustomContent>

> **注意:**
>
> - TiDB BinlogまたはTiCDCを使用してレプリケーションを行うか、BRを使用して増分バックアップを行う場合、レプリケーションまたはバックアップ操作は、テーブル属性を設定するDDLステートメントをスキップします。下流またはバックアップクラスタでテーブル属性を使用するには、下流またはバックアップクラスタでDDLステートメントを手動で実行する必要があります。

## 使用法

テーブル属性は `key=value` の形式です。複数の属性はカンマで区切られます。以下は使用法の例です。`t` は変更するテーブルの名前で、`p` は変更するパーティションの名前です。`[]` 内のアイテムはオプションです。

+ テーブルやパーティションに属性を設定する:

    ```sql
    ALTER TABLE t [PARTITION p] ATTRIBUTES [=] 'key=value[, key1=value1...]';
    ```

+ テーブルやパーティションの属性をリセットする:

    ```sql
    ALTER TABLE t [PARTITION p] ATTRIBUTES [=] DEFAULT;
    ```

+ すべてのテーブルやパーティションの属性を表示する:

    ```sql
    SELECT * FROM information_schema.attributes;
    ```

+ テーブルやパーティションに構成された属性を表示する:

    ```sql
    SELECT * FROM information_schema.attributes WHERE id='schema/t[/p]';
    ```

+ 特定の属性を持つすべてのテーブルやパーティションを表示する:

    ```sql
    SELECT * FROM information_schema.attributes WHERE attributes LIKE '%key%';
    ```

## 属性の上書きルール

テーブルに構成された属性は、テーブルのすべてのパーティションに影響を与えます。ただし、1つ例外があります。テーブルとパーティションが同じ属性を構成しても、属性値が異なる場合、パーティションの属性がテーブルの属性を上書きします。たとえば、テーブル `t` に `key=value` 属性が構成されており、パーティション `p` に `key=value1` が構成されているとします。

```sql
ALTER TABLE t ATTRIBUTES[=]'key=value';
ALTER TABLE t PARTITION p ATTRIBUTES[=]'key=value1';
```

この場合、`key=value1` が実際に `p1` パーティションに影響します。

## テーブル属性を使用してRegionのマージ動作を制御する

### ユーザー・シナリオ

書き込みホットスポットまたは読み込みホットスポットがある場合、テーブル属性を使用してRegionのマージ動作を制御することができます。最初に `merge_option` 属性をテーブルやパーティションに追加し、その値を `deny` に設定できます。2つのシナリオは次のとおりです。

#### 新しく作成されたテーブルまたはパーティションの書き込みホットスポット

新しく作成されたテーブルやパーティションにデータが書き込まれるとホットスポットの問題が発生した場合、通常はRegionを分割して散らす必要があります。ただし、分割/散布操作と書き込みの間に一定の時間間隔が存在する場合、これらの操作は書き込みホットスポットを実際に回避することができません。なぜなら、テーブルやパーティションが作成されるときに行われる分割操作によって空のRegionが生成されるため、時間間隔が存在する場合は分割されたRegionがマージされる可能性があります。この場合、`merge_option` 属性をテーブルやパーティションに追加し、その属性値を `deny` に設定できます。

#### 読み取り専用シナリオでの定期的な読み込みホットスポット

読み取り専用のシナリオで、定期的な読み込みホットスポットを手動で分割されたRegionで緩和し、読み込みホットスポットが解消された後に手動で分割されたRegionがマージされるのを避けたい場合、`merge_option` 属性をテーブルやパーティションに追加し、その値を `deny` に設定できます。

### 使用法

+ テーブルのRegionのマージを防ぐ:

    ```sql
    ALTER TABLE t ATTRIBUTES 'merge_option=deny';
    ```

+ テーブルに属するRegionのマージを許可:

    ```sql
    ALTER TABLE t ATTRIBUTES 'merge_option=allow';
    ```

+ テーブルの属性をリセットする:

    ```sql
    ALTER TABLE t ATTRIBUTES DEFAULT;
    ```

+ パーティションのRegionのマージを防ぐ:

    ```sql
    ALTER TABLE t PARTITION p ATTRIBUTES 'merge_option=deny';
    ```

+ パーティションに属するRegionのマージを許可:

    ```sql
    ALTER TABLE t PARTITION p ATTRIBUTES 'merge_option=allow';
    ```

+ `merge_option` 属性が構成されたすべてのテーブルまたはパーティションを表示する:

    ```sql
    SELECT * FROM information_schema.attributes WHERE attributes LIKE '%merge_option%';
    ```

### 属性の上書きルール

```sql
ALTER TABLE t ATTRIBUTES 'merge_option=deny';
ALTER TABLE t PARTITION p ATTRIBUTES 'merge_option=allow';
```

上記の2つの属性を同時に構成すると、パーティション `p` に属するRegionを実際にマージできます。パーティションの属性をリセットすると、パーティション `p` はテーブル `t` から属性を継承し、Regionをマージすることはできません。

<CustomContent platform="tidb">

> **注意:**
>
> - パーティションを持つテーブルの場合、テーブルレベルで `merge_option` 属性が構成されている場合、`merge_option=allow` であっても、通常実際のパーティション数に応じてデフォルトでテーブルは複数のRegionに分割されます。すべてのRegionをマージするには、[テーブルの属性をリセット](#usage)する必要があります。
> - `merge_option` 属性を使用する場合、PD構成パラメータ [`split-merge-interval`](/pd-configuration-file.md#split-merge-interval) に注意する必要があります。`merge_option` 属性が構成されていない場合、Regionが条件を満たすと、`split-merge-interval` で指定された間隔後にRegionがマージされます。`merge_option` 属性が構成されている場合、PDは`merge_option` の構成に従って、指定された間隔後にRegionをマージするかどうかを判断します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> - パーティションを持つテーブルの場合、テーブルレベルで `merge_option` 属性が構成されている場合、`merge_option=allow` であっても、通常実際のパーティション数に応じてデフォルトでテーブルは複数のRegionに分割されます。すべてのRegionをマージするには、[テーブルの属性をリセット](#usage)する必要があります。
> - `merge_option` 属性が構成されていない場合、一定時間後にRegionが条件を満たすと、Regionがマージされます。`merge_option` 属性が構成されている場合、PDは`merge_option` の構成に従って1時間後にRegionをマージするかどうかを判断します。

</CustomContent>