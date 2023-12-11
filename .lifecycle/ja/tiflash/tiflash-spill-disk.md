---
title: TiFlashディスクへのデータスピル
summary: TiFlashがデータをディスクにスピルする方法とスピル動作のカスタマイズ方法について学びます。
---

# TiFlashディスクへのデータスピル

このドキュメントでは、TiFlashが計算中にデータをディスクにスピルする方法について紹介します。

v7.0.0以降、TiFlashはメモリの圧力を軽減するために中間データをディスクにスピルすることをサポートしています。以下の演算子がサポートされています:

* 等結合条件を持つハッシュ結合演算子
* `GROUP BY`キーを持つハッシュ集約演算子
* TopN演算子、およびWindow関数内のソート演算子

## スピルのトリガー

TiFlashでは、データをディスクにスピルするための2つのトリガーメカニズムが提供されています。

* 演算子レベルのスピル: 各演算子のデータスピル閾値を指定することで、その演算子のデータをTiFlashがディスクにスピルするタイミングを制御できます。
* クエリレベルのスピル: TiFlashノード上のクエリの最大メモリ使用量とスピルのためのメモリ比率を指定することで、クエリ内のサポートされている演算子のデータを必要に応じてディスクにスピルするタイミングを制御できます。

### 演算子レベルのスピル

v7.0.0以降、TiFlashは演算子レベルでの自動スピルをサポートしています。以下のシステム変数を使用して、各演算子のデータスピルの閾値を制御できます。演算子のメモリ使用量が閾値を超えると、TiFlashはその演算子のためにスピルをトリガーします。

