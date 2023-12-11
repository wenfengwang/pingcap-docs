---
title: PD構成ファイル
summary: PD構成ファイルについて学ぶ
aliases: ['/docs/dev/pd-configuration-file/','/docs/dev/reference/configuration/pd-server/configuration-file/']
---

# PD構成ファイル

<!-- markdownlint-disable MD001 -->

PD構成ファイルは、コマンドラインパラメータよりも多くのオプションをサポートしています。デフォルトの構成ファイルは[こちら](https://github.com/pingcap/pd/blob/master/conf/config.toml)で見つけることができます。

このドキュメントは、コマンドラインパラメータに含まれていないパラメータのみを説明します。コマンドラインパラメータについては[こちら](/command-line-flags-for-pd-configuration.md)を確認してください。

> **ヒント:**
>
> 構成項目の値を調整する必要がある場合は、[構成の変更](/maintain-tidb-using-tiup.md#modify-the-configuration)を参照してください。

### `name`

- PDノードのユニークな名前
- デフォルト値: `"pd"`
- 複数のPDノードを起動する場合は、各ノードにユニークな名前を使用してください。

### `data-dir`

- PDがデータを格納するディレクトリ
- デフォルト値: `default.${name}"`

### `client-urls`

- PDが受け取るクライアントのURLのリスト
- デフォルト値: `"http://127.0.0.1:2379"`
- クラスタを展開する場合、`client-urls`として現在のホストのIPアドレスを指定する必要があります（例: `"http://192.168.100.113:2379"`）。クラスタがDocker上で実行される場合は、DockerのIPアドレスを`"http://0.0.0.0:2379"`として指定してください。

### `advertise-client-urls`

- クライアントがPDにアクセスするための広告URLのリスト
- デフォルト値: `"${client-urls}"`
- 一部の状況（例: DockerやNATネットワーク環境）では、クライアントがPDにデフォルトのクライアントURLでアクセスできない場合、広告クライアントURLを手動で設定する必要があります。
- 例えば、Dockerの内部IPアドレスが`172.17.0.1`であり、ホストのIPアドレスが`192.168.100.113`でポートマッピングが`-p 2380:2380`に設定されている場合、`advertise-client-urls`を`"http://192.168.100.113:2380"`に設定できます。クライアントはこのサービスを`"http://192.168.100.113:2380"`を介して見つけることができます。

### `peer-urls`

- PDノードが受け取るピアのURLのリスト
- デフォルト値: `"http://127.0.0.1:2380"`
- クラスタを展開する場合、`peer-urls`を現在のホストのIPアドレス（例: `"http://192.168.100.113:2380"`）として指定する必要があります。クラスタがDocker上で実行される場合は、DockerのIPアドレスを`"http://0.0.0.0:2380"`として指定してください。

### `advertise-peer-urls`

- 他のPDノード（ピア）がPDノードにアクセスするための広告URLのリスト
- デフォルト: `"${peer-urls}"`
- 一部の状況（例: DockerやNATネットワーク環境）では、他のノード（ピア）がこのPDノードにデフォルトのピアURLでアクセスできない場合、広告ピアURLを手動で設定する必要があります。
- 例えば、Dockerの内部IPアドレスが`172.17.0.1`であり、ホストのIPアドレスが`192.168.100.113`でポートマッピングが`-p 2380:2380`に設定されている場合、`advertise-peer-urls`を`"http://192.168.100.113:2380"`に設定できます。他のPDノードはこのサービスを`"http://192.168.100.113:2380"`を介して見つけることができます。

### `initial-cluster`

- ブートストラップのための初期クラスタ構成
- デフォルト値: `"{name}=http://{advertise-peer-url}"`
- 例えば、`name`が"pd"で、`advertise-peer-urls`が`"http://192.168.100.113:2380"`である場合、`initial-cluster`は`"pd=http://192.168.100.113:2380"`になります。
- 3つのPDサーバを起動する必要がある場合、`initial-cluster`は次のようになるかもしれません:

    ```
    pd1=http://192.168.100.113:2380, pd2=http://192.168.100.114:2380, pd3=192.168.100.115:2380
    ```

### `initial-cluster-state`

+ クラスタの初期状態
+ デフォルト値: `"new"`

### `initial-cluster-token`

+ ブートストラップフェーズ中に異なるクラスタを識別するトークン
+ デフォルト値: `"pd-cluster"`
+ 同じ構成を持つ複数のクラスタを順次展開する場合、異なるトークンを指定して異なるクラスタノードを分離する必要があります。

### `lease`

+ PDリーダーキーのリースタイムアウト。タイムアウト後、システムは新しいリーダーを選出します。
+ デフォルト値: `3`
+ 単位: 秒

### `quota-backend-bytes`

+ メタ情報データベースのストレージサイズ。デフォルトでは8GiBです。
+ デフォルト値: `8589934592`

### `auto-compaction-mod`

+ メタ情報データベースの自動圧縮モード
+ 利用可能なオプション: `periodic`（サイクルごと）と`revision`（バージョン番号ごと）。
+ デフォルト値: `periodic`

### `auto-compaction-retention`

+ `auto-compaction-retention`が`periodic`の場合、メタ情報データベースの自動圧縮の時間間隔。圧縮モードが`revision`に設定されている場合、このパラメータは自動圧縮のバージョン番号を示します。
+ デフォルト値: 1時間

### `force-new-cluster`

+ PDを新しいクラスタとして開始し、Raftメンバーの数を`1`に変更するかどうかを決定します。
+ デフォルト値: `false`

### `tso-update-physical-interval`

+ PDがTSOの物理時間を更新する間隔。
+ TSO物理時間のデフォルト更新間隔では、PDは最大で262144のTSOを提供します。より多くのTSOを取得するには、この構成項目の値を減らすことができます。最小値は`1ms`です。
+ この構成項目を減らすと、PDのCPU使用率が増加する可能性があります。テストによると、更新間隔が`50ms`の場合、`1ms`の間隔ではPDの[CPU使用率](https://man7.org/linux/man-pages/man1/top.1.html)が約10%増加します。
+ デフォルト値: `50ms`
+ 最小値: `1ms`

## pd-server

pd-serverに関連する構成項目

### `server-memory-limit` <span class="version-mark">v6.6.0で新規</span>

> **警告:**
>
> この構成は実験的な機能です。本番環境で使用することは推奨されません。

+ PDインスタンスのメモリ制限比率。値が`0`の場合、メモリ制限はありません。
+ デフォルト値: `0`
+ 最小値: `0`
+ 最大値: `0.99`

### `server-memory-limit-gc-trigger` <span class="version-mark">v6.6.0で新規</span>

> **警告:**
>
> この構成は実験的な機能です。本番環境で使用することは推奨されません。

+ PDがGCをトリガしようとするしきい値比率。PDのメモリ使用率が`server-memory-limit`の値 * `server-memory-limit-gc-trigger`に達すると、Golang GCがトリガされます。1分間に1回のみGCがトリガされます。
+ デフォルト値: `0.7`
+ 最小値: `0.5`
+ 最大値: `0.99`

### `enable-gogc-tuner` <span class="version-mark">v6.6.0で新規</span>

> **警告:**
>
> この構成は実験的な機能です。本番環境で使用することは推奨されません。

+ GOGC Tunerを有効にするかどうかを制御します。
+ デフォルト値: `false`

### `gc-tuner-threshold` <span class="version-mark">v6.6.0で新規</span>

> **警告:**
>
> この構成は実験的な機能です。本番環境で使用することは推奨されません。

+ GOGCをチューニングするための最大メモリしきい値比率。メモリがこのしきい値を超えると、つまり`server-memory-limit`の値 * `gc-tuner-threshold`に達すると、GOGC Tunerは動作を停止します。
+ デフォルト値: `0.6`
+ 最小値: `0`
+ 最大値: `0.9`

### `flow-round-by-digit` <span class="version-mark">TiDB 5.1で新規</span>

+ デフォルト値: 3
+ PDはRegionフロー情報の変更によって引き起こされる統計情報の更新を減らすために、フロー番号の最下位桁を丸めます。この構成項目は、Regionフロー情報の最下位桁を丸める数を指定するために使用されます。例えば、フローが`100512`の場合、デフォルト値が`3`であるため、フローは`101000`に丸められます。この構成は`trace-region-flow`を置き換えます。

> **注意:**
>
> クラスタをTiDB 4.0バージョンから現行バージョンにアップグレードした場合、アップグレード後の `flow-round-by-digit` の動作とアップグレード前の `trace-region-flow` の動作がデフォルトで一貫しています。これは、アップグレード前の `trace-region-flow` の値が `false` である場合、アップグレード後の `flow-round-by-digit` の値が127であり、アップグレード前の `trace-region-flow` の値が `true` である場合、アップグレード後の `flow-round-by-digit` の値が `3` であることを意味します。

### `min-resolved-ts-persistence-interval` <span class="version-mark">v6.0.0で新規</span>

+ PDに最小解決タイムスタンプが永続的に保存される間隔を決定します。この値が `0` に設定されている場合、永続性が無効になります。
+ デフォルト値: v6.3.0より前は、デフォルト値は`"0s"`でした。v6.3.0以降では、デフォルト値は最小の正の値である`"1s"`になりました。
+ 最小値: `0`
+ 単位: 秒

> **注意:**
>
> v6.0.0〜v6.2.0からアップグレードされたクラスタの場合、`min-resolved-ts-persistence-interval` のデフォルト値はアップグレード後も変わらず、`"0s"`のままとなります。この機能を有効にするには、この構成項目の値を手動で変更する必要があります。

## security

セキュリティに関連する構成項目

### `cacert-path`

+ CAファイルのパス
+ デフォルト値: ""

### `cert-path`

+ X509証明書を含むPrivacy Enhanced Mail (PEM)ファイルのパス
+ デフォルト値: ""

### `key-path`

+ X509キーを含むPEMファイルのパス
+ デフォルト値: ""

### `redact-info-log` <span class="version-mark">v5.0で新規</span>

+ PDログでのログリダクションを有効にするかどうかを制御します
+ 構成値を`true`に設定すると、PDログ内でユーザーデータがリダクションされます。
+ デフォルト値: `false`

## `log`

ログに関連する構成項目

### `level`

+ 出力ログのレベルを指定します
+ オプションの値: `"debug"`, `"info"`, `"warn"`, `"error"`, `"fatal"`
+ デフォルト値: `"info"`

### `format`

+ ログのフォーマット
+ オプションの値: `"text"`, `"json"`
+ デフォルト値: `"text"`

### `disable-timestamp`

+ ログで自動生成されたタイムスタンプを無効にするかどうか
+ デフォルト値: `false`

## `log.file`

ログファイルに関連する構成項目

### `max-size`

+ 単一ログファイルの最大サイズ。この値を超えると、システムは自動的にログを複数のファイルに分割します。
+ デフォルト値: `300`
+ 単位: MiB
+ 最小値: `1`

### `max-days`

+ ログを保持する最大日数
+ 構成項目が設定されていないか、その値がデフォルト値0に設定されている場合、PDはログファイルをクリーンアップしません。
+ デフォルト値: `0`

### `max-backups`

+ 保持するログファイルの最大数
+ 構成項目が設定されていないか、その値がデフォルト値0に設定されている場合、PDはすべてのログファイルを保持します。
+ デフォルト値: `0`

## `metric`

監視に関連する構成項目

### `interval`

+ 監視メトリクスデータをPrometheusにプッシュする間隔
+ デフォルト値: `15s`

## `schedule`

スケジューリングに関連する構成項目

（以下略）
> **注意:**
>
> クラスタをTiDB 4.0バージョンから現在のバージョンにアップグレードした場合、新しいフォーミュラバージョンは、アップグレード前後の一貫したPDの動作を保証するために、デフォルトで自動的に無効になります。フォーミュラバージョンを変更する場合は、`pd-ctl`の設定を手動で切り替える必要があります。詳細については、[PD Control](/pd-control.md#config-show--set-option-value--placement-rules)を参照してください。

### `store-limit-version` <span class="version-mark">v7.1.0で追加</span>

> **警告:**
>
> この構成項目を`"v2"`に設定することは実験的な機能です。本番環境での使用は推奨されません。

+ ストア制限フォーミュラのバージョンを制御します
+ デフォルト値: `v1`
+ 値のオプション:
    + `v1`: v1モードでは、`ストア制限`を手動で変更して、単一のTiKVのスケジューリング速度を制限できます。
    + `v2`: (実験的な機能) v2モードでは、`ストア制限`の値を手動で設定する必要はなく、PDがTiKVスナップショットの能力に基づいて動的に調整します。詳細については、[ストア制限v2の原則](/configure-store-limit.md#principles-of-store-limit-v2)を参照してください。

### `enable-joint-consensus` <span class="version-mark">v5.0で追加</span>

+ レプリカのスケジューリングにジョイントコンセンサスを使用するかどうかを制御します。この構成が無効になっている場合、PDは1回に1つのレプリカをスケジュールします。
+ デフォルト値: `true`

### `hot-regions-write-interval` <span class="version-mark">v5.4.0で追加</span>

+ PDがホットリージョン情報を保存する時間間隔です。
+ デフォルト値: `10m`

> **注意:**
>
> ホットリージョンの情報は3分ごとに更新されます。間隔が3分未満の場合、間隔中の更新は意味をなさない可能性があります。

### `hot-regions-reserved-days` <span class="version-mark">v5.4.0で追加</span>

+ ホットリージョン情報が保持される日数を指定します。
+ デフォルト値: `7`

## `replication`

レプリカに関連する構成項目

### `max-replicas`

+ レプリカの数、つまりリーダーとフォロワーの数の合計です。デフォルト値`3`は1つのリーダーと2つのフォロワーを意味します。この構成が動的に変更されると、PDは後ろでリージョンをスケジュールして、レプリカの数がこの構成に合致するようにします。
+ デフォルト値: `3`

### `location-labels`

+ TiKVクラスタのトポロジ情報
+ デフォルト値: `[]`
+ [クラスタートポロジ構成](/schedule-replicas-by-topology-labels.md)

### `isolation-level`

+ TiKVクラスタの最小の地理的分離レベル
+ デフォルト値: `""`
+ [クラスタートポロジ構成](/schedule-replicas-by-topology-labels.md)

### `strictly-match-label`

+ TiKVのラベルがPDの`location-labels`に一致するかどうかを厳密にチェックするかどうかを制御します。
+ デフォルト値: `false`

### `enable-placement-rules`

+ `placement-rules`を有効にします。
+ デフォルト値: `true`
+ 詳細については、[配置ルール](/configure-placement-rules.md)を参照してください。

## `label-property`

ラベルに関連する構成項目

### `key`

+ リーダーを拒否したストアのラベルキー
+ デフォルト値: `""`

### `value`

+ リーダーを拒否したストアのラベル値
+ デフォルト値: `""`

## `dashboard`

PDに組み込まれた[TiDB Dashboard](/dashboard/dashboard-intro.md)に関連する構成項目

### `tidb-cacert-path`

+ ルートCA証明書ファイルのパス。TLSを使用してTiDBのSQLサービスに接続する場合は、このパスを設定できます。
+ デフォルト値: `""`

### `tidb-cert-path`

+ SSL証明書ファイルのパス。TLSを使用してTiDBのSQLサービスに接続する場合は、このパスを設定できます。
+ デフォルト値: `""`

### `tidb-key-path`

+ SSLプライベートキーファイルのパス。TLSを使用してTiDBのSQLサービスに接続する場合は、このパスを構成できます。
+ デフォルト値: `""`

### `public-path-prefix`

+ TiDB Dashboardがリバースプロキシの背後でアクセスされる場合、この項目はすべてのWebリソースの公開URLパス接頭辞を設定します。
+ デフォルト値: `/dashboard`
+ TiDB Dashboardがリバースプロキシの背後でアクセスされない場合は、この構成項目を変更しないでください。そうしないと、アクセスの問題が発生する可能性があります。詳細については、[リバースプロキシの背後でTiDB Dashboardを使用する](/dashboard/dashboard-ops-reverse-proxy.md)を参照してください。

### `enable-telemetry`

+ TiDB Dashboardでテレメトリ収集機能を有効にするかどうかを決定します。
+ デフォルト値: `false`
+ 詳細については、[テレメトリ](/telemetry.md)を参照してください。

## `replication-mode`

全リージョンのレプリケーションモードに関連する構成項目。詳細については、[DR Auto-Syncモードを有効にする](/two-data-centers-in-one-city-deployment.md#enable-the-dr-auto-sync-mode)を参照してください。

## Controllor

このセクションでは、PDに組み込まれた[Resource Control](/tidb-resource-control.md)の構成項目について説明します。

### `degraded-mode-wait-duration`

+ 劣化モードをトリガーするまでの待ち時間。劣化モードとは、ローカルトークンバケット（LTB）とグローバルトークンバケット（GTB）が失われた場合、LTBがデフォルトのリソースグループ構成にフォールバックし、それ以上GTBの認証トークンを持たなくなることであり、ネットワークの分離や異常が発生した場合にサービスが影響を受けないようにするものです。
+ デフォルト値: 0s
+ 劣化モードはデフォルトで無効になっています。

### `request-unit`

以下は[Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru)に関する構成項目です。

#### `read-base-cost`

+ RUへのリードリクエストへの変換の基本係数
+ デフォルト値: 0.25

#### `write-base-cost`

+ RUへのライトリクエストへの変換の基本係数
+ デフォルト値: 1

#### `read-cost-per-byte`

+ RUへのリードフローへの変換の基本係数
+ デフォルト値: 1/(64 * 1024)
+ 1 RU = 64 KiBのリードバイト

#### `write-cost-per-byte`

+ RUへのライトフローへの変換の基本係数
+ デフォルト値: 1/1024
+ 1 RU = 1 KiBのライトバイト

#### `read-cpu-ms-cost`

+ CPUからRUへの変換の基本係数
+ デフォルト値: 1/3
+ 1 RU = 3ミリ秒のCPU時間