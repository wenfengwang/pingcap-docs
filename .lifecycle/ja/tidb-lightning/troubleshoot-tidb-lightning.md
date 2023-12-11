---
title: TiDB Lightningのトラブルシューティング
summary: TiDB Lightningの使用中に遭遇する可能性のある一般的な問題とそれらの解決策を学びます。
aliases: ['/docs/dev/troubleshoot-tidb-lightning/','/docs/dev/how-to/troubleshoot/tidb-lightning/','/docs/dev/tidb-lightning/tidb-lightning-misuse-handling/','/docs/dev/reference/tools/error-case-handling/lightning-misuse-handling/','/tidb/dev/tidb-lightning-misuse-handling','/tidb/dev/troubleshoot-tidb-lightning']
---

# TiDB Lightningのトラブルシューティング

このドキュメントでは、TiDB Lightningの使用時に遭遇する可能性のある一般的な問題とそれらの解決策についてまとめています。

## インポート速度が遅すぎる

通常、TiDB Lightningが256MBのデータファイルをインポートするのに1つのスレッドあたり2分かかります。もし速度がこれよりも遥かに遅い場合、エラーが発生しています。各データファイルの処理時間は`restore chunk … takes`というログまたはGrafanaのメトリクスから確認できます。

TiDB Lightningの動作が遅くなる理由はいくつかあります：

**原因1**：`region-concurrency`が高すぎるため、スレッド競合が発生しパフォーマンスが低下します。

1. `region-concurrency`の設定はログの先頭から`region-concurrency`を検索することで見つけることができます。
2. もしTiDB Lightningが他のサービス（例: TiKV Importer）と同じマシンを共有する場合、`region-concurrency`はCPUコアの合計数の75%に**手動**で設定する必要があります。
3. CPUにクォータがある場合（例: Kubernetesの設定で制限されている場合）、TiDB Lightningはこれを読み取ることができないかもしれません。この場合も`region-concurrency`を**手動**で減らす必要があります。

**原因2**：テーブルスキーマが複雑すぎる。

追加のインデックスごとに新しいKVペアが各行に導入されます。N個のインデックスがあれば、実際にインポートされるサイズはDumplingの出力サイズの約（N+1）倍になります。もしインデックスが無視できるものであれば、インポートが完了した後にそれらを`CREATE INDEX`で追加することができます。

**原因3**：各ファイルが大きすぎる。

TiDB Lightningは、データソースが256MB程度の複数のファイルに分割されている場合に最も効率的に動作します。各ファイルが大きすぎると、TiDB Lightningが応答しない可能性があります。

データソースがCSVであり、全てのCSVファイルに改行制御文字（U+000AおよびU+000D）を含むフィールドがない場合、TiDB Lightningに大きなファイルを自動的に分割させるために"strict format"をONにすることができます。

```toml
[mydumper]
strict-format = true
```

**原因4**：TiDB Lightningのバージョンが古すぎる。

最新バージョンを試してみてください。新しい速度改善があるかもしれません。

## `tidb-lightning`プロセスがバックグラウンドで突然終了する

これは`tidb-lightning`の起動が不正であり、システムが`tidb-lightning`プロセスを停止するためSIGHUPシグナルを送信した可能性があります。この状況では、`tidb-lightning.log`は通常、以下のログを出力します。

```
[2018/08/10 07:29:08.310 +08:00] [INFO] [main.go:41] ["got signal to exit"] [signal=hangup]
```

