---
title: TiUPミラーリファレンスガイド
summary: TiUPミラーの一般情報を学んでください。

# TiUPミラーリファレンスガイド

TiUPミラーはTiUPのコンポーネント倉庫であり、コンポーネントとそのメタデータを格納しています。TiUPミラーには以下の2つの形式があります:

+ ローカルディスク上のディレクトリ: ローカルTiUPクライアントで提供され、この文書ではローカルミラーと呼ばれます。
+ リモートディスクディレクトリを元に開始されたHTTPミラー: リモートTiUPクライアントで提供され、この文書ではリモートミラーと呼ばれます。

## ミラーの作成と更新

次の2つの方法でTiUPミラーを作成できます:

+ `tiup mirror init`を実行してゼロからミラーを作成します。
+ 既存のミラーからクローンするには`tiup mirror clone`を実行します。

ミラーが作成された後、`tiup mirror`コマンドを使用してミラーにコンポーネントを追加したり削除したりすることができます。TiUPはミラーを更新する際に、ファイルを追加して新しいバージョン番号を割り当てることで、ミラーからファイルを削除するのではありません。

## ミラーの構造

典型的なミラーの構造は以下のようになります:

```
+ <mirror-dir>                                  # ミラーのルートディレクトリ
|-- root.json                                   # ミラーのルート証明書
|-- {2..N}.root.json                            # ミラーのルート証明書
|-- {1..N}.index.json                           # コンポーネント/ユーザーインデックス
|-- {1..N}.{component}.json                     # コンポーネントメタデータ
|-- {component}-{version}-{os}-{arch}.tar.gz    # コンポーネントバイナリパッケージ
|-- snapshot.json                               # ミラーの最新スナップショット
|-- timestamp.json                              # ミラーの最新タイムスタンプ
|--+ commits                                    # ミラーの更新ログ (削除可能)
   |--+ commit-{ts1..tsN}
      |-- {N}.root.json
      |-- {N}.{component}.json
      |-- {N}.index.json
      |-- {component}-{version}-{os}-{arch}.tar.gz
      |-- snapshot.json
      |-- timestamp.json
|--+ keys                                       # ミラーのプライベートキー (他の場所に移動可能)
   |-- {hash1..hashN}-root.json                 # ルート証明書のプライベートキー
   |-- {hash}-index.json                        # インデックスのプライベートキー
   |-- {hash}-snapshot.json                     # スナップショットのプライベートキー
   |-- {hash}-timestamp.json                    # タイムスタンプのプライベートキー
```

> **注意:**
>
> + `commits`ディレクトリにはミラー更新の過程で生成されたログが格納され、ミラーをロールバックする際に使用されます。ディスク容量が不足する場合は、古いログディレクトリを定期的に削除できます。
> + `keys`ディレクトリに格納されているプライベートキーは機密情報です。別々に保管することを推奨します。

### ルートディレクトリ

TiUPミラーでは、ルート証明書は他のメタデータファイルの公開鍵を格納するために使用されます。TiUPクライアントが任意のメタデータファイル(`*.json`)を取得するたびに、TiUPクライアントはインストールされた`root.json`の中から、メタデータファイルのタイプ(root、index、snapshot、timestamp)に対応する公開鍵を見つける必要があります。それからTiUPクライアントは、公開鍵を使用して署名が有効かどうかを確認します。

ルート証明書の形式は以下の通りです:

