---
title: tiup mirror の初期化
---

# tiup mirror init

コマンド `tiup mirror init` は空のミラーを初期化するために使用されます。初期化されたミラーにはコンポーネントまたはコンポーネント所有者が含まれていません。このコマンドは初期化されたミラーに対して以下のファイルのみを生成します:

```
+ <mirror-dir>                                  # ミラーのルートディレクトリ
|-- root.json                                   # ミラーのルート証明書
|-- 1.index.json                                # コンポーネント/ユーザーのインデックス
|-- snapshot.json                               # ミラーの最新スナップショット
|-- timestamp.json                              # ミラーの最新のタイムスタンプ
|--+ keys                                       # ミラーの秘密鍵（他の場所に移動できます）
   |-- {hash1..hashN}-root.json                 # ルート証明書の秘密鍵
   |-- {hash}-index.json                        # インデックスの秘密鍵
   |-- {hash}-snapshot.json                     # スナップショットの秘密鍵 
   |-- {hash}-timestamp.json                    # タイムスタンプの秘密鍵
```

上記ファイルの具体的な使用法や内容形式については、[TiUP Mirror 参照ガイド](/tiup/tiup-mirror-reference.md) を参照してください。

## 構文

```shell
tiup mirror init <path> [flags]
```

`<path>` は、TiUPがミラーファイルを生成して保存するローカルディレクトリを指定するために使用されます。ローカルディレクトリは相対パスで指定することができます。指定されたディレクトリが既に存在する場合、空でなければならず、存在しない場合、TiUPが自動的に作成します。

## オプション

### -k, --key-dir

- TiUPが秘密鍵ファイルを生成するディレクトリを指定します。指定されたディレクトリが存在しない場合、TiUPが自動的に作成します。
- データタイプ: `STRING`
- このオプションがコマンドで指定されていない場合、TiUPはデフォルトで `{path}/keys` に秘密鍵ファイルを生成します。

### 出力

- コマンドが正常に実行された場合、出力はありません。
- 指定された `<path>` が空でない場合、TiUPはエラー `Error: the target path '%s' is not an empty directory` を報告します。
- 指定された `<path>` がディレクトリでない場合、TiUPはエラー `Error: fdopendir: not a directory` を報告します。

[<< 前のページに戻る - TiUP Mirror コマンドリスト](/tiup/tiup-command-mirror.md#command-list)