---
title: 暗号化と圧縮関数
summary: 暗号化と圧縮関数について学びます。
aliases: ['/docs/dev/functions-and-operators/encryption-and-compression-functions/','/docs/dev/reference/sql/functions-and-operators/encryption-and-compression-functions/']
---

# 暗号化と圧縮関数

TiDBはMySQL 5.7で利用可能なほとんどの[暗号化と圧縮関数](https://dev.mysql.com/doc/refman/5.7/en/encryption-functions.html)をサポートしています。

## サポートされている関数

| 名前                                                                                                                                               | 説明                                       |
|:------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------|
| [`MD5()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_md5)                                                             | MD5チェックサムを計算する                            |
| [`PASSWORD()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_password)                                | パスワード文字列を計算し返す            |
| [`RANDOM_BYTES()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_random-bytes)                                           | ランダムなバイトベクトルを返す                       |
| [`SHA1(), SHA()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_sha1)                                                    | SHA-1 160ビットのチェックサムを計算する               |
| [`SHA2()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_sha2)                                                           | SHA-2のチェックサムを計算する                       |
| [`SM3()`](https://en.wikipedia.org/wiki/SM3_(hash_function))                                                    | SM3のチェックサムを計算する（現在MySQLはこの機能をサポートしていません）                      |
| [`AES_DECRYPT()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_aes-decrypt)                                             | AESを使用して複合化                                 |
| [`AES_ENCRYPT()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_aes-encrypt)                                             | AESを使用して暗号化                                 |
| [`COMPRESS()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_compress)                                                   | 結果をバイナリ文字列として返す                  |
| [`UNCOMPRESS()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_uncompress)                                               | 圧縮された文字列を解凍する                    |
| [`UNCOMPRESSED_LENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_uncompressed-length)                             | 圧縮前の文字列の長さを返す  |
| [`VALIDATE_PASSWORD_STRENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_validate-password-strength) | パスワードの強度を検証する |

## 関連システム変数

`block_encryption_mode`変数は`AES_ENCRYPT()`と`AES_DECRYPT()`に使用される暗号化モードを設定します。

## サポートされていない関数

* `DES_DECRYPT()`、`DES_ENCRYPT()`、`OLD_PASSWORD()`、`ENCRYPT()`: これらの関数はMySQL 5.7で非推奨となり、8.0で削除されました。
* MySQL Enterpriseでのみ利用可能な関数 [Issue #2632](https://github.com/pingcap/tidb/issues/2632)。