---
title: 配置规则
summary: 学习如何配置放置规则。
aliases: ['/docs/dev/configure-placement-rules/','/docs/dev/how-to/configure/placement-rules/']
---

# 放置规则

> **注意：**
>
> 本文介绍如何在放置 Driver（PD）中手动指定放置规则。现在建议使用[SQL中的放置规则](/placement-rules-in-sql.md)。这提供了一种更方便的方式来配置表和分区的放置。

放置规则是在 v5.0 中引入的复制规则系统，指导 PD 为不同类型的数据生成相应的调度。通过组合不同的调度规则，您可以精细控制任何连续数据范围的属性，例如副本数量、存储位置、主机类型、是否参与Raft选举以及是否充当Raft领导者。

放置规则功能在 v5.0 及以后的 TiDB 版本中默认启用。要禁用它，请参见[禁用放置规则](#disable-placement-rules)。

## 规则系统

整个规则系统的配置由多个规则组成。每个规则可以指定诸如副本数量、Raft 角色、放置位置以及此规则生效的键范围等属性。当 PD 执行调度时，它首先根据 Region 的键范围在规则系统中找到与之对应的规则，然后生成相应的调度，使得 Region 的副本分布符合规则。

多个规则的键范围可以有重叠部分，这意味着一个 Region 可能与多个规则匹配。在这种情况下，PD 根据规则的属性决定规则是互相覆盖还是同时生效。如果多个规则同时生效，PD 将根据规则的堆叠顺序依次生成调度来进行规则匹配。

此外，为了满足来自不同来源的规则相互隔离的要求，这些规则可以以更加灵活的方式进行组织。因此，引入了“组”的概念。通常，用户可以根据不同的来源将规则放置在不同的组中。

![放置规则概述](/media/placement-rules-1.png)

### 规则字段

以下表格显示了规则中每个字段的含义：

| 字段名称           | 类型和限制                      | 描述                                |
| :---            | :---                           | :---                                |
| `GroupID`         | `string`                         |  标记规则来源的组ID。                |
| `ID`              | `string`                         |  组中规则的唯一ID。                    |
| `Index`           | `int`                            |  组中规则的堆叠序列。                     |
| `Override`       | `true`/`false`                     | 是否覆盖较小索引的规则（在同一组中）。  |
| `StartKey`        | `string`，十六进制形式              |  应用于范围的起始键。                |
| `EndKey`          | `string`，十六进制形式              |  应用于范围的结束键。                |
| `Role`            | `string` | 复制角色，包括投票者/领导者/跟随者/学习者。                     |
| `Count`           | `int`，正整数                      |  副本数量。                            |
| `LabelConstraint` | `[]Constraint`                    |  根据标签过滤节点。                |
| `LocationLabels`  | `[]string`                        |  用于物理隔离。                    |
| `IsolationLevel`  | `string`                          |  用于设置最小物理隔离级别。

`LabelConstraint` 类似于 Kubernetes 中根据四个原语过滤标签的功能：`in`、`notIn`、`exists` 和 `notExists`。这四个原语的含义如下：

+ `in`：给定键的标签值包含在给定列表中。
+ `notIn`：给定键的标签值不包含在给定列表中。
+ `exists`：包括给定的标签键。
+ `notExists`：不包括给定的标签键。

`LocationLabels` 的含义和功能与早于 v4.0 的内容相同。例如，如果您已部署了`[zone，rack，host]`，定义了三层拓扑结构：集群有多个区域（可用区），每个区域有多个机架，每个机架有多个主机。在进行调度时，PD 首先尝试将 Region 的对等体放置在不同的区域。如果此尝试失败（例如副本有三个，但总共只有两个区域），PD 会确保将这些副本放置在不同的机架。如果机架数量不足以保证隔离，那么 PD 尝试主机级别的隔离。

`IsolationLevel` 的含义和功能在[按拓扑标签调度副本](/schedule-replicas-by-topology-labels.md)中有详细说明。例如，如果您已部署了`[zone，rack，host]`，定义了具有`LocationLabels`的三层拓扑结构，并将`IsolationLevel`设置为`zone`，那么 PD 确保每个 Region 的所有对等体在调度时都被放置在不同的区域。如果无法满足`IsolationLevel`的最小隔离级别限制（例如配置了3个副本，但总共只有2个数据区域），PD 不会尝试弥补以满足此限制。`IsolationLevel` 的默认值为空字符串，表示已禁用。

### 规则组的字段

以下表格显示了规则组中每个字段的描述：

| 字段名称 | 类型和限制 | 描述 |
| :--- | :--- | :--- |
| `ID` | `string` | 标记规则来源的组ID。 |
| `Index` | `int` | 不同组的堆叠顺序。 |
| `Override` | `true`/`false` | 是否覆盖较小索引的组。 |

## 配置规则

本节中的操作基于[pd-ctl](/pd-control.md)，并且操作涉及的命令也支持通过 HTTP API 进行调用。

### 启用放置规则

放置规则功能在 v5.0 及以后的 TiDB 版本中默认启用。要禁用它，请参见[禁用放置规则](#disable-placement-rules)。在禁用后要启用此功能，可以在初始化集群之前修改 PD 配置文件如下：

{{< copyable "" >}}

```toml
[replication]
enable-placement-rules = true
```

这样，PD 在成功初始化集群后启用此功能，并根据 `max-replicas` 和 `location-labels` 配置生成相应的规则：

{{< copyable "" >}}

```json
{
  "group_id": "pd",
  "id": "default",
  "start_key": "",
  "end_key": "",
  "role": "voter",
  "count": 3,
  "location_labels": ["zone", "rack", "host"],
  "isolation_level": ""
}
```

对于已引导的集群，您还可以通过 pd-ctl 动态启用放置规则：

{{< copyable "shell-regular" >}}

```bash
pd-ctl config placement-rules enable
```

PD 还会根据 `max-replicas` 和 `location-labels` 配置生成默认规则。

> **注意：**
>
> 启用放置规则后，以前配置的 `max-replicas` 和 `location-labels` 不再生效。要调整副本策略，请使用与放置规则相关的接口。

### 禁用放置规则

您可以使用 pd-ctl 来禁用放置规则功能并切换回先前的调度策略。

{{< copyable "shell-regular" >}}

```bash
pd-ctl config placement-rules disable
```

> **注意：**
>
> 禁用放置规则后，PD 使用原始的 `max-replicas` 和 `location-labels` 配置。在启用放置规则时，规则的修改（当放置规则启用时）不会实时更新这两个配置。此外，所有已配置的规则保留在 PD 中，并将在下一次启用放置规则时使用。

### 使用 pd-ctl 设置规则

> **注意：**
>
> 规则的更改会实时影响 PD 调度。不当的规则设置可能导致较少的副本并影响系统的高可用性。

pd-ctl支持使用以下方法查看系统中的规则，输出为 JSON 格式的规则或规则列表。

- 查看所有规则的列表：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules show
    ```

- 查看 PD 组中所有规则的列表：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules show --group=pd
    ```

- 查看组中特定ID的规则：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules show --group=pd --id=default
    ```

- 查看与 Region 匹配的规则列表：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules show --region=2
    ```

上記の例では、「2」はリージョンIDです。

ルールの追加と編集は似ています。対応するルールをファイルに書き込み、次に`save`コマンドを使用してルールをPDに保存する必要があります：

{{< copyable "shell-regular" >}}

```bash
cat > rules.json <<EOF
[
    {
        "group_id": "pd",
        "id": "rule1",
        "role": "voter",
        "count": 3,
        "location_labels": ["zone", "rack", "host"]
    },
    {
        "group_id": "pd",
        "id": "rule2",
        "role": "voter",
        "count": 2,
        "location_labels": ["zone", "rack", "host"]
    }
]
EOF
pd-ctl config placement save --in=rules.json
```

上記の操作は`rule1`と`rule2`をPDに書き込みます。システムに同じ`GroupID` + `ID`のルールがすでに存在する場合、このルールは上書きされます。

ルールを削除するには、ルールの`count`を`0`に設定するだけで、同じ`GroupID` + `ID`のルールが削除されます。次のコマンドは`pd/rule2`のルールを削除します：

{{< copyable "shell-regular" >}}

```bash
cat > rules.json <<EOF
[
    {
        "group_id": "pd",
        "id": "rule2"
    }
]
EOF
pd-ctl config placement save --in=rules.json
```

### pd-ctlを使用してルールグループを構成する

- すべてのルールグループのリストを表示するには：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules rule-group show
    ```

- 特定のIDのルールグループを表示するには：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules rule-group show pd
    ```

- ルールグループの`index`と`override`属性を設定するには：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules rule-group set pd 100 true
    ```

- ルールグループの構成を削除するには（グループにルールがある場合はデフォルトのグループ構成を使用する）：

    {{< copyable "shell-regular" >}}

    ```bash
    pd-ctl config placement-rules rule-group delete pd
    ```

```json
{
  "start_key": "",
  "end_key": "",
  "role": "voter",
  "count": 2,
  "label_constraints": [
      {"key": "zone", "op": "in", "values": ["zone2"]}
  ],
  "location_labels": ["rack", "host"]
},
{
  "group_id": "pd",
  "id": "zone3",
  "start_key": "",
  "end_key": "",
  "role": "follower",
  "count": 1,
  "label_constraints": [
      {"key": "zone", "op": "in", "values": ["zone3"]}
  ],
  "location_labels": ["rack", "host"]
}
```

### シナリオ3：テーブルに2つのTiFlashレプリカを追加する

テーブルの行キーに対して個別のルールを追加し、`count`を`2`に制限します。`label_constraints`を使用して、レプリカが`engine = tiflash`のノードに生成されるようにします。なお、ここでは別個の`group_id`が使用されており、このルールがシステム内の他のソースからのルールと重複したり競合したりしないようにします。

{{< copyable "" >}}

```json
{
  "group_id": "tiflash",
  "id": "learner-replica-table-ttt",
  "start_key": "7480000000000000ff2d5f720000000000fa",
  "end_key": "7480000000000000ff2e00000000000000f8",
  "role": "learner",
  "count": 2,
  "label_constraints": [
    {"key": "engine", "op": "in", "values": ["tiflash"]}
  ],
  "location_labels": ["host"]
}
```

### シナリオ4：北京ノードで高性能ディスクを使用してテーブルに2つのfollowerレプリカを追加する

以下の例はより複雑な`label_constraints`の構成を示しています。このルールでは、レプリカは`bj1`または`bj2`の機械室に配置する必要があり、ディスクのタイプは`nvme`である必要があります。

{{< copyable "" >}}

```json
{
  "group_id": "follower-read",
  "id": "follower-read-table-ttt",
  "start_key": "7480000000000000ff2d00000000000000f8",
  "end_key": "7480000000000000ff2e00000000000000f8",
  "role": "follower",
  "count": 2,
  "label_constraints": [
    {"key": "zone", "op": "in", "values": ["bj1", "bj2"]},
    {"key": "disk", "op": "in", "values": ["nvme"]}
  ],
  "location_labels": ["host"]
}
```

### シナリオ5：SSDディスクを搭載したノードにテーブルを移行する

シナリオ3とは異なり、このシナリオは既存の構成を基に新しいレプリカを追加するのではなく、データ範囲の他の構成を強制的にオーバーライドするものです。したがって、ルールグループ構成で`index`値を十分に大きな値で指定し、`override`を`true`に設定する必要があります。

ルール:

{{< copyable "" >}}

```json
{
  "group_id": "ssd-override",
  "id": "ssd-table-45",
  "start_key": "7480000000000000ff2d5f720000000000fa",
  "end_key": "7480000000000000ff2e00000000000000f8",
  "role": "voter",
  "count": 3,
  "label_constraints": [
    {"key": "disk", "op": "in", "values": ["ssd"]}
  ],
  "location_labels": ["rack", "host"]
}
```

ルールグループ:

{{< copyable "" >}}

```json
{
  "id": "ssd-override",
  "index": 1024,
  "override": true,
}
```