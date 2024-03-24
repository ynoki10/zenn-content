---
title: "Next.js + Tailwind CSS でダークモード実装"
emoji: "🌃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "tailwindcss"]
published: true
---

## はじめに

先日Next.js + Tailwind CSSで構築している[個人ブログ](https://www.yoshinoki.dev/)でダークモードの実装を行いました。
ここではその手順を紹介しようと思います。

### 環境

| パッケージ名 | バージョン |
| ------------ | ---------- |
| next         | 14.0.4     |
| tailwindcss  | 3.4.1      |
| next-themes  | 0.3.0      |

※ Next.jsはApp Routerを使用（Pages Routerでも適用可能です）

## Tailwind CSS でのダークモード用のスタイル設定

まずはダークモード用のスタイルを用意します。

https://tailwindcss.com/docs/dark-mode

Tailwind CSSではダークモードに対応したVariantが用意されており、次のように`dark:`プレフィックスをつけるだけで簡単にスタイルを切り替えることができます。

```html
<div class="bg-white dark:bg-slate-800">
  <p class="text-slate-900 dark:text-white">テキスト</p>
</div>
```

ユーザーのOS設定に応じて切り替えるだけ（メディアクエリ `prefers-color-scheme` で判定）なら、これだけで対応が完了してしまいます。

ただしこの場合だと、サイトを訪れたユーザーはOS設定を変更しない限り見た目を切り替えることができません。

今回はユーザーに対してモード切替用のUIを提供したかったため、html要素のクラスに応じてスタイルが切り替わるように設定を変更します。

tailwind.config.jsに`darkMode`の設定を追加しましょう。

```diff js:tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
+  darkMode: ['selector', '.dark'],
  // ...
}
```

こうすることで`prefers-color-scheme`ではなく、「DOMツリーの祖先に`dark`クラスを持った要素があるか」を基準にスタイルを切り替えてくれるようになります。

:::details 任意のセレクタを基準にする

`.dark`ではなく`[data-mode="dark"]`など、任意のセレクタを基準にすることも可能です。

```js:tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ['selector', '[data-mode="dark"]'],
  // ...
}
```

:::

## `next-themes`の導入

[next-themes](https://github.com/pacocoursey/next-themes)を使うとNext.jsプロジェクトにおけるテーマの管理がとても簡単に行えるようになります。

詳しい説明はドキュメントを参照いただければと思いますが、

- 選択中のテーマをLocalStorageへ保存して再訪問時にも設定値を保持
- ページ読み込み時や更新時のチラつき防止
- カスタムフックuseThemeの提供（テーマの取得や更新が手軽にできる）
- CSSの`color-scheme`の切り替え

などをまとめて面倒見てくれるライブラリとなっています。

実際の使い勝手もよかったので、Next.jsでダークモードを実装する際にはとりあえずこちらを使っておけば間違いなさそうです。([shadcn/uiのドキュメント](https://ui.shadcn.com/docs/dark-mode/next)でも紹介されていました）

### 導入手順

:::message
ここではApp Routerを前提として進めていきます。  
Pages Routerでも導入可能ですので、詳しくは[ドキュメント](https://github.com/pacocoursey/next-themes?tab=readme-ov-file#with-pages)をご参照ください。
:::

まずはパッケージをインストールしましょう。

```bash
npm install next-themes
```

次に `app/layout.tsx` を更新していきます。

テーマを全体に適用するため、`next-themes`が提供する`ThemeProvider`でbody直下の要素全体をラップ。さらにhtml要素に `suppressHydrationWarning` を追加します。

```diff tsx:app/layout.tsx
+ import { ThemeProvider } from 'next-themes'

const RootLayout = ({ children }: { children: React.ReactNode }) => (
+  <html lang="ja" suppressHydrationWarning>
    <head />
    <body>
+      <ThemeProvider>{children}</ThemeProvider>
    </body>
  </html>
);

export default RootLayout;
```

:::message
以前はThemeProviderに`use client`を付与したラッパーを別途作成する必要がありました。
しかし`next-themes` 0.3.0 以降はライブラリが`use client`を付与してくれるため、ラッパーの作成は不要となりました。

https://github.com/pacocoursey/next-themes/releases/tag/v0.3.0
:::

#### suppressHydrationWarning について

`next-themes` はhtml要素を直接更新する（classやstyleを付与）ため、何もしないとReactのハイドレーション不一致の警告が出てしまいます。`suppressHydrationWarning` はこの警告を無視するための指定です。

なおこのプロパティは下層には影響しないため、プロジェクト内の他の場所で発生した警告を無視することはありません。

### next-themesの設定をカスタマイズ

ThemeProviderに対していくつか設定を追加します。

- テーマ切り替えにclassを利用する： `attribute="class"`
- システム設定に応じた切り替えを有効化： `enableSystem`
- デフォルトではシステム設定にしたがう： `defaultTheme="system"`

```diff tsx:app/layout.tsx
...
+ <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
    {children}
  </ThemeProvider>
...
```

## 切り替え用のUIを用意する

ここはプロジェクトによりさまざまかと思いますが、今回は[Tailwind CSSの公式サイト](https://tailwindcss.com/)と同じくドロップダウンから`Light` `Dark` `System`を選択できるようにしました。

![テーマ切り替え用のUIを示した画像：アイコン付きのボタンの下にドロップダウンメニューが表示されている。メニュー内にはLight,Dark,Systemの3つのテーマがアイコンと並んで表示されている](/images/color-theme-selector-ui.png)
_UIのイメージ_

[Radix UI の Dropdown Menu](https://www.radix-ui.com/primitives/docs/components/dropdown-menu)と[Hero Icons](https://heroicons.com/)を利用して作成したコンポーネントの完成形を以下に記載します。

```tsx:ColorThemeSelector.tsx
'use client';

import { ComputerDesktopIcon, MoonIcon, SunIcon } from '@heroicons/react/24/outline';
import * as DropdownMenu from '@radix-ui/react-dropdown-menu';
import { useTheme } from 'next-themes';
import { useEffect, useState } from 'react';

const ColorThemeSelector = () => {
  const [mounted, setMounted] = useState(false);
  const { theme, resolvedTheme, themes, setTheme } = useTheme();

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return (
      <div className="rounded border p-2 dark:border-gray-500">
        <div className="size-6"></div>
      </div>
    );
  }

  return (
    <DropdownMenu.Root>
      <DropdownMenu.Trigger asChild>
        <button
          aria-label="カラーテーマを選択する"
          className="rounded border p-2 text-gray-700 dark:border-gray-500 dark:text-slate-300"
          type="button"
        >
          {resolvedTheme === 'light' ? (
            <SunIcon className="size-6" />
          ) : (
            <MoonIcon className="size-6" />
          )}
        </button>
      </DropdownMenu.Trigger>

      <DropdownMenu.Portal>
        <DropdownMenu.Content
          align="end"
          className="overflow-hidden rounded border bg-white shadow-sm dark:border-gray-500 dark:bg-gray-950"
          sideOffset={8}
        >
          <DropdownMenu.Group className="flex flex-col">
            {themes.map((item) => (
              <DropdownMenu.Item
                className={`flex cursor-pointer items-center gap-2 px-3 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:text-slate-300 dark:hover:bg-gray-800 ${item === theme ? 'bg-gray-100 dark:bg-gray-800' : ''}`}
                key={item}
                onClick={() => setTheme(item)}
              >
                {item === 'light' ? (
                  <SunIcon className="size-5" />
                ) : item === 'system' ? (
                  <ComputerDesktopIcon className="size-5" />
                ) : (
                  <MoonIcon className="size-5" />
                )}
                <span className="capitalize">{item}</span>
                {item === theme && <span className="sr-only">（選択中）</span>}
              </DropdownMenu.Item>
            ))}
          </DropdownMenu.Group>
        </DropdownMenu.Content>
      </DropdownMenu.Portal>
    </DropdownMenu.Root>
  );
};

export default ColorThemeSelector;
```

### コンポーネントの解説

#### カスタムフック:useThemeについて

テーマの取得や更新用のAPIがカスタムフック[useTheme](https://github.com/pacocoursey/next-themes?tab=readme-ov-file#usetheme-1)として提供されています。

今回は`theme, resolvedTheme, themes, setTheme`を利用しました。

- `theme`：現在選択されているテーマ（今回の場合`light` `dark` `system`のいずれか）
- `resolvedTheme`：システム設定も踏まえた現在のテーマ（`light` `dark`のいずれか）
- `theme`：テーマのリスト
- `setTheme`：テーマを更新する関数

`resolvedTheme`を利用すると、システム設定が選択されていた場合でも現在表示中のテーマを取得することができます。
ここでは、メニューを開くボタンのアイコンを`resolvedTheme`の値に対応させています。

#### クライアントでマウントされるまではスケルトンを表示

サーバー上では`theme`を知ることができないため、`useTheme`から返される値の多くはクライアントにマウントされるまで`undefined`となります。そのためクライアントでのマウント前に`theme`などを使って UI をレンダリングしようとすると、hydration mismatch エラーが発生します。

それを回避するため、useEffectを利用して、クライアントにマウントされるまではスケルトンをレンダリングするようにしています。

```tsx
const [mounted, setMounted] = useState(false);

// useEffectはクライアントでしか実行されないため、これで安全にUIを表示できる。
useEffect(() => {
  setMounted(true);
}, []);

if (!mounted) {
  return (
    <div className="rounded border p-2 dark:border-gray-500">
      <div className="size-6"></div>
    </div>
  );
}
```

https://github.com/pacocoursey/next-themes?tab=readme-ov-file#avoid-hydration-mismatch

## まとめ

Tailwind CSSの`dark` Variantと`next-themes`を利用することで、かなり手軽にダークモードを実装することができました。

`next-themes`は`resolvedTheme`を提供してくれたり、CSSの`color-scheme`プロパティ[^color-scheme]も切り替えてくれるなど細かいところまで配慮が行き届いている点も嬉しいなと感じました。

[^color-scheme]: Webページ側から対応しているカラースキーマを提示できるプロパティです。`color-scheme: dark`の場合、多くのブラウザではスクロールバーの配色などをダークモードにあわせて変更します。[MDN](https://developer.mozilla.org/ja/docs/Web/CSS/color-scheme)

## 備考

- 今回はTailwind CSSの`dark`Variantを利用しましたが、[CSS Variablesで動的に定義するといった方法](https://zenn.dev/deer/articles/d3b104ac97711d)も考えられます。こちらについては書籍『[Tailwind CSS 実践入門](https://amzn.asia/d/c8C2d65)』で詳しく触れられていました。
- 実装時、UIについてあまり深く考えていませんでしたが、多くの場合トグルでLight/Darkを切り替える方がシンプルで良さそうだなと考え直しています。（理由↓）
  - System という選択肢が多くのユーザーには伝わらない気がする
  - 初期値を`prefers-color-scheme`から取得することでシステム設定には対応できる

## 参考URL

https://goodpatch-tech.hatenablog.com/entry/next-themes-tailwind
https://www.gaji.jp/blog/2023/11/14/17627/
https://azukiazusa.dev/blog/tailwindcss-dark-mode/
https://azukiazusa.dev/blog/tailwind-css-dark-mode-system-light-dark/
