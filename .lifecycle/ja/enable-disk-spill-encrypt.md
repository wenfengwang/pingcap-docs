---
title: ディスクスピルの暗号化の有効化
summary: TiDBでディスクスピルの暗号化を有効にする方法について学びます。

# ディスクスピルの暗号化の有効化

システム変数 [`tidb_enable_tmp_storage_on_oom`](/system-variables.md#tidb_enable_tmp_storage_on_oom) が `ON`に設定されている場合、単一のSQLステートメントのメモリ使用量がシステム変数 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) の制限を超えると、一部のオペレータは実行中に中間結果を一時ファイルとしてディスクに保存し、クエリが完了した後にファイルを削除できます。

これらの一時ファイルの読み取りによるデータへのアクセスを防ぐために、ディスクスピルの暗号化を有効にできます。

## 構成

ディスクスピルファイルの暗号化を有効にするには、TiDBの構成ファイルの `[security]` セクションで項目 [`spilled-file-encryption-method`](/tidb-configuration-file.md#spilled-file-encryption-method) を構成できます。

```toml
[security]
spilled-file-encryption-method = "aes128-ctr"
```

`spilled-file-encryption-method` の値オプションには `aes128-ctr` と `plaintext` があります。デフォルト値は `plaintext` で、これは暗号化が無効であることを意味します。