`nohup`を直接コマンドラインで使用することは推奨されていません。スクリプトを実行して[ここに記載されている](/tidb-lightning/deploy-tidb-lightning.md#step-3-start-tidb-lightning)ように`tidb-lightning`を開始できます。

また、TiDB Lightningの最後のログが"Context canceled"である場合は、最初の"ERROR"レベルのログを検索する必要があります。"ERROR"レベルのログは通常、"got signal to exit"の後に続くものであり、これはTiDB Lightningが中断シグナルを受け取り、その後終了したことを示しています。

## TiDBクラスターはTiDB Lightningの使用後、多くのCPUリソースを使用し遅く動作する

もし`tidb-lightning`が異常に終了した場合、クラスターは"import mode"でスタックする可能性があります。これは本番環境に適していません。現在のモードは次のコマンドで取得できます。

{{< copyable "shell-regular" >}}

```sh
tidb-lightning-ctl --config tidb-lightning.toml --fetch-mode
```

次のコマンドでクラスターを強制的に"normal mode"に戻すことができます。

{{< copyable "shell-regular" >}}

```sh
tidb-lightning-ctl --config tidb-lightning.toml --fetch-mode
```

## TiDB Lightningがエラーを報告する

### `could not find first pair, this shouldn't happen`

このエラーは、TiDB Lightningがソートされたローカルファイルを読み取る際に、オープンされたファイルの数がシステムの制限を超えたために発生する可能性があります。Linuxシステムでは、`ulimit -n`コマンドを使用してこのシステムの制限値が十分大きくないか確認できます。インポート中はこの値を`1000000`に調整することをお勧めします（`ulimit -n 1000000`）。

### `checksum failed: checksum mismatched remote vs local`

**原因**：ローカルデータソースのテーブルとリモートインポートされたデータベースのチェックサムが異なる。このエラーにはいくつかの深刻な原因があります。`checksum mismatched`を含むログをチェックしてさらなる原因を特定できます。

`checksum mismatched`を含む行には`total_kvs: x vs y`という情報があり、`x`はインポート完了後にターゲットクラスターによって計算されたキー-バリューペア（KVペア）の数を、`y`はローカルデータソースで生成されたキー-バリューペアの数を示します。

- もし`x`が大きい場合、ターゲットクラスターによりKVペアがより多いことを意味します。
    - インポート前にこのテーブルが空でない可能性があり、データのチェックサムに影響を及ぼす可能性があります。また、TiDB Lightningが以前に失敗し正常にリスタートされなかった可能性もあります。
- もし`y`が大きい場合、ローカルデータソースによりKVペアがより多いことを意味します。
    - ターゲットデータベースのチェックサムが全て0であれば、インポートが行われていない可能性があります。クラスターがデータを受け取るのに忙しすぎる可能性があります。
    - エクスポートされたデータに重複したデータが含まれる可能性があります。例えば、値が重複するUNIQUEおよびPRIMARY KEY、またはデータが大文字小文字を区別しないがテーブル構造が大文字小文字を区別する場合などです。
- その他の可能な原因
    - もしデータソースが自動生成され、Dumplingによってバックアップされていない場合、データがテーブルの制限に準拠していることを確認してください。例えば、AUTO_INCREMENT列は正数であり0ではない必要があります。

**解決策**：

1. `tidb-lightning-ctl`を使用して破損データを削除し、テーブル構造とデータを確認し、影響を受けたテーブルを再度インポートするためにTiDB Lightningを再起動してください。

    {{< copyable "shell-regular" >}}

    ```sh
    tidb-lightning-ctl --config conf/tidb-lightning.toml --checkpoint-error-destroy=all
    ```

2. ターゲットデータベースの負荷を軽減するために、[checkpoint] dsnを変更してチェックポイントを外部データベースに保存してください。

3. TiDB Lightningが適切に再起動されていない場合は、FAQの"[How to properly restart TiDB Lightning](/tidb-lightning/tidb-lightning-faq.md#how-to-properly-restart-tidb-lightning)"セクションも参照してください。

### `Checkpoint for … has invalid status:` (error code)

**原因**：[Checkpoint](/tidb-lightning/tidb-lightning-checkpoints.md)が有効になっており、TiDB LightningまたはTiKV Importerが以前に異常に終了しています。データの破損を防ぐために、エラーが解消されるまでTiDB Lightningを開始することはできません。

エラーコードは25未満の整数で、0、3、6、9、12、14、15、17、18、20、21の可能な値があります。この整数はインポートプロセスで予期しない終了が発生したステップを示します。整数が大きいほど、終了が遅いステップになります。

**解決策**：

もしエラーが無効なデータソースによって引き起こされた場合、「tidb-lightning-ctl」を使用してインポートされたデータを削除し、再びLightningを開始してください。

```sh
tidb-lightning-ctl --config conf/tidb-lightning.toml --checkpoint-error-destroy=all
```

その他のオプションについては、[Checkpoints control](/tidb-lightning/tidb-lightning-checkpoints.md#checkpoints-control)セクションを参照してください。

### `cannot guess encoding for input file, please convert to UTF-8 manually`

**原因**：TiDB Lightningはテーブルスキーマに対してUTF-8およびGB-18030のエンコーディングのみを認識します。もしファイルがこれらのエンコーディングのいずれにも該当しない場合、または歴史的な`ALTER TABLE`の実行により、ファイルにUTF-8とGB-18030の両方のエンコーディングが含まれている場合、このエラーが発生します。

**解決策**：

1. ファイルが完全にUTF-8またはGB-18030になるようにスキーマを修正してください。

2. ターゲットデータベースで影響を受けるテーブルを手動で`CREATE`してください。

3. `[mydumper] character-set = "binary"`を設定してチェックをスキップしてください。ただし、これによってターゲットデータベースに文字化けが生じる可能性があります。
### `[sql2kv] sql エンコードエラー = [types:1292]時間の形式が無効です: '{1970 1 1 …}'`

**原因**: テーブルに`timestamp`型のカラムが含まれていますが、時間の値そのものが存在しません。これは、サマータイムの変更や時間の値がサポートされている範囲を超えたためです (1970年1月1日から2038年1月19日)。

**解決策**:

1. TiDB Lightningとソースデータベースが同じタイムゾーンを使用していることを確認してください。

    TiDB Lightningを直接実行する場合、`$TZ`環境変数を使用してタイムゾーンを強制することができます。

    ```sh
    # 手動デプロイメント、かつAsia/Shanghaiに強制する。
    TZ='Asia/Shanghai' bin/tidb-lightning -config tidb-lightning.toml
    ```

2. クラスタ全体が同じかつ最新バージョンの`tzdata` (2018年i以上のバージョン)を使用していることを確認してください。

    CentOSでは、`yum info tzdata`を実行してインストール済みのバージョンと更新の有無を確認します。`yum upgrade tzdata`を実行してパッケージを更新します。

### `[Error 8025: entry too large, the max entry size is 6291456]`

**原因**: TiDB Lightningによって生成された1つのキー-値ペアの単一行が、TiDBで設定された制限を超えています。

**解決策**:

現在、TiDBの制限をバイパスすることはできません。他のテーブルのインポートが成功するようにするためには、このテーブルを無視するしかありません。

### TiDB Lightningがモードを切り替える際に`rpc error: code = Unimplemented ...`が発生する

**原因**: クラスタ内の一部のノードが`switch-mode`をサポートしていないことです。たとえば、TiFlashのバージョンが`v4.0.0-rc.2`より前の場合、[`switch-modeがサポートされていません`](https://github.com/pingcap/tidb-lightning/issues/273)。

**解決策**:

- クラスタにTiFlashノードがある場合、クラスタを`v4.0.0-rc.2`またはそれ以上のバージョンに更新してください。
- クラスタをアップグレードしたくない場合は、一時的にTiFlashを無効にしてください。

### `tidb lightning encountered error: TiDB version too old, expected '>=4.0.0', found '3.0.18'`

TiDB Lightning Local-backendは、TiDBクラスタのv4.0.0以降のバージョンへのデータインポートのみをサポートしています。Local-backendを使用してv2.xまたはv3.xクラスタにデータをインポートしようとすると、上記のエラーが報告されます。この場合は、構成を変更してImporter-backendまたはTiDB-backendを使用してデータをインポートすることができます。

一部の`nightly`バージョンはv4.0.0-beta.2と似ているかもしれません。これらの`nightly`バージョンのTiDB Lightningは実際にLocal-backendをサポートしています。`nightly`バージョンを使用している場合、このエラーが発生した場合は、構成`check-requirements = false`を設定してバージョンのチェックをスキップできます。ただし、このパラメータを設定する前に、TiDB Lightningの構成が対応するバージョンをサポートしていることを確認してください。それ以外の場合、インポートが失敗する可能性があります。

### `restore table test.district failed: unknown columns in header [...]`

通常、このエラーは、CSVデータファイルにヘッダーが含まれていないために発生します (最初の行が列名ではなくデータです)。そのため、TiDB Lightning構成ファイルに次の構成を追加する必要があります:

```
[mydumper.csv]
header = false
```

### `Unknown character set`

TiDBはすべてのMySQLの文字セットをサポートしていません。したがって、インポート中にテーブルスキーマ作成時にサポートされていない文字セットが使用されると、TiDB Lightningはこのエラーを報告します。このエラーをバイパスするには、特定のデータに応じて、TiDBでサポートされている[文字セット](/character-set-and-collation.md)を使用して下流で事前にテーブルスキーマを作成することができます。

### `invalid compression type ...`

- TiDB Lightning v6.4.0以降のバージョンでは、次の圧縮されたデータファイルのみがサポートされています: `gzip`、`snappy`、`zstd`。それ以外の種類の圧縮ファイルはエラーを起こします。サポートされていない圧縮ファイルがインポートデータファイルが格納されているディレクトリに存在する場合、このタスクはエラーを報告します。これらのサポートされていないファイルをインポートデータディレクトリから移動して、このようなエラーを回避することができます。詳細については、[圧縮ファイル](/tidb-lightning/tidb-lightning-data-source.md#compressed-files)を参照してください。

> **注意:**
>
> Snappy圧縮ファイルは、[公式のSnappy形式](https://github.com/google/snappy)である必要があります。その他のSnappy圧縮形式はサポートされていません。