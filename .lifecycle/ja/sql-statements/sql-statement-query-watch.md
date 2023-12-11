---
title: クエリウォッチ
summary: TiDBデータベースでのQUERY WATCHの使用概要。

# クエリウォッチ

`QUERY WATCH`ステートメントは、リソースグループ内の暴走クエリの監視リストを手動で管理するために使用されます。

> **警告:**
>
> この機能は実験的なものです。本番環境で使用しないでください。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)を報告できます。

## 概要

```ebnf+diagram
AddQueryWatchStmt ::=
    "QUERY" "WATCH" "ADD" QueryWatchOptionList
QueryWatchOptionList ::=
    QueryWatchOption
|   QueryWatchOptionList QueryWatchOption
|   QueryWatchOptionList ',' QueryWatchOption
QueryWatchOption ::=
    "RESOURCE" "GROUP" ResourceGroupName
|   "RESOURCE" "GROUP" UserVariable
|   "ACTION" EqOpt ResourceGroupRunawayActionOption
|   QueryWatchTextOption
ResourceGroupName ::=
    Identifier
|   "DEFAULT"
QueryWatchTextOption ::=
    "SQL" "DIGEST" SimpleExpr
|   "PLAN" "DIGEST" SimpleExpr
|   "SQL" "TEXT" ResourceGroupRunawayWatchOption "TO" SimpleExpr

ResourceGroupRunawayWatchOption ::=
    "EXACT"
|   "SIMILAR"
|   "PLAN"

DropQueryWatchStmt ::=
    "QUERY" "WATCH" "REMOVE" NUM
```

## パラメータ

[`QUERY WATCH`パラメータ](/tidb-resource-control.md#query-watch-parameters)を参照してください。

## MySQL互換性

このステートメントは、MySQLの構文にTiDB拡張機能です。

## 関連項目

* [暴走クエリ](/tidb-resource-control.md#manage-queries-that-consume-more-resources-than-expected-runaway-queries)