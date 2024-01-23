---
title: "WindowsでBIZ UDフォントのアンダーバーが薄くなる問題に対処する"
emoji: "🫥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css", "アクセシビリティ", "googlefonts", "tips"]
published: true
---

## BIZ UDフォントとは

モリサワが開発したユニバーサルデザインフォント「BIZ UDフォント」をご存知でしょうか？  
Google FontsからもWebフォントとして提供されているため、知っている方や使ったことがある方も多いと思います。

https://fonts.google.com/specimen/BIZ+UDGothic

> モリサワのBIZ UDゴシックは、教育やビジネス文書作成などに活用できるよう、より多くの方にとって読みやすく使いやすいように設計されたユニバーサルデザインフォントです。

多くの人がわかりやすく、読みやすいように工夫されたユニバーサルデザインフォントは、アクセシビリティの向上のためにも、積極的に使用していきたいものです。
政府広報オンラインのサイト（アクセシビリティにはかなり力を入れているはず）でもBIZ UDゴシックが使用されていました。[^1]

[^1]: 2024年1月23日現在

https://www.gov-online.go.jp/

また、現在Google Fontsで利用可能な日本語のユニバーサルデザインフォントはこのBIZ UDフォント（ゴシックと明朝）のみとなっています。
そのためユニバーサルフォントを採用するとなった場合、まずこちらが最有力候補となるでしょう。

## 問題点

しかしこのBIZ UDフォント、Windows環境（10のみ？）では特定のフォントサイズや画面の拡大・縮小時にアンダーバーが薄くなったり見えなくなるという問題が存在しています。

https://qiita.com/Qiibow/items/36035f4a3c07d086bfaa

Windowsユーザーの方は試しに[Google FontsのBIZ UDゴシックのページ](https://fonts.google.com/specimen/BIZ+UDGothic)で「_ ＿」と入力し、フォントサイズを色々と変えてみてください。

あるところで極端に細くなったり、薄く見えたり、場合によってはほぼ消えてしまうように見えると思います。私の環境では16~20px、27～29pxあたりで線が細くなってほぼ見えなくなってしまいました。

@[codepen](https://codepen.io/saka-na/pen/VwRzMJG)

この問題を解決するための対処法を紹介します。

## 対処法

対応としてはシンプルで、アンダーバーにだけ別のフォントを適用することで解決できます。
アンダーバー自体はただの棒なので、別のフォントになっても違和感が出ることは無いでしょう。

ここではOS標準フォントをアンダーバーに適用してみます。

### @font-face でアンダーバー専用のfont-familyを定義

以下のようにCSSの `@font-face` を使って、アンダーバー専用の `font-family` を定義します。

```css
@font-face {
  font-family: "lowline-only";
  src: local("Hiragino Sans"), local("Hiragino-Sans"), local("游ゴシック体"),
    local("YuGothic"), local("Meiryo"), local("Meiryo UI"), local("メイリオ");
  unicode-range: U+005F, U+FF3F;
}
```

- `font-family` には任意の名前を指定
- `src` でOS標準のフォントを指定
- `unicode-range` で適用させたい文字のUnicodeを指定

これによって、特定の文字（この場合はアンダーバー）にだけ適用されるフォントファミリーを定義できました。

:::message
`src` の指定はもっとシンプルにできる or より良い方法があるかもしれません。
時間ができたら調査してみたいですが、もし詳しい方がいれば教えていただけると幸いです。
:::

### font-familyの指定
あとは定義したフォントファミリーを"BIZ UDGothic"の前に指定するだけです。

```css
html {
  font-family: "lowline-only", "BIZ UDGothic", sans-serif;
}
```

これにより、アンダーバーには標準フォントが、その他の文字にはBIZ UDGothicが適用されます。

@[codepen](https://codepen.io/saka-na/pen/vYPJWNb)

無事アンダーバーがはっきりと見えるようになりました。

## まとめ

まずは「アンダーバー薄くなる問題」自体が根本的に解決されることを望みたいです。
ただ、とりあえずはこの方法で安心してBIZ UDフォントを使えるのではないでしょうか。

そして、これからより多くのサイトでUDフォントの採用が増えると良いなと思います。

## 参考リンク
https://developer.mozilla.org/ja/docs/Web/CSS/@font-face
https://qiita.com/unotovive/items/59f4f3d2379af5bcef5a
