---
title: 読み取り専用ストレージノードのベストプラクティス
summary: 重要なオンラインサービスを物理的に分離するための読み取り専用ストレージノードの設定方法について学びます。

# 読み取り専用ストレージノードのベストプラクティス

この文書では、読み取り専用ストレージノードの設定方法と、バックアップ、解析、テストなどのトラフィックをこれらのノードに向ける方法について紹介します。これにより、遅延に対する高い耐性を持つ負荷を重要なオンラインサービスから物理的に分離できます。

## 手順

### 1. いくつかの TiKV ノードを読み取り専用に指定する

TiKV ノードを読み取り専用に指定するには、これらのノードに特別なラベルを付けます（ラベルキーのプレフィックスとして `$` を使用します）。配置ルールを使用して明示的にこれらのノードにデータを保存するまで、PD はこれらのノードにデータをスケジュールしません。

次のコマンドを実行して読み取り専用ノードを構成できます。

```
tikv_servers:
  - host: ...
    ...
    labels:
      $mode: readonly
```

### 2. プレースメントルールを使用して読み取り専用ノードにデータを learners として保存する

1. `pd-ctl config placement-rules` コマンドを実行してデフォルトのプレースメントルールをエクスポートします。

    ```shell
    pd-ctl config placement-rules rule-bundle load --out="rules.json"
    ```

    プレースメントルールを構成していない場合、出力は次のようになります。

    ```json
    [
      {
        "group_id": "pd",
        "group_index": 0,
        "group_override": false,
        "rules": [
          {
            "group_id": "pd",
            "id": "default",
            "start_key": "",
            "end_key": "",
            "role": "voter",
            "count": 3
          }
        ]
      }
    ]
    ```

2. デフォルト構成に基づいて、すべてのデータを読み取り専用ノードに learner として保存します。次の例はデフォルト構成に基づいています。

    ```json
    [
      {
        "group_id": "pd",
        "group_index": 0,
        "group_override": false,
        "rules": [
          {
            "group_id": "pd",
            "id": "default",
            "start_key": "",
            "end_key": "",
            "role": "voter",
            "count": 3
          },
          {
            "group_id": "pd",
            "id": "readonly",
            "start_key": "",
            "end_key": "",
            "role": "learner",
            "count": 1,
            "label_constraints": [
              {
                "key": "$mode",
                "op": "in",
                "values": [
                  "readonly"
                ]
              }
            ],
            "version": 1
          }
        ]
      }
    ]
    ```

3. 前述の構成を PD に書き込むために `pd-ctl config placement-rules` コマンドを使用します。

    ```shell
    pd-ctl config placement-rules rule-bundle save --in="rules.json"
    ```

> **注意:**
>
> - データセットが大きいクラスタで前述の操作を実行する場合、クラスタ全体でデータを読み取り専用ノードに完全にレプリケートするには時間がかかる場合があります。その間、読み取り専用ノードはサービスを提供できないことがあります。
> - バックアップの特別な実装により、各ラベルの learner 数は 1 を超えることはできません。それ以外の場合、バックアップ中に重複データが生成されます。

### 3. 読み取り専用ノードからデータを読み取るための Follower Read の使用

#### 3.1 TiDB で Follower Read を使用する

TiDB を使用する場合、読み取り専用ノードからデータを読み取るには、システム変数 [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40) を `learner` に設定します。

```sql
set tidb_replica_read=learner;
```

#### 3.2 TiSpark で Follower Read を使用する

TiSpark を使用する場合、Spark 構成ファイルで設定項目 `spark.tispark.replica_read` を `learner` に設定します。

```
spark.tispark.replica_read learner
```

#### 3.3 クラスタデータをバックアップする際の Follower Read の使用

クラスタデータをバックアップする際に読み取り専用ノードからデータを読み取るには、br コマンドラインで `--replica-read-label` オプションを指定します。次のコマンドをシェルで実行する場合は、`$` の解釈を防ぐためにラベルをシングルクォートで囲む必要があります。

```shell
br backup full ... --replica-read-label '$mode:readonly'
```