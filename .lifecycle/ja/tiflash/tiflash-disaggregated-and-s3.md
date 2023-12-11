---
title: TiFlashの分離ストレージとコンピューティングアーキテクチャとS3サポート
summary: TiFlashの分離ストレージとコンピューティングアーキテクチャとS3サポートについて学びます。
---

# TiFlashの分離ストレージとコンピューティングアーキテクチャとS3サポート

デフォルトでは、TiFlashは結合されたストレージとコンピュートアーキテクチャを使用して展開されます。ここでは、各TiFlashノードがストレージとコンピュートの両方の役割を果たします。TiDB v7.0.0からは、TiFlashは分離ストレージとコンピュートアーキテクチャをサポートし、Amazon S3やMinIOなどのS3互換のオブジェクトストレージにデータを保存することができます。

## アーキテクチャ概要

![TiFlash 書き込みとコンピューティングの分離アーキテクチャ](/media/tiflash/tiflash-s3.png)

分離されたストレージとコンピューティングアーキテクチャでは、TiFlashプロセスの異なる機能は2種類のノード、書き込みノードとコンピュートノードに分かれ、割り当てられます。これらの2種類のノードは別々に展開し、独立してスケールすることができます。つまり、必要に応じて展開する書き込みノードとコンピュートノードの数を決定することができます。

- TiFlash書き込みノード

    書き込みノードはTiKVからRaftログデータを受信し、データを列形式に変換し、一定期間ごとに更新されたデータをS3にパッケージ化してアップロードします。また、書き込みノードは、データを管理し、クエリのパフォーマンスを向上させるために継続的にデータを整理し、不要なデータを削除します。

    書き込みノードは、最新の書き込まれたデータをキャッシュしておくためにローカルディスク（通常はNVMe SSD）を使用します。

- TiFlashコンピュートノード

    コンピュートノードはTiDBノードから送信されたクエリリクエストを実行します。まず、書き込みノードにアクセスしてデータのスナップショットを取得し、その後、最新のデータ（つまり、まだS3にアップロードされていないデータ）を書き込みノードから読み込み、残りのほとんどのデータをS3から読み込みます。

    コンピュートノードは状態を持たず、スケーリング速度は秒単位です。これを使用してコストを削減できます：

    - クエリのワークロードが低い場合、コンピュートノードの数を減らしてコストを節約します。クエリがない場合、すべてのコンピュートノードを停止することさえできます。
    - クエリのワークロードが増加すると、クエリのパフォーマンスを確保するために迅速にコンピュートノードの数を増やすことができます。

## シナリオ

TiFlashの分離されたストレージとコンピューティングアーキテクチャは、コスト効率の高いデータ解析サービスに適しています。このアーキテクチャでは、ストレージとコンピュートリソースを必要に応じて別々にスケーリングできるため、次のようなシナリオでかなりの利点を得ることができます。

- データ量が多いが、頻繁にクエリされるデータはわずかで、ほとんどのデータは冷たいデータで滅多にクエリされません。この場合、頻繁にクエリされるデータは通常、コンピュートノードのローカルSSDにキャッシュされて高速なクエリパフォーマンスを提供し、ほとんどの他の冷たいデータは低コストのS3や他のオブジェクトストレージに保存され、ストレージコストを節約します。

- コンピュートリソースの需要が明らかなピークと谷があります。たとえば、集中的な調整クエリは通常夜に行われ、高いコンピュートリソースを要求します。この場合は、夜間に一時的により多くのコンピュートノードを追加することを検討できます。その他の時間帯では、定期的なクエリタスクを完了するために少ないコンピュートノードしか必要ありません。

## 前提条件

