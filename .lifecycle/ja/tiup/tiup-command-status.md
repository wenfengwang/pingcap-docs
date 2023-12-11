---
title: tiup status
---

# tiup status（tiup ステータス）

`tiup status` コマンドは、`tiup [flags] <component> [args...]` コマンドを使用してコンポーネントを実行した後のコンポーネントの操作情報を表示するために使用されます。

> **注：**
>
> 次のコンポーネントの情報のみを確認できます：
>
> - まだ動作中のコンポーネント
> - `tiup -T/--tag` で指定されたタグを通って実行されたコンポーネント

## 構文

```shell
tiup status [flags]
```

## オプション

なし

## 出力

以下のフィールドから構成されるテーブル：

- `Name`: `-T/--tag` で指定されたタグ名。指定されていない場合はランダムな文字列です。
- `Component`: 動作中のコンポーネント。
- `PID`: 動作中のコンポーネントの対応するプロセスID。
- `Status`: 動作中のコンポーネントのステータス。
- `Created Time`: コンポーネントの起動時間。
- `Directory`: コンポーネントのデータディレクトリ。
- `Binary`: コンポーネントのバイナリファイルパス。
- `Args`: 動作中のコンポーネントの起動引数。

### コンポーネントのステータス

コンポーネントは次のステータスのいずれかで動作することができます：

- Up：コンポーネントは正常に動作しています。
- Down または Unreachable：コンポーネントは動作していないか、対応するホストでネットワークの問題が発生しています。
- Tombstone: コンポーネントのデータは完全に移行され、縮小が完了しています。このステータスはTiKVまたはTiFlashのみで存在します。
- Pending Offline: コンポーネントのデータは移行され、縮小が進行中です。このステータスはTiKVまたはTiFlashのみで存在します。
- Unknown: コンポーネントの実行ステータスが不明です。

> **注：**
>
> TiUPの `Pending Offline`、PD APIで返される `Offline`、TiDB Dashboardで表示される `Leaving` は、同じステータスを示しています。

コンポーネントのステータスは、PDのスケジューリング情報から派生します。詳細については、[情報収集](/tidb-scheduling.md#information-collection)を参照してください。

[<< 前のページに戻る - TiUPリファレンスコマンド一覧](/tiup/tiup-reference.md#command-list)