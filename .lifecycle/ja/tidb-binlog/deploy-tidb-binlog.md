---
title: TiDB Binlogクラスターのデプロイ
summary: TiDB Binlogクラスターのデプロイ方法について学びます。
aliases: ['/docs/dev/tidb-binlog/deploy-tidb-binlog/','/docs/dev/reference/tidb-binlog/deploy/','/docs/dev/how-to/deploy/tidb-binlog/']
---

# TiDB Binlogクラスターのデプロイ

このドキュメントでは、[バイナリパッケージを使用してTiDB Binlogをデプロイする方法](#deploy-tidb-binlog-using-a-binary-package)について説明します。

## ハードウェア要件

PumpとDrainerは、Intel x86-64アーキテクチャを備えた64ビットユニバーサルハードウェアサーバープラットフォーム上で展開され、動作します。

開発、テスト、本番環境では、サーバーハードウェアの要件は以下のとおりです:

| サービス | サーバーの数 | CPU | ディスク | メモリ |
| :-------- | :-------- | :-------- | :--------------- | :------ |
| Pump | 3 | 8コア以上 | SSD、200 GB以上 | 16G |
| Drainer | 1 | 8コア以上 | SAS、100 GB以上 (binlogがローカルファイルとして出力される場合、ディスクサイズはこれらのファイルを保持する期間に依存します。) | 16G |

## TiUPを使用したTiDB Binlogのデプロイ

TiDB BinlogをTiUPを使用してデプロイすることをお勧めします。これを行うには、TiUPを使用してTiDBをデプロイする際に、「TiDB Binlogデプロイメントトポロジー」に`drainer`および`pump`のノード情報を追加する必要があります。詳細なデプロイ情報については、[TiUPを使用したTiDBクラスターのデプロイ](/production-deployment-using-tiup.md)を参照してください。

## バイナリパッケージを使用したTiDB Binlogのデプロイ

### 公式のバイナリパッケージをダウンロードする

TiDB BinlogのバイナリパッケージはTiDB Toolkitに含まれています。TiDB Toolkitをダウンロードするには、[TiDBツールのダウンロード](/download-ecosystem-tools.md)を参照してください。

### 使用例

PDノードが3つ、TiDBノードが1つ、Pumpノードが2つ、Drainerノードが1つあると仮定し、各ノードの情報は以下の通りです:

| ノード | IP           |
| :---------|:------------ |
| TiDB     | 192.168.0.10 |
| PD1      | 192.168.0.16 |
| PD2      | 192.168.0.15 |
| PD3      | 192.168.0.14 |
| Pump     | 192.168.0.11 |
| Pump     | 192.168.0.12 |
| Drainer  | 192.168.0.13 |

次の部分では、上記のノードを基にPumpとDrainerを使用する方法を示します。

1. バイナリを使用してPumpをデプロイする。

    - Pumpのコマンドラインパラメータを表示するには、`./pump -help`を実行します:

        ```bash
        使用法: Pump:
        -L string
            ログの出力情報レベル: debug、info、warn、error、fatal ("info"がデフォルト)
        -V
            バージョン情報を表示
        -addr string
            Pumpがサービスを提供するためのRPCアドレス (-addr="192.168.0.11:8250")
        -advertise-addr string
            Pumpが外部サービスを提供するためのRPCアドレス (-advertise-addr="192.168.0.11:8250")
        -config string
            構成ファイルのパス。構成ファイルを指定すると、Pumpはまず構成ファイルの構成を読み取ります。コマンドラインパラメータの対応する構成が存在する場合、Pumpは構成ファイルのものを上書きします。
        -data-dir string
            Pumpデータが保存されるパス
        -gc int
            Pumpでデータを保持する日数 ("7"がデフォルト)
        -heartbeat-interval int
            PumpがPDに送信するハートビートの間隔 (秒単位)
        -log-file string
            ログのファイルパス
        -log-rotate string
            ログの切り替え頻度 (時間/日)
        -metrics-addr string
            Prometheus Pushgatewayアドレス。設定されていない場合、監視メトリクスの報告は禁止されます。
        -metrics-interval int
            監視メトリクスの報告頻度 ("15"がデフォルト、秒単位)
        -node-id string
            Pumpノードの一意のID。このIDを指定しない場合、システムはホスト名とリッスンポートに基づいて自動的にIDを生成します。
        -pd-urls string
            PDクラスターノードのアドレス (-pd-urls="http://192.168.0.16:2379,http://192.168.0.15:2379,http://192.168.0.14:2379")
        -fake-binlog-interval int
            Pumpノードが偽のbinlogを生成する頻度 ("3"がデフォルト、秒単位)
        ```

    - 例として、"192.168.0.11"にPumpをデプロイする場合、Pumpの構成ファイルは次のようになります:

        ```toml
        # Pump構成

        # Pumpのバウンドアドレス
        addr = "192.168.0.11:8250"

        # Pumpがサービスを提供するためのアドレス
        advertise-addr = "192.168.0.11:8250"

        # Pumpでデータを保持する日数 ("7"がデフォルト)
        gc = 7

        # Pumpデータが保存されるディレクトリ
        data-dir = "data.pump"

        # PumpがPDに送信するハートビートの間隔 (秒単位)
        heartbeat-interval = 2

        # PDクラスターノードのアドレス (それぞれがカンマで区切られ、空白はありません)
        pd-urls = "http://192.168.0.16:2379,http://192.168.0.15:2379,http://192.168.0.14:2379"

        # [security]
        # 通常、特別なセキュリティ設定が必要ない場合は、このセクションはコメントアウトされます。
        # クラスターに接続された信頼されたSSL CAのリストを含むファイルパス。
        # ssl-ca = "/path/to/ca.pem"
        # クラスターに接続されたPEM形式のX509証明書へのパス。
        # ssl-cert = "/path/to/drainer.pem"
        # クラスターに接続されたPEM形式のX509キーへのパス。
        # ssl-key = "/path/to/drainer-key.pem"

        # [storage]
        # binlogデータがディスクにフラッシュされて信頼性が保証されるようにするにはtrue（デフォルト）に設定します
        # sync-log = true

        # 利用可能なディスク容量が設定値よりも少ない場合、Pumpはデータの書き込みを停止します。
        # 42 MB -> 42000000, 42 mib -> 44040192
        # デフォルト: 10 gib
        # stop-write-at-available-space = "10 gib"
        # Pumpに埋め込まれたLSM DBの設定。この部分をよく理解していない場合は、通常、コメントアウトされています。
        # [storage.kv]
        # block-cache-capacity = 8388608
        # block-restart-interval = 16
        # block-size = 4096
        # compaction-L0-trigger = 8
        # compaction-table-size = 67108864
        # compaction-total-size = 536870912
        # compaction-total-size-multiplier = 8.0
        # write-buffer = 67108864
        # write-L0-pause-trigger = 24
        # write-L0-slowdown-trigger = 17
        ```

    - Pumpを起動する例:

        {{< copyable "shell-regular" >}}

        ```bash
        ./pump -config pump.toml
        ```

        コマンドラインパラメータの値が構成ファイルのパラメータと同じ場合、コマンドラインパラメータの値が使用されます。

2. バイナリを使用してDrainerをデプロイする。

    - Drainerのコマンドラインパラメータを表示するには、`./drainer -help`を実行します:

        ```bash
            使用法: Drainer:
                -L string
                    ログの出力情報レベル: debug、info、warn、error、fatal ("info"がデフォルト)
                -V
                    バージョン情報を表示
                -addr string
                    Drainerがサービスを提供するためのアドレス (-addr="192.168.0.13:8249")
                -c int
                    レプリケーションの下流の並行性の数。値が大きいほど並行性のスループット性能が向上します ("1"がデフォルト)。
                -cache-binlog-count int
                    キャッシュに格納されるbinlogアイテムの数の制限 ("8"がデフォルト)
                    上流の大きな単一のbinlogアイテムがDrainerでOOMを引き起こす場合、このパラメータの値を下げてメモリ使用量を減らすことを試みてください。
                -config string
                    構成ファイルのディレクトリ。Drainerはまず構成ファイルを読み取ります。
                    コマンドラインパラメータの対応する構成が存在する場合、Drainerは構成ファイルのものを上書きします。
                -data-dir string
                    Drainerデータが格納されるディレクトリ ("data.drainer"がデフォルト)
                -dest-db-type string
                    Drainerの下流サービスタイプ
                    値は"mysql"、"tidb"、"kafka"、"file"にすることができます ("mysql"がデフォルト)
                -detect-interval int
        ```
            オンライン Pump をPDで確認する間隔（デフォルトで"10"秒）
        -disable-detect
            衝突モニタリングを無効にするかどうか
        -disable-dispatch
            単一のbinlogファイルを分割するSQL機能を無効にするかどうか。“true”に設定されている場合、各binlogファイルはbinlogの順序に従って単一のトランザクションに復元されます。下流がMySQLの場合、"False"に設定されます。
        -ignore-schemas string
            DBフィルタリスト（デフォルトで"INFORMATION_SCHEMA,PERFORMANCE_SCHEMA,mysql,test"）
            '無視スキーマ'テーブルのリネームDDL操作はサポートされていません。
        -initial-commit-ts
            Drainerに関連するブレークポイント情報がない場合、このパラメータを使用して関連するブレークポイント情報を構成できます。（デフォルトで"-1"）
            このパラメータの値が`-1`の場合は、DrainerはPDから最新のタイムスタンプを自動的に取得します。
        -log-file string
            ログファイルのパス
        -log-rotate string
            ログファイルの切り替え頻度、hour/day
        -metrics-addr string
            Prometheus Pushgatewayアドレス
            設定されていない場合、モニタリングメトリクスは報告されません。
        -metrics-interval int
            モニタリングメトリクスの報告頻度（デフォルトで"15"秒）
        -node-id string
            DrainerノードのユニークID。このIDを指定しない場合、システムはホスト名とリッスンポートに基づいて自動的にIDを生成します。
        -pd-urls string
            PDクラスターノードのアドレス（-pd-urls="http://192.168.0.16:2379,http://192.168.0.15:2379,http://192.168.0.14:2379"）
        -safe-mode
            安全モードを有効にするかどうか。これにより、データを下流のMySQL/TiDBに繰り返し書き込むことができます。
            このモードは、`INSERT`ステートメントを`REPLACE`ステートメントに置き換え、`UPDATE`ステートメントを`DELETE`に置き換えて分割します。
        -txn-batch int
            トランザクションのSQLステートメントの数（デフォルトで"1"）
```
        # ブローカーリクエスト内のメッセージの最大数（binlogの数）。空白のままにするか、0よりも小さい値を設定した場合、デフォルト値1024が使用されます。
        # kafka-max-messages = 1024
        # ブローカーリクエストの最大サイズ（単位: バイト）。デフォルト値は1 GiBで、最大値は2 GiBです。
        # kafka-max-message-size = 1073741824

        # binlogデータを保存するKafkaクラスターのトピック名。デフォルト値は<cluster-id>_obinlogです。
        # 同じKafkaクラスターに複数のDrainerを実行してデータをレプリケートするには、各Drainerに異なる`topic-name`を設定する必要があります。
        # topic-name = ""
        ```

    - Drainerを起動する:

        > **注意:**
        >
        > 下流がMySQL/TiDBの場合、データの整合性を保証するために`initial-commit-ts`の値を取得し、Drainerを初めて起動する前にデータの完全バックアップを取得し、データをリストアする必要があります。

        Drainerを初めて起動する場合は、`initial-commit-ts`パラメータを使用します。

        {{< copyable "shell-regular" >}}

        ```bash
        ./drainer -config drainer.toml -initial-commit-ts {initial-commit-ts}
        ```

        コマンドラインパラメータと構成ファイルパラメータが同じ場合、コマンドラインのパラメータ値が使用されます。

3. TiDBサーバーを起動する:

    - PumpとDrainerを起動した後、TiDBサーバーをbinlogを有効にして起動するには、TiDBサーバーの構成ファイルにこのセクションを追加します:

        ```
        [binlog]
        enable=true
        ```

    - TiDBサーバーは、PDから登録されたPumpのアドレスを取得し、それら全てにデータをストリーム配信します。登録されたPumpのインスタンスが存在しない場合、TiDBサーバーは起動を拒否するか、Pumpのインスタンスがオンラインになるまで起動をブロックします。

> **注意:**
>
> - TiDBが実行されている場合、少なくとも1つのPumpが正常に実行されていることを保証する必要があります。
> - TiDBサーバーでTiDB Binlogサービスを有効にするには、TiDBで起動パラメータ`-enable-binlog`を使用するか、TiDBサーバーの構成ファイルの[binlog]セクションにenable=trueを追加してください。
> - TiDBクラスター内の全てのTiDBインスタンスでTiDB Binlogサービスを有効にする必要があります。そうしないと、上流と下流のデータの整合性が損なわれる可能性があります。TiDB Binlogサービスを有効にしていないTiDBインスタンスで一時的に実行する場合は、TiDB構成ファイルで`run_ddl=false`を設定してください。
> - Drainerは、`ignore schemas`（フィルタリスト内のスキーマ）のテーブルでの`rename`DDL操作をサポートしていません。
> - 既存のTiDBクラスターでDrainerを起動する場合は、通常、クラスターデータの完全バックアップを作成し、**スナップショットタイムスタンプ**を取得し、データをターゲットデータベースにインポートし、その後Drainerを起動して対応する**スナップショットタイムスタンプ**から増分データをレプリケートする必要があります。
> - 下流のデータベースがTiDBまたはMySQLの場合、上流と下流のデータベースの`sql_mode`が一致していることを確認してください。つまり、上流で実行される各SQL文が下流にレプリケートされるときには、`sql_mode`が同じである必要があります。上流と下流でそれぞれ`select @@sql_mode;`ステートメントを実行して、`sql_mode`を比較してください。
> - 上流でサポートされているDDLステートメントが下流と互換性がない場合、Drainerはデータをレプリケートできません。例として、下流のデータベースMySQLがInnoDBエンジンを使用している場合に`CREATE TABLE t1(a INT) ROW_FORMAT=FIXED;`ステートメントをレプリケートする場合が挙げられます。この場合、[Drainerでトランザクションをスキップ](/tidb-binlog/tidb-binlog-faq.md#what-can-i-do-when-some-ddl-statements-supported-by-the-upstream-database-cause-error-when-executed-in-the-downstream-database)を構成し、下流のデータベースで互換性のあるステートメントを手動で実行してください。