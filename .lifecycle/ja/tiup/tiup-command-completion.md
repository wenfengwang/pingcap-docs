---
title: tiupの補完
---

# tiupの補完

ユーザーコストを削減するために、TiUPは`tiup completion`コマンドを提供して、自動的なコマンドライン補完のための設定ファイルを生成します。現在、TiUPは`bash`と`zsh`コマンドの補完をサポートしています。

`bash`コマンドを補完する場合は、まず`bash-completion`をインストールする必要があります。以下の手順を参照してください：

- macOS: もしbashのバージョンが4.1より古い場合は、`brew install bash-completion`を実行してください。それ以外の場合は、`brew install bash-completion@2`を実行してください。
- Linux: パッケージマネージャを使用して`bash-completion`をインストールしてください。例えば、`yum install bash-completion`や`apt install bash-completion`を実行してください。

## 構文

```shell
tiup completion <shell>
```

`<shell>`は使用しているシェルのタイプを設定するために使用されます。現在、`bash`と`zsh`がサポートされています。

## 使用方法

### bash

`tiup completion bash`コマンドをファイルに書き込んで、`.bash_profile`でファイルをソースにしてください。以下の例を参照してください：

```shell
tiup completion bash > ~/.tiup.completion.bash

printf "
# tiup shell completion
source '$HOME/.tiup.completion.bash'
" >> $HOME/.bash_profile

source $HOME/.bash_profile
```

### zsh

```shell
tiup completion zsh > "${fpath[1]}/_tiup"
```

[<< 前のページに戻る - TiUP リファレンスコマンドリスト](/tiup/tiup-reference.md#command-list)