```
{
    "signatures": [                                             # 各メタデータファイルにはいくつかのプライベートキーによって署名されたいくつかの署名があります。
        {
            "keyid": "{id-of-root-key-1}",                      # 署名に参加する最初のプライベートキーのIDです。このIDは、対応するプライベートキーの公開鍵の内容をハッシュ化したものです。
            "sig": "{signature-by-root-key-1}"                  # このプライベートキーによってこのファイルの署名部分です。
        },
        ...
        {
            "keyid": "{id-of-root-key-N}",                      # 署名に参加するN番目のプライベートキーのIDです。
            "sig": "{signature-by-root-key-N}"                  # このプライベートキーによってこのファイルの署名部分です。
        }
    ],
    "signed": {                                                 # 署名の部分です。
        "_type": "root",                                        # このファイルのタイプです。root.jsonのタイプはrootです。
        "expires": "{expiration-date-of-this-file}",            # このファイルの有効期限です。ファイルが期限切れの場合、クライアントはファイルを拒否します。
        "roles": {                                              # 各メタデータファイルに署名するキーを記録します。
            "{role:index,root,snapshot,timestamp}": {           # 各関係するメタデータファイルにはindex、root、snapshot、timestampが含まれます。
                "keys": {                                       # `keys`に記録された鍵の署名のみ有効です。
                    "{id-of-the-key-1}": {                      # {role}を署名する最初のキーのIDです。
                        "keytype": "rsa",                       # 鍵のタイプです。現在、鍵のタイプはrsaと固定されています。
                        "keyval": {                             # 鍵のペイロードです。
                            "public": "{public-key-content}"    # 公開鍵の内容です。
                        },
                        "scheme": "rsassa-pss-sha256"           # 現在、スキームはrsassa-pss-sha256と固定されています。
                    },
                    "{id-of-the-key-N}": {                      # {role}を署名するN番目のキーのIDです。
                        "keytype": "rsa",
                        "keyval": {
                            "public": "{public-key-content}"
                        },
                        "scheme": "rsassa-pss-sha256"
                    }
                },
                "threshold": {N},                               # メタデータファイルには最低N個の鍵の署名が必要です。
                "url": "/{role}.json"                           # ファイルが取得できるアドレスです。 インデックスファイルの場合、バージョン番号を付けてください(例:/{N}.index.json)。
            }
        },
        "spec_version": "0.1.0",                                # このファイルに続く指定されたバージョン番号です。将来ファイルの構造が変更される場合、バージョン番号をアップグレードする必要があります。 現在のバージョン番号は0.1.0です。
        "version": {N}                                          # このファイルのバージョン番号です。ファイルを更新するたびに、新しい{N+1}.root.jsonを作成し、そのバージョンをN + 1に設定する必要があります。
    }
}
```

### インデックス

インデックスファイルには、ミラー内のすべてのコンポーネントとコンポーネントの所有者情報が記録されています。

インデックスファイルの形式は以下の通りです:

