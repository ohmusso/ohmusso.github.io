---
layout: post
title:  jekyll mermaid
date:   2023-03-05 22:30:00 +0900
categories: Jekyll
tags: jekyll mermaid javascript
mermaid: true
toppage: true
---

このサイトはurlから分かるように、githubページを使用しています。

[github-pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/creating-a-github-pages-site)

チュートリアルに従って、jekyllを使用してページを生成していますが、シーケンス図を書きたいので、mermaidを導入しました。

[mermaid](https://mermaid.js.org/)

以下のようなコードから図を生成します。

```code-mermaid
sequenceDiagram
    Alice->>+John: Hello John, how are you?
    Alice->>+John: John, can you hear me?
    John-->>-Alice: Hi Alice, I can hear you!
    John-->>-Alice: I feel great!
```

```mermaid
sequenceDiagram
    Alice->>+John: Hello John, how are you?
    Alice->>+John: John, can you hear me?
    John-->>-Alice: Hi Alice, I can hear you!
    John-->>-Alice: I feel great!
```

* 変更差分
  * <https://github.com/ohmusso/ohmusso.github.io/commit/a29399ff77e90f4066f3a3dbcaca3f56ccc95a87>
    * ローカルでは問題なかったのですが、デプロイすると図が表示されなかったです。。。
    htmlでclass=mermaidを直書きした場合は図が表示されたため、以下の補足に記載しているコードブロックの変換前にmermaidがレンダリングしているような動きでした。

  * <https://github.com/ohmusso/ohmusso.github.io/commit/45317daf539d1d3a870557a6051080d364e5893e>
    * mermaidのバージョンをv10にあげました。図をレンダリングするタイミングをコントロールできるようになり、解決できました。

* 補足

一番のポイントは、memaid.htmlに書かれたjavascriptです。
jekyllはmarkdownに書かれたコードブロック\(marmaid\)から以下のhtmlを生成します。

``` html
<pre>
    <code class=language-mermaid>XXX</code>
</pre>
```

しかし、mermaid.jsは上記のhtmlを図に変換しません。
以下のhtmlに変換する必要があります。

``` html
<pre class=mermaid>XXX</pre>
```

* 問題

図や文字が横に長くなった時に、折り返さなくなりました。。。
調査中です。

* 参考にしたサイト

参考になりました!ありがとうございます!

[先駆者様 1](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_includes/mermaid.html)

[先駆者様 2](https://qiita.com/fumitoh/items/ff28e0720ab0ebc84e96)
