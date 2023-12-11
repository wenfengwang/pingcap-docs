---
title: TiDBデータ移行のオンラインDDLツール対応
summary: DMにおける一般的なオンラインDDLツールのサポート、使用法、および注意点について学びます。
---

# TiDBデータ移行のオンラインDDLツール対応

MySQLエコシステムでは、gh-ostやpt-oscなどのツールが広く使用されています。TiDBデータ移行（DM）は、これらのツールに対応し、不要な中間データの移行を回避します。

この文書では、DMにおける一般的なオンラインDDLツールのサポート、使用法、および注意点について紹介します。

オンラインDDLツールに対するDMの動作原理および実装方法については、[online-ddl](/dm/feature-online-ddl.md)を参照してください。

## 制限事項

- DMはgh-ostおよびpt-oscのみをサポートしています。
- `online-ddl`が有効になっている場合、増分レプリケーションに対応するチェックポイントはオンラインDDLの実行中ではないことを確認する必要があります。たとえば、上流のオンラインDDL操作がバイナリログの`position-A`で開始し、`position-B`で終了する場合、増分レプリケーションの開始ポイントは`position-A`よりも前であるか、`position-B`よりも後である必要があります。そうでないと、エラーが発生します。詳細については、[FAQ](/dm/dm-faq.md#how-to-handle-the-error-returned-by-the-ddl-operation-related-to-the-gh-ost-table-after-online-ddl-true-is-set)を参照してください。

## 設定パラメータ

<SimpleTab>
<div label="v2.0.5 およびそれ以降">

v2.0.5およびそれ以降のバージョンでは、`task`構成ファイルで`online-ddl`設定項目を使用する必要があります。

- 上流のMySQL/MariaDB（同時に）がgh-ostまたはpt-oscツールを使用する場合、`task`構成ファイルで`online-ddl`を`true`に設定してください:

```yml
online-ddl: true
```

> **注意:**
>
> v2.0.5以降、`online-ddl-scheme`は非推奨となりました。そのため、`online-ddl-scheme`の代わりに`online-ddl`を使用する必要があります。つまり、`online-ddl: true`の設定は、`online-ddl-scheme`を上書きし、`online-ddl-scheme: "pt"`または`online-ddl-scheme: "gh-ost"`を`online-ddl: true`に変換します。

</div>

<div label="v2.0.5 より前">

v2.0.5より前のバージョン（v2.0.5を含まない）では、`task`構成ファイルで`online-ddl-scheme`設定項目を使用する必要があります。

- 上流のMySQL/MariaDBがgh-ostツールを使用する場合、`task`構成ファイルでそれを設定してください:

```yml
online-ddl-scheme: "gh-ost"
```

- 上流のMySQL/MariaDBがptツールを使用する場合、`task`構成ファイルでそれを設定してください:

```yml
online-ddl-scheme: "pt"
```

</div>
</SimpleTab>