```
{
    "signatures": [                                             # ファイルの署名。
        {
            "keyid": "{id-of-index-key-1}",                     # 署名に参加する最初のプライベートキーのIDです。
            "sig": "{signature-by-index-key-1}",                # このプライベートキーによってこのファイルの署名部分です。
        },
        ...
        {
            "keyid": "{id-of-root-key-N}",                      # 署名に参加するN番目のプライベートキーのIDです。
            "sig": "{signature-by-root-key-N}"                  # このプライベートキーによってこのファイルの署名部分です。
        }
    ],
    "signed": {
        "_type": "index",                                       # ファイルのタイプ。
        "components": {                                         # コンポーネントリスト。
            "{component1}": {                                   # 最初のコンポーネントの名前。
                "hidden": {bool},                               # 非表示コンポーネントかどうか。
                "owner": "{owner-id}",                          # コンポーネント所有者のID。
                "standalone": {bool},                           # 単独コンポーネントかどうか。
                "url": "/{component}.json",                     # コンポーネントを取得できるアドレス。バージョン番号を付ける必要があります(例:/{N}.{component}.json)。
                "yanked": {bool}                                # コンポーネントが削除されたかどうかを示します。
            },
            ...
            "{componentN}": {                                   # N番目のコンポーネントの名前。
                ...
            },
        },
        "default_components": ["{component1}".."{componentN}"], # ミラーが含まれている必須コンポーネント。現在、このフィールドはデフォルトで空です(無効)。
        "expires": "{expiration-date-of-this-file}",            # このファイルの有効期限です。ファイルが期限切れの場合、クライアントはファイルを拒否します。
        "owners": {
            "{owner1}": {                                       # 最初の所有者のID。
                "keys": {                                       # `keys`に記録された鍵の署名のみ有効です。
                    "{id-of-the-key-1}": {                      # 所有者の最初のキー。
                        "keytype": "rsa",                       # 鍵のタイプ。現在、鍵のタイプはrsaと固定されています。
                        "keyval": {                             # 鍵のペイロード。
                            "public": "{public-key-content}"    # 公開鍵の内容。
                        },
                        "scheme": "rsassa-pss-sha256"           # 現在、スキームはrsassa-pss-sha256と固定されています。
                    },
                    ...
                    "{id-of-the-key-N}": {                      # 所有者のN番目のキー。
                        ...
                    }
                },
                "name": "{owner-name}",                         # 所有者の名前。
                "threshod": {N}                                 # 所有者が所有するコンポーネントには少なくともN個の有効な署名が必要です。
            },
            ...
            "{ownerN}": {                                       # N番目の所有者のID。
                ...
            }
        },
        "spec_version": "0.1.0"                                # このファイルに続く指定されたバージョン番号です。将来ファイルの構造が変更される場合、バージョン番号をアップグレードする必要があります。 現在のバージョン番号は0.1.0です。
```
```json
{
    "version": {N}                                          # このファイルのバージョン番号です。ファイルを更新するたびに、新しい {N+1}.index.json を作成し、そのバージョンを N + 1 に設定する必要があります。
}
```

### コンポーネント

コンポーネントのメタデータファイルは、コンポーネント固有のプラットフォームとバージョンの情報を記録しています。

コンポーネントメタデータファイルの形式は次のとおりです:

```json
{
    "signatures": [                                             # ファイルの署名。
        {
            "keyid": "{id-of-index-key-1}",                     # 署名に参加する最初のプライベートキーのID。
            "sig": "{signature-by-index-key-1}",                # このプライベートキーによってこのファイルの署名された部分。
        },
        ...
        {
            "keyid": "{id-of-root-key-N}",                      # 署名に参加するN番目のプライベートキーのID。
            "sig": "{signature-by-root-key-N}"                  # このプライベートキーによってこのファイルの署名された部分。
        }
    ],
    "signed": {
        "_type": "component",                                   # ファイルの種類。
        "description": "{description-of-the-component}",        # コンポーネントの説明。
        "expires": "{expiration-date-of-this-file}",            # ファイルの有効期限。ファイルが有効期限切れの場合、クライアントはファイルを拒否します。
        "id": "{component-id}",                                 # コンポーネントのグローバルに一意なID。
        "nightly": "{nightly-cursor}",                          # ナイトリーカーソルであり、その値は最新のナイトリーバージョン番号です（たとえば、v5.0.0-nightly-20201209）。
        "platforms": {                                          # コンポーネントのサポートされているプラットフォーム（たとえば、darwin/amd64、linux/arm64）。
            "{platform-pair-1}": {
                "{version-1}": {                                # セマンティックバージョン番号（たとえば、v1.0.0）。
                    "dependencies": null,                       # コンポーネント間の依存関係を指定します。このフィールドは現在は使用されず、nullとして固定されています。
                    "entry": "{entry}",                         # tarパッケージ内のエントリバイナリファイルの相対パス。
                    "hashs": {                                  # tarパッケージのチェックサムです。sha256とsha512が使用されます。
                        "sha256": "{sum-of-sha256}",
                        "sha512": "{sum-of-sha512}",
                    },
                    "length": {length-of-tar},                  # tarパッケージの長さ。
                    "released": "{release-time}",               # バージョンのリリース日。
                    "url": "{url-of-tar}",                      # tarパッケージのダウンロードアドレス。
                    "yanked": {bool}                            # このバージョンが無効かどうかを示します。
                }
            },
            ...
            "{platform-pair-N}": {
                ...
            }
        },
        "spec_version": "0.1.0",                                # このファイルに従う指定されたバージョン。将来ファイル構造が変更された場合、バージョン番号をアップグレードする必要があります。現在のバージョン番号は0.1.0です。
        "version": {N}                                          # このファイルのバージョン番号です。ファイルを更新するたびに、新しい {N+1}.{component}.json を作成し、そのバージョンを N + 1 に設定する必要があります。
}
```

