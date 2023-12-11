---
title: TiSparkを使用してTiFlashレプリカを読み取る方法
summary: TiSparkを使用してTiFlashレプリカを読み取る方法について学びます。
---

# TiSparkを使用してTiFlashレプリカを読み取る方法

本書は、TiSparkを使用してTiFlashレプリカを読み取る方法について紹介します。

現在、TiDBのエンジン分離の方法に似た方法でTiSparkを使用してTiFlashレプリカを読み取ることができます。この方法は`spark.tispark.isolation_read_engines`パラメータを設定することです。パラメータ値のデフォルトは`tikv,tiflash`で、これはTiDBがCBOの選択に応じてTiFlashまたはTiKVからデータを読み取ることを意味します。パラメータ値を`tiflash`に設定した場合、TiDBは強制的にTiFlashからデータを読み取ります。

> **注意:**
>
> このパラメータが`tiflash`に設定されている場合、クエリに関連するすべてのテーブルのTiFlashレプリカのみが読み取られ、これらのテーブルはTiFlashレプリカを持つ必要があります。TiFlashレプリカを持たないテーブルの場合、エラーが報告されます。このパラメータが`tikv`に設定されている場合、TiKVレプリカのみが読み取られます。

このパラメータは、以下の方法のいずれかで設定できます：

* `spark-defaults.conf`ファイルに以下の項目を追加します：

    ```
    spark.tispark.isolation_read_engines tiflash
    ```

* Spark shellまたはThriftサーバーを初期化する際に、初期化コマンドに`--conf spark.tispark.isolation_read_engines=tiflash`を追加します。

* Spark shellでリアルタイムに`spark.conf.set("spark.tispark.isolation_read_engines", "tiflash")`を設定します。

* Thriftサーバーがbeelineを介して接続された後、Thriftサーバーで`set spark.tispark.isolation_read_engines=tiflash`を設定します。