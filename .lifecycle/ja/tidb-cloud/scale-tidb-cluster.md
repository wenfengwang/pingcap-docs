---
title: TiDB クラスタのスケーリング
summary: TiDB Cloud クラスタのスケーリング方法について学びます。

# TiDB クラスタのスケーリング

> **注意:**
>
> - [TiDB Serverless クラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless) はスケーリングできません。
> - クラスタが **MODIFYING** ステータスの場合、新しいスケーリング操作を実行することはできません。

次の次元で TiDB クラスタをスケーリングできます:

- TiDB、TiKV、および TiFlash のノード数
- TiDB、TiKV、および TiFlash の vCPU と RAM
- TiKV および TiFlash のストレージ

TiDB クラスタのサイズを決定する方法についての詳細は、[TiDB サイズの決定](/tidb-cloud/size-your-cluster.md) を参照してください。

> **注意:**
>
> TiDB または TiKV の vCPU と RAM のサイズが **2 vCPU, 8 GiB (Beta)** または **4 vCPU, 16 GiB** に設定されている場合は、次の制限に注意してください。これらの制限をバイパスするには、まず [vCPU と RAM を増やす](#change-vcpu-and-ram) ことができます。
>
> - TiDB のノード数は 1 または 2にのみ設定でき、TiKV のノード数は固定されて3です。
> - 2 vCPU TiDB は 2 vCPU TiKV とのみ使用でき、2 vCPU TiKV は 2 vCPU TiDB とのみ使用できます。
> - 4 vCPU TiDB は 4 vCPU TiKV とのみ使用でき、4 vCPU TiKV は 4 vCPU TiDB とのみ使用できます。
> - TiFlash は使用できません。

## ノード数の変更

TiDB、TiKV、または TiFlash ノードの数を増やしたり減らしたりすることができます。

> **警告:**
>
> TiKV または TiFlash ノード数を減らすことはリスクが伴い、残ったノードでのストレージ容量不足、過剰な CPU 使用率、または過剰なメモリ使用率につながる可能性があります。

TiDB、TiKV、または TiFlash ノードの数を変更する手順は次のとおりです:

1. TiDB Cloud コンソールで、プロジェクトの [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動します。
2. スケーリングを行いたいクラスタの行で、**...** をクリックします。

    > **ヒント:**
    >
    > または、**クラスタ** ページでスケーリングを行いたいクラスタの名前をクリックし、右上隅の**...**をクリックすることもできます。

3. ドロップダウンメニューで **変更** をクリックします。**クラスタ変更** ページが表示されます。
4. **クラスタ変更** ページで、TiDB、TiKV、または TiFlash ノードの数を変更します。
5. 右側のペインでクラスタのサイズを確認し、**確認** をクリックします。

また、[Dedicated クラスタを変更](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster) エンドポイントを介して TiDB Cloud API を使用して TiDB、TiKV、または TiFlash ノードの数を変更することもできます。現在、TiDB Cloud API はベータ版です。詳細については、[TiDB Cloud API のドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta) を参照してください。

## vCPU と RAM の変更

TiDB、TiKV、または TiFlash ノードの vCPU と RAM を増減することができます。

> **注意:**
>
> - vCPU と RAM の変更は次のクラスタにのみ適用されます:
>     - AWS でホストされ、2022/12/31以降に作成されたクラスタ
>     - Google Cloud でホストされ、2023/04/26以降に作成されたクラスタ
> - AWS では vCPU と RAM の変更にクールダウン期間があります。AWS で TiDB クラスタをホストしている場合、ストレージ、または TiKV または TiFlash の vCPU と RAM を変更した後は、少なくとも6時間待つ必要があります。

TiDB、TiKV、または TiFlash ノードの vCPU と RAM を変更する手順は次のとおりです:

1. TiDB Cloud コンソールで、プロジェクトの [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動します。
2. スケーリングを行いたいクラスタの行で、**...** をクリックします。

    > **ヒント:**
    >
    > または、**クラスタ** ページでスケーリングを行いたいクラスタの名前をクリックし、右上隅の**...**をクリックすることもできます。

3. ドロップダウンメニューで **変更** をクリックします。**クラスタ変更** ページが表示されます。
4. **クラスタ変更** ページで、TiDB、TiKV、または TiFlash ノードの vCPU と RAM を変更します。
5. 右側のペインでクラスタのサイズを確認し、**確認** をクリックします。

また、[Dedicated クラスタを変更](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster) エンドポイントを介して TiDB Cloud API を使用して TiDB、TiKV、または TiFlash ノードの vCPU と RAM を変更することもできます。現在、TiDB Cloud API はベータ版です。詳細については、[TiDB Cloud API のドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta) を参照してください。

## ストレージの変更

TiKV または TiFlash のストレージを増やすことができます。

> **注意:**
>
> - 実行中のクラスタでは、AWS および Google Cloud はインプレースのストレージ容量のダウングレードを許可しません。
> - AWS ではストレージの変更にクールダウン期間があります。AWS で TiDB クラスタをホストしている場合、ストレージ、または TiKV または TiFlash の vCPU と RAM を変更した後は、少なくとも6時間待つ必要があります。

TiKV または TiFlash のストレージを変更する手順は次のとおりです:

1. TiDB Cloud コンソールで、プロジェクトの [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動します。
2. スケーリングを行いたいクラスタの行で、**...** をクリックします。

    > **ヒント:**
    >
    > または、**クラスタ** ページでスケーリングを行いたいクラスタの名前をクリックし、右上隅の**...**をクリックすることもできます。

3. ドロップダウンメニューで **変更** をクリックします。**クラスタ変更** ページが表示されます。
4. **クラスタ変更** ページで、各 TiKV または TiFlash ノードのストレージを変更します。
5. 右側のペインでクラスタのサイズを確認し、**確認** をクリックします。

また、[Dedicated クラスタを変更](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster) エンドポイントを介して TiDB Cloud API を使用して TiKV または TiFlash ノードのストレージを変更することもできます。現在、TiDB Cloud API はベータ版です。詳細については、[TiDB Cloud API のドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta) を参照してください。