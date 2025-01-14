---
title: バックアップの自動チューニング
summary: TiDBのバックアップとリストアの自動チューニング機能について学びましょう。この機能は、高いクラスタリソース使用時にバックアップがクラスタに与える影響を低減するため、バックアップに使用されるリソースを自動的に制限します。

# バックアップの自動チューニング <span class="version-mark">v5.4.0で新規導入</span>

TiDB v5.4.0以前では、Backup & Restore（BR）を使用してデータをバックアップする際、バックアップに使用されるスレッド数は論理CPUコアの75%を占めていました。速度制限がないと、バックアッププロセスは多くのクラスタリソースを消費し、オンラインクラスタの性能にかなりの影響を与えます。スレッドプールのサイズを調整することでバックアップの影響を削減することができますが、CPU負荷を観察して手動でスレッドプールのサイズを調整するのは手間がかかります。

クラスタへのバックアップタスクの影響を低減するために、TiDB v5.4.0では自動チューニング機能が導入され、デフォルトで有効化されています。クラスタリソースの利用が高い場合、BRは自動的にバックアップタスクに使用されるリソースを制限し、それによってクラスタへの影響を減少させます。自動チューニング機能はデフォルトで有効化されています。

## 使用シナリオ

バックアップタスクがクラスタに与える影響を低減したい場合は、自動チューニング機能を有効にできます。この機能を有効にすると、TiDBはクラスタに過剰な影響を与えることなくバックアップタスクをできるだけ速く実行します。

また、TiKVの構成項目 [`backup.num-threads`](/tikv-configuration-file.md#num-threads-1) を使用するか、パラメータ `--ratelimit` を使用してバックアップ速度を制限することもできます。

## 自動チューニングの使用

自動チューニング機能は追加の設定不要でデフォルトで有効化されています。

> **注意:**
>
> v5.3.xからv5.4.0以降にアップグレードしたクラスタでは、自動チューニング機能はデフォルトで無効化されています。手動で有効にする必要があります。

自動チューニング機能を手動で有効にするには、TiKVの構成項目 [`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540) を `true` に設定する必要があります。

TiKVでは、クラスタを再起動することなく、自動チューニング機能を動的に設定できます。自動チューニング機能を動的に有効または無効にするには、次のコマンドを実行します。

{{< copyable "shell-regular" >}}

```shell
tikv-ctl modify-tikv-config -n backup.enable-auto-tune -v <true|false>
```

オフラインクラスタでバックアップタスクを実行する場合、`tikv-ctl` を使用して `backup.num-threads` の値を大きな値に変更してバックアップを高速化することができます。

## 制限事項

自動チューニングはバックアップ速度を制限するための厳密な解決策ではなく、手動の調整を必要としません。ただし、細かい制御が不足しているため、自動チューニングではバックアップがクラスタに与える影響を完全に取り除けない場合があります。

自動チューニング機能には次の問題があり、それに対応する解決策があります。

- 問題1: **書き込みが多いクラスタ** の場合、自動チューニングはワークロードとバックアップタスクを「正のフィードバックループ」に入れる可能性があります。バックアップタスクが多くのリソースを使用するため、クラスタが少ないリソースを使用するようになり、この状況で自動チューニングが誤ってクラスタが非常に多いワークロードでないと見なし、バックアップの実行を許可する可能性があります。このような場合、自動チューニングは無効です。

    - 解決策: `backup.num-threads` をより小さな数に手動で調整して、バックアップタスクが使用するスレッド数を制限します。動作原理は以下の通りです。

        バックアッププロセスには多くのSSTのデコード、エンコード、圧縮、解凍が含まれ、CPUリソースを消費します。また、以前のテストケースではバックアッププロセス中、バックアップタスクによって使用されるスレッドプールのCPU利用率が100%に近いことが示されています。つまり、バックアップタスクは多くのCPUリソースを使用します。バックアップタスクが使用するスレッド数を調整することで、TiKVはバックアップタスクによって使用されるCPUコア数を制限し、それによりクラスタへのバックアップタスクの影響を低減します。

- 問題2: **ホットスポットがあるクラスタ** の場合、ホットスポットがあるTiKVノード上のバックアップタスクが過度に制限され、全体のバックアッププロセスが遅くなる可能性があります。

    - 解決策: ホットスポットノードを除去するか、ホットスポットノード上で自動チューニングを無効にします（これによりクラスタのパフォーマンスが低下する可能性があります）。

- 問題3: **トラフィックの乱れが大きいシナリオ** の場合、自動チューニングは速度制限を固定間隔（デフォルトでは1分）で調整するため、トラフィックの大きな乱れに対応できない場合があります。詳細は [`auto-tune-refresh-interval`](#implementation) を参照してください。

    - 解決策: 自動チューニングを無効にします。

## 実装

自動チューニングはバックアップタスクに使用されるスレッドプールのサイズを調整し、クラスタの全体的なCPU利用率が特定のしきい値を超えないようにします。

この機能には、TiKV構成ファイルに記載されていない2つの関連する構成項目があります。これらの2つの構成項目は内部調整用であり、バックアップタスクを実行する際にこれらの構成項目を設定する必要はありません。

- `backup.auto-tune-remain-threads`:

    - 自動チューニングは、バックアップタスクによって使用されるリソースを制御し、同じノード上の他のタスクに最低限 `backup.auto-tune-remain-threads` コアが利用可能であることを確保します。
    - デフォルト値: `round(0.2 * vCPU)`

- `backup.auto-tune-refresh-interval`:

    - `backup.auto-tune-refresh-interval` 分ごとに、自動チューニングは統計を更新し、バックアップタスクが使用できるCPUコアの最大数を再計算します。
    - デフォルト値: `1m`

以下は、自動チューニングの動作例です。`*` はバックアップタスクによって使用されるCPUコアを示し、`^` は他のタスクによって使用されるCPUコアを示します。`-` はアイドル状態のCPUコアを示します。

```
|--------| サーバーには論理CPUコア8があります。
|****----| デフォルトでは、`backup.num-threads` は `4` です。自動チューニングはスレッドプールのサイズが常に `backup.num-threads` より大きくならないようにします。
|^^****--| デフォルトでは、`auto-tune-remain-threads` = round(8 * 0.2) = 2 です。自動チューニングはスレッドプールのサイズを `4` に調整します。
|^^^^**--| クラスタのワークロードが高くなると、自動チューニングはスレッドプールのサイズを `2` に調整します。その後、クラスタには2つのアイドルCPUコアがあります。
```

**バックアップCPU利用率** パネルでは、自動チューニングによって調整されたスレッドプールのサイズが表示されます:

![Grafanaダッシュボードのバックアップ自動チューニング指標の例](/media/br/br-auto-throttle.png)

上記画像では、黄色の半透明エリアがバックアップタスクに使用可能なスレッドを表しています。バックアップタスクのCPU利用率がこの黄色い領域を超えないようになっています。