1. TiFlashデータを保存するためのAmazon S3バケットを準備します。

    既存のバケットを使用することもできますが、各TiDBクラスタのために専用のキーのプレフィックスを予約する必要があります。S3バケットに関する詳細な情報については、[AWSドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/creating-buckets-s3.html)を参照してください。

    [MinIO](https://min.io/)のような他のS3互換のオブジェクトストレージを使用することもできます。

    TiFlashはデータにアクセスするために以下のS3 APIを使用する必要があります。TiDBクラスタのTiFlashノードがこれらのAPIに必要な権限を持っていることを確認してください。

    - PutObject
    - GetObject
    - CopyObject
    - DeleteObject
    - ListObjectsV2
    - GetObjectTagging
    - PutBucketLifecycle

2. TiDBクラスタに結合されたストレージとコンピュートアーキテクチャを使用して展開されたTiFlashノードがないことを確認します。ある場合は、すべてのテーブルのTiFlashレプリカの数を `0` に設定し、その後、すべてのTiFlashノードを削除します。例：

    ```sql
    SELECT * FROM INFORMATION_SCHEMA.TIFLASH_REPLICA; # TiFlashレプリカを持つすべてのテーブルをクエリ
    ALTER TABLE table_name SET TIFLASH REPLICA 0;     # すべてのテーブルのTiFlashレプリカの数を `0` に設定
    ```

    ```shell
    tiup cluster scale-in mycuster -R tiflash # すべてのTiFlashノードを削除
    tiup cluster display mycluster            # すべてのTiFlashノードがTombstone状態になるのを待つ
    tiup cluster prune mycluster              # Tombstone状態のすべてのTiFlashノードを削除
    ```

## 使用方法

デフォルトでは、TiUPは結合されたストレージとコンピューティングアーキテクチャでTiFlashを展開します。分離されたストレージとコンピューティングアーキテクチャでTiFlashを展開する場合は、手動で次の手順を実行してください：

1. 次の構成でTiFlashトポロジ設定ファイル（例：`scale-out.topo.yaml`）を準備します：

    ```yaml
    tiflash_servers:
      # TiFlashのトポロジ設定ファイルでは、`storage.s3`の構成は分離されたストレージとコンピューティングアーキテクチャが展開されていることを示しています。
      # ノードに`flash.disaggregated_mode: tiflash_compute`が構成されている場合、それはコンピュートノードです。
      # ノードに`flash.disaggregated_mode: tiflash_write`が構成されている場合、それは書き込みノードです。

      # 172.31.8.1~2 はTiFlashの書き込みノードです
      - host: 172.31.8.1
        config:
          flash.disaggregated_mode: tiflash_write               # これは書き込みノードです
          storage.s3.endpoint: http://s3.{region}.amazonaws.com # S3エンドポイントアドレス
          storage.s3.bucket: mybucket                           # TiFlashはこのバケットにすべてのデータを保存します
          storage.s3.root: /cluster1_data                       # S3バケット内のデータが格納されるルートディレクトリ
          storage.s3.access_key_id: {ACCESS_KEY_ID}             # ACCESS_KEY_IDでS3にアクセスします
          storage.s3.secret_access_key: {SECRET_ACCESS_KEY}     # SECRET_ACCESS_KEYでS3にアクセスします
          storage.main.dir: ["/data1/tiflash/data"]             # 書き込みノードのローカルデータディレクトリ。結合されたストレージとコンピュートアーキテクチャのディレクトリ構成と同じように構成してください。
      - host: 172.31.8.2
        config:
          flash.disaggregated_mode: tiflash_write               # これは書き込みノードです
          storage.s3.endpoint: http://s3.{region}.amazonaws.com # S3エンドポイントアドレス
          storage.s3.bucket: mybucket                           # TiFlashはこのバケットにすべてのデータを保存します
          storage.s3.root: /cluster1_data                       # S3バケット内のデータが格納されるルートディレクトリ
          storage.s3.access_key_id: {ACCESS_KEY_ID}             # ACCESS_KEY_IDでS3にアクセスします
          storage.s3.secret_access_key: {SECRET_ACCESS_KEY}     # SECRET_ACCESS_KEYでS3にアクセスします
          storage.main.dir: ["/data1/tiflash/data"]             # 書き込みノードのローカルデータディレクトリ。結合されたストレージとコンピュートアーキテクチャのディレクトリ構成と同じように構成してください。

      # 172.31.9.1~2 はTiFlashのコンピュートノードです
      - host: 172.31.9.1
        config:
          flash.disaggregated_mode: tiflash_compute             # これはコンピュートノードです
          storage.s3.endpoint: http://s3.{region}.amazonaws.com # S3エンドポイントアドレス
          storage.s3.bucket: mybucket                           # TiFlashはこのバケットにすべてのデータを保存します
          storage.s3.root: /cluster1_data                       # S3バケット内のデータが格納されるルートディレクトリ
          storage.s3.access_key_id: {ACCESS_KEY_ID}             # ACCESS_KEY_IDでS3にアクセスします
          storage.s3.secret_access_key: {SECRET_ACCESS_KEY}     # SECRET_ACCESS_KEYでS3にアクセスします
          storage.main.dir: ["/data1/tiflash/data"]             # コンピュートノードのローカルデータディレクトリ。結合されたストレージとコンピュートアーキテクチャのディレクトリ構成と同じように構成してください
          storage.remote.cache.dir: /data1/tiflash/cache        # コンピュートノードのローカルデータキャッシュディレクトリ
          storage.remote.cache.capacity: 858993459200           # 800 GiB
      - host: 172.31.9.2
        config:
          flash.disaggregated_mode: tiflash_compute             # これはコンピュートノードです
          storage.s3.endpoint: http://s3.{region}.amazonaws.com # S3エンドポイントアドレス
          storage.s3.bucket: mybucket                           # TiFlashはこのバケットにすべてのデータを保存します
          storage.s3.root: /cluster1_data                       # S3バケット内のデータが格納されるルートディレクトリ
          storage.s3.access_key_id: {ACCESS_KEY_ID}             # ACCESS_KEY_IDでS3にアクセスします
          storage.s3.secret_access_key: {SECRET_ACCESS_KEY}     # SECRET_ACCESS_KEYでS3にアクセスします
```
          storage.main.dir: ["/data1/tiflash/data"]             # Compute Nodeのローカルデータディレクトリ。ストレージとコンピューティングの連携アーキテクチャのディレクトリ構成と同じように構成します
          storage.remote.cache.dir: /data1/tiflash/cache        # Compute Nodeのローカルデータキャッシュディレクトリ
          storage.remote.cache.capacity: 858993459200           # 800 GiB
    ```

    * 上記の `ACCESS_KEY_ID` と `SECRET_ACCESS_KEY` は、設定ファイルに直接記述されています。環境変数を使用して別々に設定することもできます。両方の方法を設定する場合は、環境変数が優先されます。

        `ACCESS_KEY_ID` と `SECRET_ACCESS_KEY` を環境変数を使用して設定するには、TiFlashプロセス（通常は`tidb`）を起動するユーザー環境に切り替え、TiFlashプロセスが展開されているすべてのマシンで `~/.bash_profile` を変更して、次の構成を追加します：

        ```shell
        export S3_ACCESS_KEY_ID={ACCESS_KEY_ID}
        export S3_SECRET_ACCESS_KEY={SECRET_ACCESS_KEY}
        ```

    * `storage.s3.endpoint` は、`http` または `https` モードでS3に接続することをサポートし、URLを直接変更することでモードを設定できます。例えば、`https://s3.{region}.amazonaws.com`。

2. TiFlashノードを追加し、TiFlashレプリカの数をリセットします：

    ```shell
    tiup cluster scale-out mycluster ./scale-out.topo.yaml
    ```

    ```sql
    ALTER TABLE table_name SET TIFLASH REPLICA 1;
    ```

3. TiDB構成を変更し、分離されたストレージとコンピューティングアーキテクチャを使用してTiFlashをクエリします。

    1. TiDB構成ファイルを編集モードで開きます：

          ```shell
          tiup cluster edit-config mycluster
          ```

    2. TiDB構成ファイルに以下の構成項目を追加します：

        ```shell
        server_configs:
        tidb:
        disaggregated-tiflash: true   # 分離されたストレージとコンピューティングアーキテクチャを使用してTiFlashをクエリする
        ```

    3. TiDBを再起動します：

        ```shell
        tiup cluster reload mycluster -R tidb
        ```

## 制限事項

- TiFlashは **連携されたストレージとコンピューティングアーキテクチャ** と **分離されたストレージとコンピューティングアーキテクチャ** の間でのインプレースの切り替えをサポートしていません。分離されたアーキテクチャに切り替える前に、連携アーキテクチャを使用して展開されているすべての既存のTiFlashノードを削除する必要があります。
- 一つのアーキテクチャから別のアーキテクチャへの移行後、すべてのTiFlashデータを再度レプリケーションする必要があります。
- 同じTiDBクラスター内で同じアーキテクチャのみを使用できます。二つのアーキテクチャは同一のクラスター内で共存できません。
- 分離されたストレージとコンピューティングアーキテクチャはS3 APIを使用したオブジェクトストレージのみをサポートし、一方、連携されたストレージとコンピューティングアーキテクチャはローカルストレージのみをサポートしています。
- S3ストレージを使用する場合、TiFlashノードは自分自身のノード上にないファイルのキーを取得することができないため、[静置時の暗号化](/encryption-at-rest.md) 機能は使用できません。