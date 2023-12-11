---
title: tiup ミラー
---

# tiup ミラー

TiUPでは、[ミラー](/tiup/tiup-mirror-reference.md)は重要な概念です。TiUPは現在、2つの形式のミラーリングをサポートしています。

- ローカルミラー：TiUPクライアントとミラーが同じマシン上にあり、クライアントはファイルシステムを介してミラーにアクセスします。
- リモートミラー：TiUPクライアントとミラーが同じマシン上になく、クライアントはネットワークを介してミラーにアクセスします。

`tiup mirror`コマンドは、ミラーを管理し、ミラーの作成、コンポーネントの配布、キーの管理方法を提供します。

## 構文

```shell
tiup mirror <command> [flags]
```

`<command>`はサブコマンドを表します。サポートされているサブコマンドの一覧については、以下の[コマンドリスト](#command-list)を参照してください。

## オプション

なし

## コマンドリスト

- [genkey](/tiup/tiup-command-mirror-genkey.md)：秘密鍵ファイルを生成します。
- [sign](/tiup/tiup-command-mirror-sign.md)：指定されたファイルに秘密鍵ファイルを使用して署名します。
- [init](/tiup/tiup-command-mirror-init.md)：空のミラーを初期化します。
- [set](/tiup/tiup-command-mirror-set.md)：現在のミラーを設定します。
- [grant](/tiup/tiup-command-mirror-grant.md)：現在のミラーの新しいコンポーネントの所有者を付与します。
- [publish](/tiup/tiup-command-mirror-publish.md)：新しいコンポーネントを現在のミラーに公開します。
- [modify](/tiup/tiup-command-mirror-modify.md)：現在のミラー内のコンポーネントの属性を変更します。
- [rotate](/tiup/tiup-command-mirror-rotate.md)：現在のミラー内のルート証明書を更新します。
- [clone](/tiup/tiup-command-mirror-clone.md)：既存のミラーから新しいミラーをクローンします。
- [merge](/tiup/tiup-command-mirror-merge.md)：ミラーをマージします。

[<< 前のページに戻る - TiUP 参照コマンドリスト](/tiup/tiup-reference.md#command-list)