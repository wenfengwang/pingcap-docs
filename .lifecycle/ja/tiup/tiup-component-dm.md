---
title: TiUP DM
---

# TiUP DM

[TiUP Cluster](/tiup/tiup-component-cluster.md)と同様にTiDBクラスタを管理する[TiUP DM](/tiup/tiup-component-cluster.md)はDMクラスタを管理するために使用されます。 TiUP DMコンポーネントを使用して、デプロイ、開始、停止、破棄、弾性スケーリング、DMクラスタのアップグレード、およびDMクラスタの構成パラメータを管理することができます。

## 構文

```shell
tiup dm [command] [flags]
```

`[command]`はコマンドの名前を渡すために使用されます。サポートされているコマンドについては[Command list](#command-list)を参照してください。

## Options

### --ssh

- コマンドの実行のためにリモートエンド（TiDBサービスがデプロイされているマシン）に接続するためのSSHクライアントを指定します。
- データ型： `STRING`
- サポートされている値：

    - `builtin`：tiup-clusterの組み込みeasysshクライアントをSSHクライアントとして使用します。
    - `system`：現在のオペレーティングシステムのデフォルトのSSHクライアントを使用します。
    - `none`：SSHクライアントは使用されません。デプロイメントは現在のマシンのみです。

- このオプションがコマンドで指定されていない場合、デフォルト値として`builtin`が使用されます。

### --ssh-timeout

- SSH接続のタイムアウトを秒単位で指定します。
- データ型： `UINT`
- このオプションがコマンドで指定されていない場合、デフォルトのタイムアウトは `5` 秒です。

### --wait-timeout

- 操作プロセスの各ステップに対する最大待ち時間（秒単位）を指定します。操作プロセスには、systemctlの指定、サービスの開始または停止の待ち時間など、多くのステップが含まれます。各ステップには数秒かかる場合があります。ステップの実行時間が指定されたタイムアウトを超えると、そのステップはエラーで終了します。
- データ型： `UINT`
- このオプションがコマンドで指定されていない場合、各ステップの最大待ち時間は `120` 秒です。

### -y、--yes

- すべてのリスクのある操作の二次確認をスキップします。TiUPを呼び出すスクリプトを使用しない限り、このオプションの使用は推奨されません。
- データ型： `BOOLEAN`
- このオプションはデフォルトで無効になっており、`false`値となっています。このオプションを有効にするには、このオプションをコマンドに追加し、`true`値を渡すか、任意の値を渡さないでください。

### -v、--version

- TiUP DMの現在のバージョンを出力します。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効になっており、`false`値となっています。このオプションを有効にするには、このオプションをコマンドに追加し、`true`値を渡すか、任意の値を渡さないでください。

### -h、--help

- 指定したコマンドに関するヘルプ情報を出力します。
- データ型： `BOOLEAN`
- このオプションはデフォルトで無効になっており、`false`値となっています。このオプションを有効にするには、このオプションをコマンドに追加し、`true`値を渡すか、任意の値を渡さないでください。

## Command list

- [import](/tiup/tiup-component-dm-import.md): DM-Ansibleによって展開されたDM v1.0クラスタをインポートします。
- [template](/tiup/tiup-component-dm-template.md): トポロジテンプレートを出力します。
- [deploy](/tiup/tiup-component-dm-deploy.md): 指定されたトポロジに基づいてクラスタをデプロイします。
- [list](/tiup/tiup-component-dm-list.md): デプロイされたクラスタのリストをクエリします。
- [display](/tiup/tiup-component-dm-display.md): 指定されたクラスタのステータスを表示します。
- [start](/tiup/tiup-component-dm-start.md): 指定されたクラスタを開始します。
- [stop](/tiup/tiup-component-dm-stop.md): 指定されたクラスタを停止します。
- [restart](/tiup/tiup-component-dm-restart.md): 指定されたクラスタを再起動します。
- [scale-in](/tiup/tiup-component-dm-scale-in.md): 指定されたクラスタをスケーリングインします。
- [scale-out](/tiup/tiup-component-dm-scale-out.md): 指定されたクラスタをスケーリングアウトします。
- [upgrade](/tiup/tiup-component-dm-upgrade.md): 指定されたクラスタをアップグレードします。
- [prune](/tiup/tiup-component-dm-prune.md): タンストーンステータスのインスタンスをクリーンアップします。
- [edit-config](/tiup/tiup-component-dm-edit-config.md): 指定されたクラスタの構成を変更します。
- [reload](/tiup/tiup-component-dm-reload.md): 指定されたクラスタの構成をリロードします。
- [patch](/tiup/tiup-component-dm-patch.md): 展開されたクラスタ内の指定されたサービスを置換します。
- [destroy](/tiup/tiup-component-dm-destroy.md): 指定されたクラスタを破棄します。
- [audit](/tiup/tiup-component-dm-audit.md): 指定されたクラスタの操作監査ログをクエリします。
- [replay](/tiup/tiup-component-dm-replay.md): 指定されたコマンドを再生します。
- [enable](/tiup/tiup-component-dm-enable.md): マシンが再起動された後にクラスタサービスの自動有効化を有効にします。
- [disable](/tiup/tiup-component-dm-disable.md): マシンが再起動された後にクラスタサービスの自動有効化を無効にします。
- [help](/tiup/tiup-component-dm-help.md): ヘルプ情報を出力します。

[<<前のページに戻る - TiUP リファレンスコンポーネントリスト](/tiup/tiup-reference.md#component-list)