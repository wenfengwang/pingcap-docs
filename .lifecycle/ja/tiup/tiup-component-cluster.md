---
title: TiUPクラスタ
---

# TiUPクラスタ

TiUPクラスタはGolangで記述されたTiUPのクラスタ管理コンポーネントです。TiUPクラスタコンポーネントを使用して、デプロイ、起動、シャットダウン、破棄、弾性スケーリング、TiDBクラスタのアップグレード、およびTiDBクラスタのパラメータ管理など、日常の運用および保守を行うことができます。

## 構文

```shell
tiup cluster [command] [flags]
```

`[command]`はコマンドの名前です。サポートされるコマンドについては、以下の[コマンドリスト](#コマンドリスト)を参照してください。

## オプション

### --ssh

- コマンドの実行のためにリモート端末（TiDBサービスがデプロイされたマシン）に接続するためのSSHクライアントを指定します。
- データ型: `STRING`
- サポートされる値:

    - `builtin`: tiup-clusterに組み込まれたeasysshクライアントをSSHクライアントとして使用します。
    - `system`: 現在のオペレーティングシステムのデフォルトのSSHクライアントを使用します。
    - `none`: SSHクライアントは使用されません。デプロイメントは現在のマシンのみに対して行われます。

- このオプションがコマンドで指定されていない場合、デフォルト値として`builtin`が使用されます。

### --ssh-timeout

- SSH接続のタイムアウトを秒単位で指定します。
- データ型: `UINT`
- このオプションがコマンドで指定されていない場合、デフォルトのタイムアウトは`5`秒です。

### --wait-timeout

- 各ステップでの操作プロセスの最大待機時間（秒単位）を指定します。操作プロセスにはsystemctlを指定してサービスを開始または停止したり、ポートがオンラインまたはオフラインになるのを待ったりするステップが多数含まれます。各ステップには数秒かかる場合があります。ステップの実行時間が指定されたタイムアウトを超えると、ステップはエラーで終了します。
- データ型: `UINT`
- このオプションがコマンドで指定されていない場合、各ステップの最大待機時間は`120`秒です。

### -y, --yes

- すべてのリスクのある操作の二次確認をスキップします。このオプションの使用は推奨されませんが、TiUPを呼び出すスクリプトを使用する場合を除きます。
- このオプションは`false`値でデフォルトで無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、任意の値を渡さないでください。

### -v, --version

- TiUPクラスタの現在のバージョンを出力します。
- データ型: `BOOLEAN`
- このオプションは`false`値でデフォルトで無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、任意の値を渡さないでください。

### -h, --help

- 関連するコマンドのヘルプ情報を出力します。
- データ型: `BOOLEAN`
- このオプションは`false`値でデフォルトで無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、任意の値を渡さないでください。

## コマンドリスト

- [import](/tiup/tiup-component-cluster-import.md): Ansibleによってデプロイされたクラスタをインポートします
- [template](/tiup/tiup-component-cluster-template.md): トポロジーのテンプレートを出力します
- [check](/tiup/tiup-component-cluster-check.md): デプロイ前およびデプロイ後にクラスタをチェックします
- [deploy](/tiup/tiup-component-cluster-deploy.md): 指定されたトポロジに基づいてクラスタをデプロイします
- [list](/tiup/tiup-component-cluster-list.md): デプロイされたクラスタのリストをクエリします
- [display](/tiup/tiup-component-cluster-display.md): 指定されたクラスタのステータスを表示します
- [start](/tiup/tiup-component-cluster-start.md): 指定されたクラスタを開始します
- [stop](/tiup/tiup-component-cluster-stop.md): 指定されたクラスタを停止します
- [restart](/tiup/tiup-component-cluster-restart.md): 指定されたクラスタを再起動します
- [scale-in](/tiup/tiup-component-cluster-scale-in.md): 指定されたクラスタをスケールインします
- [scale-out](/tiup/tiup-component-cluster-scale-out.md): 指定されたクラスタをスケールアウトします
- [upgrade](/tiup/tiup-component-cluster-upgrade.md): 指定されたクラスタをアップグレードします
- [prune](/tiup/tiup-component-cluster-prune.md): Tombstoneステータスのインスタンスをクリーンアップします
- [edit-config](/tiup/tiup-component-cluster-edit-config.md): 指定されたクラスタの構成を変更します
- [reload](/tiup/tiup-component-cluster-reload.md): 指定されたクラスタの構成をリロードします
- [patch](/tiup/tiup-component-cluster-patch.md): デプロイされたクラスタのサービスを置換します
- [rename](/tiup/tiup-component-cluster-rename.md): クラスタの名前を変更します
- [clean](/tiup/tiup-component-cluster-clean.md): 指定されたクラスタからデータを削除します
- [destroy](/tiup/tiup-component-cluster-destroy.md): 指定されたクラスタを破棄します
- [audit](/tiup/tiup-component-cluster-audit.md): 操作の監査ログをクエリします
- [replay](/tiup/tiup-component-cluster-replay.md): 指定されたコマンドを再試行します
- [enable](/tiup/tiup-component-cluster-enable.md): マシンが再起動された後にクラスタサービスの自動有効化を有効にします
- [disable](/tiup/tiup-component-cluster-disable.md): マシンが再起動された後にクラスタサービスの自動有効化を無効にします
- [meta backup](/tiup/tiup-component-cluster-meta-backup.md): 操作およびメンテナンスに必要なTiUPメタファイルのバックアップを行います
- [meta restore](/tiup/tiup-component-cluster-meta-restore.md): 指定されたクラスタのTiUPメタファイルを復元します
- [help](/tiup/tiup-component-cluster-help.md): ヘルプ情報を出力します

[<< 前のページに戻る - TiUPリファレンスコンポーネントリスト](/tiup/tiup-reference.md#component-list)