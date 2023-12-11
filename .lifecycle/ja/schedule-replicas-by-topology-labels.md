---
title: トポロジーラベルによるレプリカのスケジュール
summary: トポロジーラベルを使用してレプリカをスケジュールする方法について学びます。
aliases: ['/docs/dev/location-awareness/','/docs/dev/how-to/deploy/geographic-redundancy/location-awareness/','/tidb/dev/location-awareness']
---

# トポロジーラベルによるレプリカのスケジュール

> **注意:**
>
> TiDB v5.3.0 では [SQL での配置ルール](/placement-rules-in-sql.md)が導入されました。これにより、テーブルやパーティションの配置をより便利に構成できます。SQL での配置ルールは将来のリリースで PD を介した配置設定の取って代わることがあります。

TiDB クラスタの高可用性および災害復旧機能を向上させるためには、TiKV ノードができる限り物理的に分散していることが推奨されます。たとえば、TiKV ノードは異なるラックや異なるデータセンターに分散配置されることがあります。TiKV のトポロジ情報に基づき、PD スケジューラは各リージョンのレプリカをできるだけ分離し、災害復旧の能力を最大限に引き出すためバックグラウンドでスケジューリングを自動的に実行します。

このメカニズムを効果的にするためには、クラスタのトポロジ情報、特に TiKV の位置情報がデプロイ中に PD に報告されるように TiKV および PD を適切に構成する必要があります。始める前に、まず[ TiUP を使用して TiDB をデプロイ](/production-deployment-using-tiup.md)することをお勧めします。

## クラスタトポロジに基づいて `labels` を構成する

### TiKV および TiFlash のために `labels` を構成する

コマンドラインフラグを使用するか、TiKV または TiFlash の構成ファイルを使用してキーと値のペアの形式でいくつかの属性をバインドすることができます。これらの属性は `labels` と呼ばれます。TiKV と TiFlash が起動した後、それらの `labels` が PD に報告され、ユーザーは TiKV と TiFlash ノードの位置を特定することができます。

例えば、トポロジにはゾーン > データセンター（dc） > ラック > ホストの 4 つの階層があるとします。そして、これらのラベル（zone、dc、rack、host）を使用して TiKV と TiFlash の位置を設定できます。TiKV と TiFlash のラベルを設定するには、次の方法のいずれかを使用できます。

+ TiKV インスタンスを起動する際にコマンドラインフラグを使用する:

    ```shell
    tikv-server --labels zone=<zone>,dc=<dc>,rack=<rack>,host=<host>
    ```

+ TiKV の構成ファイルで設定:

    ```toml
    [server]
    [server.labels]
    zone = "<zone>"
    dc = "<dc>"
    rack = "<rack>"
    host = "<host>"
    ```

TiFlash の `labels` を設定するには、tiflash-proxy の設定ファイルである `tiflash-learner.toml` ファイルを使用できます:

```toml
[server]
[server.labels]
zone = "<zone>"
dc = "<dc>"
rack = "<rack>"
host = "<host>"
```

### (オプション) TiDB に `labels` を構成する

[Follower read](/follower-read.md) が有効になっている場合、TiDB が同じリージョンからデータを読み取ることを希望する場合、TiDB ノードに `labels` を構成する必要があります。

TiDB の `labels` を構成するには、構成ファイルを使用できます:

```toml
[labels]
zone = "<zone>"
dc = "<dc>"
rack = "<rack>"
host = "<host>"
```

