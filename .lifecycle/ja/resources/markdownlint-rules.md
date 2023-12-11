# マークダウンのルール

TiDBのドキュメントはマークダウンで書かれています。ドキュメントを修正する際には、品質と一貫したフォーマットを確保するため、特定のマークダウンのルールに従う必要があります。そして、私たちはdocsリポジトリ内のマークダウンファイルに[markdownlintチェック](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md)を導入しました。

ルールに準拠しないPull Request（PR）があると、PRが**markdownlintチェックに合格できない**可能性があります。そのような場合、このPRをマージできなくなります。

関連するマークダウンのルールを把握していない場合や、PRがmarkdownlintチェックに合格しない場合は、心配しないでください。エラーメッセージには明確にどの行のどのファイルが特定のルールに合格しないかが記載されており、そのメッセージに従ってドキュメントを更新できます。

また、ローカルでmarkdownlintチェックを実行することもできます：

```bash
./scripts/markdownlint [FILE...]
```

👇以下の表は、TiDBドキュメント向けに事前設定された25の[markdownlintルール](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md)とその簡潔な説明を示しています。 🤓この表を一目見て、マークダウンの基本を把握できることでしょう。

| NO. | ルール | 説明 |
| :--- | :--- | :--- |
| 1 | [MD001 - 見出しのレベルは1つずつ増加すべき](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md001---heading-levels-should-only-increment-by-one-level-at-a-time) | 見出しは最初のレベルから始まるべきです。複数の見出しレベルを使用する場合、レベルを省略しないようにしてください。たとえば、3つ目の見出しレベルは直接1つ目の見出しレベルの下に配置できません。また、4つ目の見出しレベルは直接2つ目の見出しレベルの下に配置できません。 |
| 2 | [MD003 - 見出しのスタイル](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md003---heading-style) | 見出しはATXスタイルで書かれている必要があり、つまり`#`文字を使用して見出しのレベルを指定する必要があります。 |
| 3 | [MD018 - ATXスタイル見出しのハッシュの後にスペースを追加しない](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md018---no-space-after-hash-on-atx-style-heading) | **`#`の後にスペース** を1つ追加する必要があります。 |
| 4 | [MD019 - ATXスタイル見出しのハッシュの後に複数のスペースを追加しない](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md019---multiple-spaces-after-hash-on-atx-style-heading) | `#`の後に**複数のスペース**を追加してはいけません。1つのスペースのみを追加してください。 |
| 5 | [MD023 - 見出しは行の先頭から始まる必要があります](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md023---headings-must-start-at-the-beginning-of-the-line) | 見出しは行の先頭から始まる必要があります。見出しの`#`の前にスペースを追加してはいけません。 |
| 6 | [MD026 - 見出し内の末尾句読点](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md026---trailing-punctuation-in-heading) | 見出しの末尾には特定の句読点のみ使用できます。疑問符`?`、バッククォート`` ` ` ``、二重引用符`"`、シングルクォート`'`のみを末尾に使用することができます。それ以外の句読点（例：コロン`:`, コンマ`, ピリオド`.`, 感嘆符`!`）は使用できません。 |
| 7 | [MD022 - 見出しは空行で囲まれるべきです](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md022---headings-should-be-surrounded-by-blank-lines) | 見出しの前後には空行を入れる必要があります。 |
| 8 | [MD024 - 同じ内容の連続する見出しは許可されません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md024---multiple-headings-with-the-same-content) | 同じ内容の連続する見出しはドキュメント内で許可されていません。たとえば、1レベルの見出しが `# TiDB Architecture` である場合、それに続く2レベルの見出しを `## TiDB Architecture` にすることはできません。ただし、2つの見出しが連続でない場合は同じ内容にすることができます。 |
| 9 | [MD025 - 同じドキュメント内に複数のトップレベルの見出しを持つことはできません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md025---multiple-top-level-headings-in-the-same-document) | 同じドキュメント内には1つのトップレベル見出しがだけが許可されています。`title`と`category`を指定するメタデータがトップレベルの見出しの前にある場合は、このルールに違反しません。 |
| 10 | [MD041 - ファイルの最初の行はトップレベルの見出しでなければなりません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md041---first-line-in-file-should-be-a-top-level-heading) | ファイルの最初の行はトップレベルの見出しである必要があります。CIチェックは無視してファイルの最初の数行のメタデータを調べ、その後にトップレベルの見出しが続いているかどうかを確認します。 |
| 11 | [MD007 - 箇条書きのインデント](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md007---unordered-list-indentation) | 一般的には、全ての`.md`ファイルにおいてリスト項目は4つのスペースでインデントされます。例外は`TOC.md`ファイルで、ここではリスト項目は2つのスペースでインデントされます。 |
| 12 | [MD010 - タブの使用は禁止](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md010---hard-tabs) | **タブ文字**はコードブロックを含む全てのドキュメントで使用することはできません。インデントを行う場合は、**スペース** を使用してください。 |
| 13 | [MD012 - 複数の連続した空行は許可されません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md012---multiple-consecutive-blank-lines) | 複数の連続した空行は許可されません。 |
| 14 | [MD027 - ブロック引用記号の後の複数のスペースは許可されません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md027---multiple-spaces-after-blockquote-symbol) | ブロック引用記号`>`の後に複数のスペースを使用してはいけません。**1つだけ** のスペースを使用し、その後に引用内容が続きます。 |
| 15 | [MD029 - 順序付きリスト項目のプレフィックス](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md029---ordered-list-item-prefix) | 順序付きリストを使用する際、項目のプレフィックスは`1.`で始まり、数字の順に増加していく必要があります。 |
| 16 | [MD030 - リストマーカーの後のスペース](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md030---spaces-after-list-markers) | リストを使用する際、リストマーカー（`+`, `-`, `*`または数字）とリスト項目のテキストの間には**1つのスペース**のみが追加されます。 |
| 17 | [MD032 - リストは空行で囲まれるべきです](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md032---lists-should-be-surrounded-by-blank-lines) | リスト（どの種類でも）は前後に空行を持たなければなりません。 |
| 18 | [MD031 - コードブロックは空行で囲まれるべきです](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md031---fenced-code-blocks-should-be-surrounded-by-blank-lines) | コードブロックは前後に空行を持たなければなりません。|
| 19 | [MD034 - ベアURLの使用は禁止されています](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md034---bare-url-used) | ベアURLはドキュメントで使用することはできません。そのURLを直接クリックして開かせたい場合は、URLの周りに角括弧を追加してください（`<URL>`）。特別な状況でベアURLを使用する必要があり、ユーザーにそれをクリックして開くことを要求しない場合は、URLの周りにバッククォートを追加してください（`` `URL` ``）。 |
| 20 | [MD037 - 強調記号の内側のスペースは許可されません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md037---spaces-inside-emphasis-markers) | 強調記号（太字、斜体）を使用する場合、強調記号の内側に余分なスペースは許可されません。たとえば、`** 太字のテキスト **` は許可されません。 |
| 21 | [MD038 - コードスパン要素の内側のスペースは許可されません](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md038---spaces-inside-code-span-elements) | バッククォートを使用してコードスパン要素を囲む場合、コードスパン内に余分なスペースを入れてはいけません。たとえば、`` ` 例のテキスト ` `` は許可されません。 |
| 22 | [MD039 - リンクテキスト内のスペース](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md039---spaces-inside-link-text) | リンクテキストの両側に余分なスペースは許可されていません。例えば、`[ a link ](https://www.example.com/)`は許可されていません。 |
| 23 | [MD042 - 空のリンクは不可](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md042---no-empty-links) | リンクには宛先が必要です。例えば、`[empty link 1]()`や`[empty link 2](#)`は許可されていません。 |
| 24 | [MD045 - 画像には代替テキスト（altテキスト）が必要](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md045---images-should-have-alternate-text-alt-text) | 画像には代替テキスト（altテキスト）が必要です。代替テキストは`![]()`の`[]`内に記述されます。画像の読み込みに失敗した場合に、画像を説明します。 |
| 25 | [MD046 - コードブロックのスタイル](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md#md046---code-block-style) | ドキュメント内のコードブロックを囲むには、**3つのバッククォート** ` ``` `を使用してください。**4つのスペースのインデント**はコードブロックを示すために**使用しないで**ください。 |