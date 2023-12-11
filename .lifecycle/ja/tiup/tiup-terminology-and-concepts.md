---
title: TiUP用語およびコンセプト
summary: TiUPの用語およびコンセプトを説明します。
aliases: ['/docs/dev/tiup/tiup-terminology-and-concepts/']
---

# TiUP用語およびコンセプト

このドキュメントでは、TiUPの重要な用語とコンセプトについて説明します。

## TiUPコンポーネント

TiUPプログラムには、コンポーネントのダウンロード、更新、アンインストールに関するわずかなコマンドしか含まれていません。TiUPはさまざまなコンポーネントで機能を拡張します。**コンポーネント**とは、実行できるプログラムまたはスクリプトのことです。`tiup <component>`を介してコンポーネントを実行すると、TiUPは一連の環境変数を追加し、プログラムのためのデータディレクトリを作成した後、プログラムを実行します。

`tiup <component>`コマンドを実行することで、TiUPでサポートされているコンポーネントを実行できます。実行ロジックは次のとおりです：

+ `tiup <component>[:version]`を介してコンポーネントのバージョンを指定する場合：

    - ローカルに指定のバージョンのコンポーネントがインストールされていない場合、TiUPはミラーサーバーから最新の安定バージョンをダウンロードします。
    - ローカルに1つ以上のバージョンがインストールされており、指定されたバージョンがない場合、TiUPは指定されたバージョンをミラーサーバーからダウンロードします。
    - 指定されたバージョンのコンポーネントがローカルにインストールされている場合、TiUPは環境変数を設定してインストール済みのバージョンを実行します。

+ `tiup <component>`を介してバージョンを指定しないでコンポーネントを実行する場合：

    - ローカルに指定のバージョンのコンポーネントがインストールされていない場合、TiUPはミラーサーバーから最新の安定バージョンをダウンロードします。
    - 1つ以上のバージョンがインストールされている場合、TiUPは環境変数を設定して最新のインストール済みバージョンを実行します。

## TiUPミラー

TiUPのすべてのコンポーネントはTiUPミラーからダウンロードされます。TiUPミラーには各コンポーネントのTARパッケージと対応するメタ情報（バージョン、起動ファイル、チェックサム）が含まれています。TiUPはデフォルトでPingCAPの公式ミラーを使用します。`TIUP_MIRRORS`環境変数を通じてミラーソースをカスタマイズすることができます。

TiUPミラーはローカルファイルディレクトリまたはオンラインのHTTPサーバーになります：

+ `TIUP_MIRRORS=/path/to/local tiup list`
+ `TIUP_MIRRORS=https://private-mirrors.example.com tiup list`