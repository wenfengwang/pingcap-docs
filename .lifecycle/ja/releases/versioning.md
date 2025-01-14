---
title: TiDB バージョニング
summary: TiDB のバージョン番号システムについて学ぶ。

# TiDB バージョニング

<重要>

常にリリースシリーズの最新のパッチリリースにアップグレードすることをお勧めします。

</重要>

TiDB には、2 つのリリースシリーズがあります。

* 長期サポート版リリース
* 開発マイルストーン版リリース (TiDB v6.0.0 で導入)

TiDB のメジャーリリースのサポートポリシーについては、[TiDB リリースサポートポリシー](https://en.pingcap.com/tidb-release-support-policy/) をご覧ください。

## リリースバージョニング

TiDB のバージョニングは `X.Y.Z` の形式を取ります。`X.Y` はリリースシリーズを表します。

- TiDB 1.0 以降、`X` が毎年増加します。各 `X` リリースで新機能と改善が導入されます。
- `Y` は 0 から増加します。各 `Y` リリースで新機能と改善が導入されます。
- リリースシリーズの最初のリリースでは `Z` はデフォルトで 0 に設定されます。パッチリリースでは `Z` が 1 から増加します。

TiDB v5.0.0 およびそれ以前のバージョンのバージョニングシステムについては、[過去のバージョニング](#historical-versioning-deprecated) を参照してください。

## 長期サポート版リリース

長期サポート(LTS)バージョンは約 6 ヶ月ごとにリリースされ、新機能、改善、バグ修正、およびセキュリティ脆弱性修正が導入されます。

LTS リリースは `X.Y.Z` としてバージョニングされます。`Z` はデフォルトで 0 になります。

バージョン例:

- 6.1.0
- 5.4.0

LTS のライフサイクル中、必要に応じてパッチリリースが利用可能になります。パッチリリースにはバグ修正とセキュリティ脆弱性修正が含まれ、新機能は導入されません。

パッチリリースは `X.Y.Z` としてバージョニングされます。`X.Y` は対応する LTS バージョニングと一貫しています。パッチ番号 `Z` は 1 から増加します。

バージョン例:

- 6.1.1

<注意>

v5.1.0、v5.2.0、v5.3.0、v5.4.0 は、前のリリースからわずか 2 か月後にリリースされましたが、これら 4 つのリリースはすべて LTS であり、パッチリリースを提供しています。

</注意>

## 開発マイルストーン版リリース

開発マイルストーン版リリース (DMR) は、約 2 ヶ月ごとにリリースされ、LTS を含みません。DMR バージョンは新機能、改善、バグ修正が導入されます。TiDB は DMR を基にパッチリリースを提供せず、関連するバグは次のリリースシリーズで修正されます。

DMR はバージョニングされ、`X.Y.Z` はデフォルトで 0 になります。バージョン番号には `-DMR` 接尾辞が付きます。

バージョン例:

- 6.0.0-DMR

## TiDB エコシステムツールのバージョニング

TiDB サーバーと同じバージョン番号システムを使用して一緒にリリースされるいくつかの TiDB ツールがあります。例: TiDB Lightning。TiDB サーバーから別々にリリースされる他の TiDB ツールもあり、独自のバージョン番号システムを使用しています。例: TiUP および TiDB Operator。

## 過去のバージョニング (非推奨)

### 一般提供版リリース

一般提供版(GA)リリースは TiDB の現行リリースシリーズの安定したバージョンです。GA バージョンはリリース候補(RC)バージョンの後にリリースされます。GA は本番環境で使用可能です。

バージョン例:

- 1.0
- 2.1 GA
- 5.0 GA

### リリース候補版リリース

リリース候補版(RC)リリースは新機能と改善を導入します。RC バージョンはベータ版よりもはるかに安定しています。RC は初期テストに使用できますが、本番環境には適しません。

バージョン例:

- RC1
- 2.0-RC1
- 3.0.0-rc.1

### ベータ版リリース

ベータ版リリースは新機能と改善を導入します。ベータ版はアルファ版よりも大幅に改善され、重大なバグが解消されていますが、まだいくつかのバグが含まれています。ベータ版はユーザーが最新の機能をテストするために利用できます。

バージョン例:

- 1.1 Beta
- 2.1 Beta
- 4.0.0-beta.1

### アルファ版リリース

アルファ版リリースはテスト用の内部リリースであり、新機能と改善が導入されます。アルファ版リリースは現行リリースシリーズの最初のバージョンです。アルファ版リリースには一部のバグが存在する可能性があり、ユーザーは最新の機能をテストできます。

バージョン例:

- 1.1 Alpha