* [`tidb_max_bytes_before_tiflash_external_group_by`](/system-variables.md#tidb_max_bytes_before_tiflash_external_group_by-new-in-v700)
* [`tidb_max_bytes_before_tiflash_external_join`](/system-variables.md#tidb_max_bytes_before_tiflash_external_join-new-in-v700)
* [`tidb_max_bytes_before_tiflash_external_sort`](/system-variables.md#tidb_max_bytes_before_tiflash_external_sort-new-in-v700)

#### 例

この例では、`GROUP BY`キーを持つハッシュ集約演算子のメモリ使用量が多いSQLステートメントを構築し、スピルの動作を実証します。

1. 環境を準備します。2つのノードを持つTiFlashクラスタを作成し、TPCH-100データをインポートします。
2. 次のステートメントを実行します。これらのステートメントは、`GROUP BY`キーを持つハッシュ集約演算子のメモリ使用量を制限しません。

    ```sql
    SET tidb_max_bytes_before_tiflash_external_group_by = 0;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

3. TiFlashのログから、クエリが単一のTiFlashノードで29.55 GiBのメモリを消費する必要があることがわかります:

    ```
    [DEBUG] [MemoryTracker.cpp:69] ["Peak memory usage (total): 29.55 GiB."] [source=MemoryTracker] [thread_id=468]
    ```

4. 次のステートメントを実行します。これらのステートメントは、`GROUP BY`キーを持つハッシュ集約演算子のメモリ使用量を10737418240 (10 GiB)に制限します。

    ```sql
    SET tidb_max_bytes_before_tiflash_external_group_by = 10737418240;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

5. TiFlashのログから、`tidb_max_bytes_before_tiflash_external_group_by`を構成することで、TiFlashが中間結果をスピルし、クエリで使用されるメモリ量を著しく減らすことがわかります。

    ```
    [DEBUG] [MemoryTracker.cpp:69] ["Peak memory usage (total): 12.80 GiB."] [source=MemoryTracker] [thread_id=110]
    ```

### クエリレベルのスピル

v7.4.0以降、TiFlashはクエリレベルでの自動スピルをサポートしています。以下のシステム変数を使用して、この機能を制御できます:

* [`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740): TiFlashノード上のクエリの最大メモリ使用量を制限します。
* [`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740): データスピルをトリガーするメモリ比率を制御します。

`tiflash_mem_quota_query_per_node`と`tiflash_query_spill_ratio`の両方が0より大きい値に設定されている場合、TiFlashはクエリのメモリ使用量が`tiflash_mem_quota_query_per_node * tiflash_query_spill_ratio`を超えたときに、クエリ内のサポートされている演算子のために自動的にスピルをトリガーします。

#### 例

この例では、クエリレベルのスピルを実証するために、多くのメモリを消費するSQLステートメントを構築します。

1. 環境を準備します。2つのノードを持つTiFlashクラスタを作成し、TPCH-100データをインポートします。

2. 次のステートメントを実行します。これらのステートメントは、クエリまたは`GROUP BY`キーを持つハッシュ集約演算子のメモリ使用量を制限しません。

    ```sql
    SET tidb_max_bytes_before_tiflash_external_group_by = 0;
    SET tiflash_mem_quota_query_per_node = 0;
    SET tiflash_query_spill_ratio = 0;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

3. TiFlashのログから、クエリが単一のTiFlashノードで29.55 GiBのメモリを消費することがわかります:

    ```
    [DEBUG] [MemoryTracker.cpp:69] ["Peak memory usage (total): 29.55 GiB."] [source=MemoryTracker] [thread_id=468]
    ```

4. 次のステートメントを実行します。これらのステートメントは、TiFlashノード上のクエリの最大メモリ使用量を5 GiBに制限します。

    ```sql
    SET tiflash_mem_quota_query_per_node = 5368709120;
    SET tiflash_query_spill_ratio = 0.7;
    SELECT
      l_orderkey,
      MAX(L_COMMENT),
      MAX(L_SHIPMODE),
      MAX(L_SHIPINSTRUCT),
      MAX(L_SHIPDATE),
      MAX(L_EXTENDEDPRICE)
    FROM lineitem
    GROUP BY l_orderkey
    HAVING SUM(l_quantity) > 314;
    ```

5. TiFlashのログから、クエリレベルのスピルを構成することで、TiFlashが中間結果をスピルし、クエリで使用されるメモリ量を著しく減らすことがわかります。

    ```
    [DEBUG] [MemoryTracker.cpp:101] ["Peak memory usage (for query): 3.94 GiB."] [source=MemoryTracker] [thread_id=1547]
    ```

## 注記

* ハッシュ集約演算子に`GROUP BY`キーがない場合、スピルはサポートされません。ハッシュ集約演算子にユニークな集約関数が含まれていても、スピルはサポートされません。
* 演算子レベルのスピルの閾値は、現在それぞれの演算子ごとに計算されます。2つのハッシュ集約演算子が含まれるクエリでは、クエリレベルのスピルが構成されておらず、集約演算子の閾値が10 GiBに設定されている場合、2つのハッシュ集約演算子はそれぞれのメモリ使用量が10 GiBを超えたときのみデータをスピルします。
* 現時点では、ハッシュ集約演算子とTopN/Sort演算子はリストア段階でマージ集約とマージソートアルゴリズムを使用します。そのため、これらの演算子は1ラウンドのスピルのみをトリガーします。メモリの需要が非常に高く、リストア段階でのメモリ使用量が閾値を超えても、それ以上のスピルはトリガーされません。
* ハッシュ結合演算子はパーティションベースのスピル戦略を使用します。リストア段階でのメモリ使用量が閾値を超えても、スピルが再度トリガーされます。ただし、スピルのスケールを制御するため、スピルのラウンド数は3回に制限されます。3回のスピル後にもリストア段階でのメモリ使用量が閾値を超えている場合、スピルは再度トリガーされません。
* クエリレベルのスピルが構成されている場合(つまり、[`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740)と[`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740)の両方が0より大きい値に設定されている場合)、TiFlashは個々の演算子のスピル閾値を無視し、クエリレベルのスピル閾値に基づいてクエリ内の関連する演算子のために自動的にスピルをトリガーします。
* クエリレベルのスピルが構成されている場合でも、クエリで使用される演算子のいずれもがスピルをサポートしていない場合、そのクエリの中間計算結果は依然としてディスクにはスピルできません。この場合、クエリのメモリ使用量が関連する閾値を超えると、TiFlashはエラーを返し、クエリを中止します。
* クエリレベルのスピルが構成されている場合でも、クエリにスピルをサポートする演算子が含まれている場合でも、次のいずれかのシナリオでクエリがエラーを返すことがあります:
    - クエリ内のスピルをサポートしていない他の演算子が多くのメモリを消費した場合。
    - スピル演算子が適切なタイミングでディスクにスピルしなかった場合。
```markdown
To address situations where spilling operators do not spill to disk in time, you can try reducing [`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740) to avoid memory threshold errors.
```

```markdown
時間内にディスクにスパイルしないことがある場合は、[`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740) を減らしてメモリ閾値エラーを回避することができます。
```