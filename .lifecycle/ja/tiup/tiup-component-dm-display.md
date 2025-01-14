---
title: tiup dm display
---

# tiup dm display

DMクラスターの各コンポーネントの運用状況を確認する場合、一台ずつログインするのは非効率です。そのため、tiup-dmはこの作業を効率的に行うための `tiup dm display` コマンドを提供しています。

## 構文

```shell
tiup dm display <cluster-name> [flags]
```

`<cluster-name>` は操作対象のクラスター名です。クラスター名を忘れた場合は、[`tiup dm list`](/tiup/tiup-component-dm-list.md) コマンドを使用して確認できます。

## オプション

### -N, --node

- 複数のノードをカンマで区切って指定することができます。ノードのIDがわからない場合は、このオプションを省略してコマンドを実行すると、出力にすべてのノードのIDと状態が表示されます。
- データ型: `STRING`
- このオプションはデフォルトで `[]`（すべてのノード）が渡されて有効になっています。

> **注意:**
> 
> `-R, --role` も指定されている場合、`-N, --node` と `-R, --role` の両方の条件に一致するサービスノードのみがクエリされます。

### -R, --role

- 複数のロールをカンマで区切って指定することができます。ノードに展開されているロールがわからない場合は、このオプションを省略してコマンドを実行すると、出力にすべてのノードのロールと状態が表示されます。
- データ型: `STRING`
- このオプションはデフォルトで `[]`（すべてのロール）が渡されて有効になっています。

> **注意:**
> 
> `-N, --node` も指定されている場合、`-N, --node` と `-R, --role` の両方の条件に一致するサービスノードのみがクエリされます。

### -h, --help

- ヘルプ情報を表示します。
- データ型: `BOOLEAN`
- このオプションはデフォルトで `false` で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` の値を渡すか、値を渡さないようにします。

## 出力

- クラスター名
- クラスターバージョン
- SSHクライアントタイプ
- 以下のフィールドを含むテーブル:
    - `ID`: ノードID（IP:PORTで構成されています）。
    - `Role`: ノードに展開されたサービスのロール（例: TiDBまたはTiKV）。
    - `Host`: ノードに対応するマシンのIPアドレス。
    - `Ports`: サービスで使用されるポート番号。
    - `OS/Arch`: ノードのオペレーティングシステムとマシンアーキテクチャ。
    - `Status`: ノード上のサービスの現在の状態。
    - `Data Dir`: サービスのデータディレクトリ。 `-` はデータディレクトリがないことを示します。
    - `Deploy Dir`: サービスの展開ディレクトリ。

[<<前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)