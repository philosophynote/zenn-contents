# ZennのMarkdown記法一覧

このページでは Zenn のmarkdown記法を一覧で紹介します。

## 見出し

```markdown
# 見出し1
## 見出し2
### 見出し3
#### 見出し4
```

## リスト

```markdown
- Hello!
- Hola!
- Bonjour!
* Hi!
```

リストのアイテムには `*` もしくは `-` を使います。

### 番号付きリスト

```markdown
1. First
2. Second
```

## テキストリンク

```markdown
[アンカーテキスト](リンクのURL)
```

Markdownエディタでは、テキストを範囲選択した状態でURLをペーストすることで選択範囲がリンクになります。

## 画像

```markdown
![](https://画像のURL)
```

### 画像の横幅を指定する

画像の表示が大きすぎる場合は、URL の後に半角スペースを空けて `=○○x` と記述すると、画像の幅を px 単位で指定できます。

```markdown
![](https://画像のURL =250x)
```

### Altテキストを指定する

```markdown
![Altテキスト](https://画像のURL)
```

### キャプションをつける

画像のすぐ下の行に `*` で挟んだテキストを配置すると、キャプションのような見た目で表示されます。

```markdown
![](https://画像のURL)
*キャプション*
```

### 画像にリンクを貼る

以下のようにすることで画像に対してリンクを貼ることもできます。

```markdown
[![](画像のURL)](リンクのURL)
```

## テーブル

```markdown
| Head | Head | Head |
| ---- | ---- | ---- |
| Text | Text | Text |
| Text | Text | Text |
```

## コードブロック

コードは「```」で挟むことでブロックとして挿入できます。以下のように言語を指定するとコードへ装飾（シンタックスハイライト）が適用されます。

```js
const great = () => {
  console.log("Awesome");
};
```

シンタックスハイライトには Prism.js を使用しています。
[📄 対応言語の一覧 →](https://prismjs.com/#supported-languages)

### ファイル名を表示する

`言語:ファイル名` と `:` 区切りで記載することで、ファイル名がコードブロックの上部に表示されるようになります。

```js:ファイル名
const great = () => {
  console.log("Awesome")
}
```

### diff のシンタックスハイライト

`diff` と言語のハイライトを同時に適用するには、以下のように `diff` と `言語名` を半角スペース区切りで指定します。

```diff js
@@ -4,6 +4,5 @@
+ const foo = bar.baz([1, 2, 3]) + 1;
- let foo = bar.baz([1, 2, 3]);
```

なお、`diff` の使用時には、先頭に `+`、`-`、`>`、`<`、`半角スペース` のいずれが入っていない行はハイライトされません。

同時にファイル名を指定することも可能です。

```diff js:ファイル名
@@ -4,6 +4,5 @@
+ const foo = bar.baz([1, 2, 3]) + 1;
- let foo = bar.baz([1, 2, 3]);
```

## 数式

Zenn では KaTeX による数式表示に対応しています。
KaTeX のバージョンは常に最新バージョンを使用します。

### 数式のブロックを挿入する

`$$` で記述を挟むことで、数式のブロックが挿入されます。たとえば

```
$$
e^{i\theta} = \cos\theta + i\sin\theta
$$
```

### インラインで数式を挿入する

`$a\ne0$` というように `$` ひとつで挟むことで、インラインで数式を含めることができます。

## 引用

```markdown
> 引用文
> 引用文
```

## 脚注

脚注を指定するとページ下部にその内容が表示されます。

```markdown
脚注の例[^1]です。インライン^[脚注の内容その2]で書くこともできます。

[^1]: 脚注の内容その1
```

## 区切り線

```markdown
-----
```

## インラインスタイル

```markdown
*イタリック*
**太字**
~~打ち消し線~~
インラインで`code`を挿入する
```

## インラインのコメント

自分用のメモをしたいときは HTML のコメント記法を使用できます。

```html
<!-- TODO: ◯◯について追記する -->
```

この形式で書いたコメントは公開されたページ上では表示されません。ただし、複数行のコメントには対応していないのでご注意ください。

## Zenn 独自の記法

### メッセージ

```markdown
:::message
メッセージをここに
:::

:::message alert
警告メッセージをここに
:::
```

### アコーディオン（トグル）

```markdown
:::details タイトル
表示したい内容
:::
```

#### 要素をネストさせるには

外側の要素の開始/終了に `:` を追加します。

```markdown
::::details タイトル
:::message
ネストされた要素
:::
::::
```

## コンテンツの埋め込み

### リンクカード

```markdown
# URLだけの行
https://zenn.dev/zenn/articles/markdown-guide
```

URL だけが貼り付けられた行があると、その部分がカードとして表示されます。

また `@[card](URL)` という書き方でカード型のリンクを貼ることもできます。

#### アンダースコア _ を含むURLが正しく認識されない場合

アンダースコア（`_`）を含むURLで、正しくURLが認識されないことがあります。

```markdown
https://zenn.dev/__example__
```

対処法：
- カード型のリンクとして表示したい場合は `@[card](ここにURL)` という書き方をしてください
- 単純にリンク化された URL を貼り付けたい場合は `<https://zenn.dev/__example__>` のような形で `<` と `>` で URL を囲むようにしてください

### X（Twitter）のポスト（ツイート）

```markdown
# ポストのURLだけの行（前後に改行が必要です）
https://twitter.com/jack/status/20

# x.comドメインの場合
https://x.com/jack/status/20
```