> **注意:**
>
> 現在、TiDB は同じリージョンにあるレプリカを一致させるために `zone` ラベルに依存しています。この機能を使用するには、[PD の `location-labels` を構成](#configure-location-labels-for-pd)する際に `zone` を含め、TiDB、TiKV、TiFlash の `labels` を構成する際に `zone` を構成する必要があります。詳細については、[TiKV および TiFlash の `labels` を構成する](#configure-labels-for-tikv-and-tiflash)を参照してください。

### PD のために `location-labels` を構成する

前述の説明に基づいて、ラベルは TiKV の属性を記述するために使用されるキーと値のペアであることができます。ただし、PD は位置に関連するラベルとこれらのラベルの階層関係を識別することができません。したがって、以下のように PD を理解させるために以下の構成が必要です。

文字列の配列として定義されている `location-labels` は PD の構成です。この構成の各項目は TiKV の `labels` のキーに対応します。また、各キーの順序は異なるラベルの階層関係を表し、各キーの隔離レベルは左から右に減少します。

`zone`、`rack`、または `host` などの `location-labels` の値をカスタマイズできます。この構成にはデフォルト値がありません。また、この構成には、TiKV サーバのラベルと一致する限りラベルレベルの数に制限はありません。

> **注意:**
>
> - 構成を有効にするには、PD の `location-labels` を構成し、同時に TiKV の `labels` を構成する必要があります。そうしないと、PD はトポロジに応じてスケジューリングを実行しません。
> - SQL で配置ルールを使用する場合、TiKV の `labels` のみを構成する必要があります。現在、SQL で配置ルールは PD の `location-labels` 構成と互換性がなく、この構成を無視します。`location-labels` と SQL で配置ルールを同時に使用することは推奨されません。それ以外の場合、予期しない結果が発生する可能性があります。

クラスタを [TiUP を使用して構成](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)する

TiUP を使用してクラスタをデプロイする際には、TiKV の位置を [初期化構成ファイル](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)で構成することができます。TiUP はデプロイ時に TiKV、PD、TiFlash の対応する設定ファイルを生成します。

次の例では、`zone/host` の 2 層トポロジが定義されています。クラスタの TiKV ノードと TiFlash ノードが z1、z2、z3 の 3 つのゾーンに分散されています。

- 各ゾーンには、2 つのホストが TiKV インスタンスがデプロイされています。z1 では、各ホストに 2 つの TiKV インスタンスがデプロイされています。z2 と z3 では、各ホストに個別の TiKV インスタンスがデプロイされています。
- 各ゾーンには、2 つのホストが TiFlash インスタンスがデプロイされており、各ホストに個別の TiFlash インスタンスがデプロイされています。

次の例では、`tikv-host-machine-n` は `n` 番目の TiKV ノードの IP アドレスを表し、`tiflash-host-machine-n` は `n` 番目の TiFlash ノードの IP アドレスを表します。

```
server_configs:
  pd:
    replication.location-labels: ["zone", "host"]

tikv_servers:
# z1
  # z1 の machine-1
  - host: tikv-host-machine-1
    port：20160
    config:
      server.labels:
        zone: z1
        host: tikv-host-machine-1
  - host: tikv-host-machine-1
    port：20161
    config:
      server.labels:
        zone: z1
        host: tikv-host-machine-1
  # z1 の machine-2
  - host: tikv-host-machine-2
    port：20160
    config:
      server.labels:
        zone: z1
        host: tikv-host-machine-2
  - host: tikv-host-machine-2
    port：20161
    config:
      server.labels:
        zone: z1
        host: tikv-host-machine-2
# z2
  - host: tikv-host-machine-3
    config:
      server.labels:
        zone: z2
        host: tikv-host-machine-3
- host: tikv-host-machine-4
  config:
    server.labels:
      zone: z2
      host: tikv-host-machine-4
# z3
- host: tikv-host-machine-5
  config:
    server.labels:
      zone: z3
      host: tikv-host-machine-5
- host: tikv-host-machine-6
  config:
    server.labels:
      zone: z3
      host: tikv-host-machine-6

tiflash_servers:
# z1
- host: tiflash-host-machine-1
  learner_config:
    server.labels:
      zone: z1
      host: tiflash-host-machine-1
- host: tiflash-host-machine-2
  learner_config:
    server.labels:
      zone: z1
      host: tiflash-host-machine-2
# z2
- host: tiflash-host-machine-3
  learner_config:
    server.labels:
      zone: z2
      host: tiflash-host-machine-3
- host: tiflash-host-machine-4
  learner_config:
    server.labels:
      zone: z2
      host: tiflash-host-machine-4
# z3
- host: tiflash-host-machine-5
  learner_config:
    server.labels:
      zone: z3
      host: tiflash-host-machine-5
- host: tiflash-host-machine-6
  learner_config:
    server.labels:
      zone: z3
      host: tiflash-host-machine-6
```

詳細は[地理的展開トポロジ](/geo-distributed-deployment-topology.md)を参照してください。

> **注意:**
>
> 構成ファイルで`replication.location-labels`を構成していない場合、このトポロジファイルを使用してクラスタを展開する際にエラーが発生する可能性があります。クラスタの展開前に、構成ファイルで`replication.location-labels`が確認されることをお勧めします。

## トポロジラベルに基づくPDスケジュール

PDは、異なるデータの同じ複製ができるだけ散らばるようにラベル層に従って複製をスケジュールします。

前のセクションのトポロジを例に取ってみましょう。

クラスタの複製数が3（`max-replicas=3`）だと仮定します。全体で3つのゾーンがあるため、PDは各リージョンの3つの複製がそれぞれz1、z2、z3に配置されるように保証します。このようにすることで、1つのゾーンが故障してもTiDBクラスタは引き続き利用可能です。

次に、クラスタの複製数が5（`max-replicas=5`）だと仮定します。全体で3つのゾーンしかないため、PDはゾーンレベルで各複製の分離を保証することができません。この状況では、PDスケジューラはホストレベルで複製の分離を保証します。言い換えると、リージョンの複数の複製が同じゾーンに配置されることがありますが、同じホストには配置されません。

5複製の構成の場合、z3が故障したり全体で分離されたりして一定時間（`max-store-down-time`によって制御される）回復できない場合、PDはスケジューリングを通じて5つの複製を補います。この時、ホストが4つしか利用可能でないことを意味します。これはホストレベルの分離が保証できず、複数の複製が同じホストにスケジュールされる可能性があることを意味します。ただし`isolation-level`の値が空欄に残さず`zone`に設定されている場合、これはリージョン複製の最小物理的分離要件を指定します。つまり、PDは同じリージョンの複製が異なるゾーンに散らばるように保証します。複数の複製の`max-replicas`の要件を満たさない場合でも、この分離制限に従う計画を実行しません。

`isolation-level`設定が`zone`になっている場合、これは物理レベルでのリージョン複製の最小分離要件を指定します。この場合、PDは常に同じリージョンの複製が異なるゾーンに分散されることを保証します。複数の複製の`max-replicas`の要件を満たさない場合でも、PDはそれに応じてスケジュールしません。z1、z2、z3の3つのデータゾーンに分散したTiKVクラスタを例に取ると、各リージョンが3つの複製を必要とする場合、PDは同じリージョンの3つの複製をそれぞれのデータゾーンに配置します。z1で停電が発生し、一定時間（デフォルトで30分, `max-store-down-time`によって制御される）の後に回復できない場合、PDはリージョン複製がz1では利用できないと判断します。ただし、`isolation-level`が`zone`に設定されているため、PDは同じリージョンの異なる複製が同じデータゾーンにスケジュールされないように厳密に保証する必要があります。z2とz3に既に複製があるため、`isolation-level`の最小分離要件に従ってスケジューリングを実行しません。異なるゾーンへの複製の同じリージョンを保証するためです。

同様に、`isolation-level`が`rack`に設定されている場合、最小分離レベルは同じデータセンター内の異なるラックに適用されます。この構成では、可能であればまずゾーンのレベルで分離が保証されます。ゾーンのレベルの分離が保証されない場合、PDは同じゾーン内の異なるラックに異なる複製をスケジュールするようにします。`isolation-level`が`host`に設定されている場合も同様で、PDはまずラックの分離レベルを保証し、次にホストのレベルを保証します。

要するに、PDは現在のトポロジに基づいてクラスタの災害復旧を最大限にします。そのため、一定の災害復旧レベルを達成したい場合は、トポロジに応じて`max-replicas`の数より多くのマシンを異なるサイトに展開することを推奨します。また、TiDBはさまざまなシナリオに応じてデータのトポロジ的分離レベルを柔軟に制御するための`isolation-level`などの強制的な構成項目を提供しています。