### スナップショット

スナップショットファイルは、各メタデータファイルのバージョン番号を記録します:

スナップショットファイルの構造は次のとおりです:

```json
{
    "signatures": [                                             # ファイルの署名。
        {
            "keyid": "{id-of-index-key-1}",                     # 署名に参加する最初のプライベートキーのID。
            "sig": "{signature-by-index-key-1}",                # このファイルの署名された部分。
        },
        ...
        {
            "keyid": "{id-of-root-key-N}",                      # 署名に参加するN番目のプライベートキーのID。
            "sig": "{signature-by-root-key-N}"                  # このファイルの署名された部分。
        }
    ],
    "signed": {
        "_type": "snapshot",                                    # ファイルの種類。
        "expires": "{expiration-date-of-this-file}",            # ファイルの有効期限。ファイルが有効期限切れの場合、クライアントはファイルを拒否します。
        "meta": {                                               # 他のメタデータファイルの情報。
            "/root.json": {
                "length": {length-of-json-file},                # root.jsonの長さ
                "version": {version-of-json-file}               # root.jsonのバージョン
            },
            "/index.json": {
                "length": {length-of-json-file},
                "version": {version-of-json-file}
            },
            "/{component-1}.json": {
                "length": {length-of-json-file},
                "version": {version-of-json-file}
            },
            ...
            "/{component-N}.json": {
                ...
            }
        },
        "spec_version": "0.1.0",                                # このファイルに従う指定されたバージョン。将来ファイル構造が変更された場合、バージョン番号をアップグレードする必要があります。現在のバージョン番号は0.1.0です。
        "version": 0                                            # このファイルのバージョン番号です。0に固定されています。
    }
```

### タイムスタンプ

タイムスタンプファイルは、現在のスナップショットのチェックサムを記録します。

タイムスタンプファイルの形式は次のとおりです:

```json
{
    "signatures": [                                             # ファイルの署名。
        {
            "keyid": "{id-of-index-key-1}",                     # 署名に参加する最初のプライベートキーのID。
            "sig": "{signature-by-index-key-1}",                # このファイルの署名された部分。
        },
        ...
        {
            "keyid": "{id-of-root-key-N}",                      # 署名に参加するN番目のプライベートキーのID。
            "sig": "{signature-by-root-key-N}"                  # このファイルの署名された部分。
        }
    ],
    "signed": {
        "_type": "timestamp",                                   # ファイルの種類。
        "expires": "{expiration-date-of-this-file}",            # ファイルの有効期限。ファイルが有効期限切れの場合、クライアントはファイルを拒否します。
        "meta": {                                               # snapshot.jsonの情報。
            "/snapshot.json": {
                "hashes": {
                    "sha256": "{sum-of-sha256}"                 # snapshot.jsonのsha256。
                },
                "length": {length-of-json-file}                 # snapshot.jsonの長さ。
            }
        },
        "spec_version": "0.1.0",                                # このファイルに従う指定されたバージョン。将来ファイル構造が変更された場合、バージョン番号をアップグレードする必要があります。現在のバージョン番号は0.1.0です。
        "version": {N}                                          # このファイルのバージョン番号です。ファイルを更新するたびに、新しい timestamp.json を上書きし、そのバージョンを N + 1 に設定する必要があります。
```