以前は `@[tweet](ポストのURL)` の記法を採用していましたが、現在はポストのURLを貼り付けるだけで埋め込みが表示されます。

#### アンダースコア _ を含む URL が正しく認識されない場合

URL の `/` の区切りの中に 2 つ以上アンダースコア（`_`）を含むと、自動リンクが途中で途切れてしまいます。

対処法：このような URL では `@[tweet](ポストのURL)` という書き方をしていただくようお願いします。

#### リプライ元のポストを非表示にする

リプライを埋め込んだ場合、デフォルトでリプライ元のポスト含まれて表示されます。ポストのURL`?conversation=none` のようにクエリパラメータに `conversation=none` を指定すると、リプライ元のポストが含まれなくなります。

### YouTube

```markdown
# YouTubeのURLだけの行（前後に改行が必要です）
https://www.youtube.com/watch?v=WRVsOCh907o
```

以前は `@[youtube](YouTubeの動画ID)` という記法を採用していましたが、現在は動画URLを貼り付けるだけで動画を埋め込むことができます。

### GitHub

GitHub上のファイルへのURLまたはパーマリンクだけの行を作成すると、その部分にGitHubの埋め込みが表示されます。

```markdown
# GitHubのファイルURLまたはパーマリンクだけの行（前後に改行が必要です）
https://github.com/octocat/Hello-World/blob/master/README
```

#### 行の指定

GitHubと同じように、リンクの末尾に `#L00-L00` のような形で表示するファイルの開始行と終了行を指定することができます。

```markdown
# コードの開始行と終了行を指定
https://github.com/octocat/Spoon-Knife/blob/main/README.md#L1-L3

# コードの開始行のみ指定
https://github.com/octocat/Spoon-Knife/blob/main/README.md#L3
```

#### テキストファイル以外は埋め込めません

埋め込めるファイルは、ソースコードなどのテキストファイルのみとなっています。

### GitHub Gist

```markdown
@[gist](GistのページURL)
```

特定のファイルだけ埋め込みたい場合は `@[gist](https://gist.github.com/foo/bar?file=example.json)` のようにクエリ文字列で `?file=ファイル名` という形で指定します。

### CodePen

```markdown
@[codepen](ページのURL)
```

デフォルトの表示タブはページのURL`?default-tab=html,css` のようにクエリを指定することで変更できます。

### SlideShare

```markdown
@[slideshare](スライドのkey)
```

SlideShare の埋め込み iframe に含まれる `...embed_code/key/○○...` の ◯◯ の部分を入力します。

### SpeakerDeck

```markdown
@[speakerdeck](スライドのID)
```

SpeakerDeck で取得した埋め込みコードに含まれる `data-id` の値を入力します。

### Docswell

```markdown
@[docswell](スライドのURL)
# もしくは
@[docswell](埋め込み用のURL)
```

スライドのURL（`https://www.docswell.com/s/{UserId}/{SlideId}-xxx-xxx`）、もしくは埋め込み用のURL（`https://www.docswell.com/slide/{SlideId}/embed`）を入力します。

### JSFiddle

```markdown
@[jsfiddle](ページのURL)
```

埋め込みオプションを指定する場合、iframe用の埋め込みURL（ページのURL + `/embedded/{Tabs}/{Visual}/`）を入力します。

### CodeSandbox

```markdown
@[codesandbox](embed用のURL)
```

CodeSandbox では、各ページから埋め込み用の `<iframe>` を取得できます。この `<iframe>` に含まれる `src` の URL を括弧の中に入力します。

### StackBlitz

```markdown
@[stackblitz](embed用のURL)
```

StackBlitz では、各ページから「Embed URL」を取得できます。取得した URL をそのまま括弧の中に入力します。

### Figma

```markdown
@[figma](ファイルまたはプロトタイプのURL)
```

Figma では、ファイルまたはプロトタイプのページで共有リンクを取得できます。取得したURLをそのまま括弧の中に入力します。

### その他の埋め込み可能なコンテンツ

#### blueprintUE

```markdown
@[blueprintue](ページのURL)
```

例：
```markdown
@[blueprintue](https://blueprintue.com/render/0ovgynk-/)
```

blueprintUE を埋め込むには、公開されているページのURLをそのまま括弧の中に入力します。

## ダイアグラム

mermaid.js によるダイアグラム表示に対応しています。コードブロックの言語名を `mermaid` とすることで自動的にレンダリングされます。

```mermaid
graph TB
A[Hard edge] -->|Link text| B(Round edge)
B --> C{Decision}
C -->|One| D[Result one]
C -->|Two| E[Result two]
```

他にもシーケンス図やクラス図が表示できます。文法は mermaid.js に従っていますので、どのように書けばよいかは公式サイトの文法を参照してください。

### 制限事項

Zenn で mermaid.js 対応を行うにあたり、いくつか制限事項を設定させていただいています。

- **クリックイベントの無効化**: Interaction機能として図の要素にクリックイベントなどが設定できますが、セキュリティの観点でZennでは無効にさせていただきます。
- **ブロックあたりの文字数制限**: 2000文字以内
- **ブロックあたりのChain数制限**: 10以下

## 入力補完

### 絵文字（Emoji）

`:` に続いて任意の1文字を入力すると、絵文字の候補が表示されます。

## オンラインエディターではモーダルから挿入可能

オンラインのエディターでは「+」ボタンを押すことで、外部コンテンツ埋め込み用のモーダルを表示できます。