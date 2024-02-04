---
title: "Next.js + TypeScript + Tailwind CSS の開発環境をできるだけ丁寧に構築する【2024年】"
emoji: "🪴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "eslint", "prettier", "secretlint", "markuplint"]
published: true
---

## はじめに

最近、Next.js、TypeScript、Tailwind CSSを使って[技術ブログ](https://www.yoshinoki.dev/)を立ち上げました。（まだあまり更新は進んでいませんが…）

このプロジェクトを通じて構築した開発環境がわりと快適だったので、誰かの参考になるかもしれないと記事を書いてみることにしました。

できる限りわかりやすく詳細な説明を心がけましたが、その結果、記事のボリュームが大きくなってしまいました。長文ですが、興味のある方はぜひ読んでみてください🙏

https://github.com/ynoki10/next-ts-tailwind-starter
また、この記事内で紹介した内容をセットアップしたリポジトリを公開しています。
Next.jsのボイラープレートとして活用可能ですので、興味のある方はぜひ覗いてみてください。

:::message
内容に関してはできる限り調査して情報まとめたつもりですが、もし間違いがあった場合はコメントで教えていただけると幸いです。
アドバイスやご感想のコメントもお待ちしております！
:::

:::details 更新履歴
- JSXのPropsの順番をソート を追加しました（2024/02/04）
:::

## 概要

### 使用ツール・パッケージ

#### 前提とする環境

| ツール名           | バージョン |
| ------------------ | ---------- |
| Visual Studio Code | 1.85.2     |
| Node.js            | 20.10.0    |
| npm                | 10.2.3     |

※ npm は適宜 yarn や pnpm に読みかえていただいても問題ありません。

#### インストールしたパッケージ

| パッケージ名                     | バージョン |
| -------------------------------- | ---------- |
| next                             | 14.1.0     |
| typescript                       | 5.3.3      |
| tailwindcss                      | 3.4.1      |
| eslint                           | 8.56.0     |
| husky                            | 9.0.6      |
| lint-staged                      | 15.2.0     |
| markuplint                       | 3.15.0     |
| prettier                         | 3.2.4      |
| secretlint                       | 8.1.0      |
| @typescript-eslint/eslint-plugin | 6.19.1     |
| @typescript-eslint/parser        | 6.19.1     |
| eslint-config-prettier           | 9.1.0      |
| eslint-plugin-tailwindcss        | 3.14.1     |
| eslint-plugin-unused-imports     | 3.0.0      |

※ Next.jsはApp Routerで進めていますが、当記事の内容はPages Routerでもそのまま適用できるようになっていると思います。

### 設定したこと

- TypeScript と ESLint のルール設定
- import の整列
- JSX の Props の整列【2024/02 追記】
- Tailwind CSS の記法の統一
- マークアップ（HTML）の構文チェック（Markuplint）
- API トークンや秘密鍵などのコミットを防止（Secretlint）
- 各種 VSCode 拡張機能の導入と設定
- ファイル保存時に自動フォーマット&エラー修正
- コミット前に自動フォーマット
- コミット前にコンパイラチェックと各種Lintを実行（エラーがあればコミットを中止）

### 設定していないこと

- CIでのチェック
- テストツールの導入

## Next.jsのプロジェクトを作成する

まずは [create-next-app](https://nextjs.org/docs/app/api-reference/create-next-app) を利用して新規プロジェクトを作成します。
https://nextjs.org/docs/app/api-reference/create-next-app

```sh
npx create-next-app@latest
```

対話形式の質問に答えることで、回答に応じたプロジェクトを作成してくれます。

```sh
What is your project named?  [プロジェクト名]
Would you like to use TypeScript?  Yes
Would you like to use ESLint?  Yes
Would you like to use Tailwind CSS?  Yes
Would you like to use `src/` directory?  No
Would you like to use App Router? (recommended)  Yes
Would you like to customize the default import alias (@/*)?  No
```

ここでは全てデフォルトの設定を利用します。
すべての項目に答えると必要なパッケージが読み込まれ、プロジェクトが作成されます。

## TypeScript（tsconfig.json）の設定を追加する

続いてTypeScriptに関する設定を行います。
`create-next-app` で作成されたデフォルトの `tsconfig.json` でもすでに十分な印象はありますが、ここでは少しだけ設定を追加しておきます。

```diff json:tsconfig.json
 {
   "compilerOptions": {
     "lib": ["dom", "dom.iterable", "esnext"],
-    "allowJs": true,
+    "allowJs": false,
     "skipLibCheck": true,
     "strict": true,
     "noEmit": true,
     "esModuleInterop": true,
     "module": "esnext",
     "moduleResolution": "bundler",
     "resolveJsonModule": true,
     "isolatedModules": true,
     "jsx": "preserve",
     "incremental": true,
+    "forceConsistentCasingInFileNames": true,
+    "noImplicitReturns": true,
+    "noUncheckedIndexedAccess": true,
     "plugins": [
       {
         "name": "next"
       }
     ],
     "paths": {
       "@/*": ["./*"]
     }
   },
   "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
   "exclude": ["node_modules"]
 }
```

追加・変更した内容は次の通りです。

### "allowJs": false

型の恩恵に預かりたいのでJSは使用しない方針でいきます。
`false`にすることで`.js` `.jsx`をコンパイル対象外とすることができます。

### "forceConsistentCasingInFileNames": true

ファイル名の大文字小文字を区別します。
https://www.typescriptlang.org/tsconfig#forceConsistentCasingInFileNames

### "noImplicitReturns": true

戻り値が`void`以外の関数において、すべての条件分岐で戻り値を返すよう強制します。
https://typescriptbook.jp/reference/tsconfig/noimplicitreturns

### "noUncheckedIndexedAccess": true

インデックス型のプロパティや配列要素を参照したとき`undefined`のチェックを必須にします。
https://typescriptbook.jp/reference/tsconfig/nouncheckedindexedaccess
https://azukiazusa.dev/blog/typescript-no-unchecked-indexed-access/

:::message
ここで追加した設定以外にも「これ入れとくといいよ！」というものがあればコメントください！
:::

## Prettierでコードをフォーマットする

[Prettier](https://prettier.io/)を導入しコードの自動整形を行います。
コードをきれいに保っておくためにも必ず入れておきたいですね。
https://prettier.io/

### Prettierのインストール

まずは Prettier 本体をインストールします。

```sh
npm i -D prettier
```

### Prettierの設定（.prettierrc.json）

`.prettierrc.json`というファイル名で設定ファイルを作成します。
設定はそれぞれの好みですが、私はこんな感じにしています。

```json:.prettierrc.json
{
  "printWidth": 120,
  "tabWidth": 2,
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "arrowParens": "always"
}
```

https://zenn.dev/rescuenow/articles/c07dd571dfe10f

### .prettierignoreの設定

Prettierはプロジェクトの`.gitignore`で指定されたファイルを自動的にフォーマット対象外とします。
追加で除外したいファイルがある場合は`.prettierignore`というファイルで指定が可能です。
https://prettier.io/docs/en/ignore.html

ここでは`package-lock.json` を指定しておきます。（`yarn`や`pnpm`を利用している場合はそれぞれの lock ファイルを指定してください）

```ignore:.prettierignore
/package-lock.json
```

### フォーマット用のコマンドを追加

npm-scriptsにフォーマットのためのコマンドを追加しておきましょう。
https://ics.media/entry/12226/

```diff json:package.json
   "scripts": {
     ...
+    "prettier": "prettier --write ."
   },
```

この状態で

```sh
npm run prettier
```

を実行すると、プロジェクト内の全ての対象ファイルをフォーマットしてくれます。

### VSCodeのPrettier拡張機能と設定の追加

VSCodeにPrettierの拡張機能を入れることでファイル保存時に自動フォーマットが可能になります。
`esbenp.prettier-vscode`をインストールして有効化してください。
https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

さらに`.vscode/settings.json`を作成して、次のように記述します。

```json:.vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode", // Prettierをフォーマッターとして設定
  "editor.formatOnSave": true // ファイル保存時にフォーマットする
}
```

これにより、ファイル保存時に自動でPrettierによるフォーマットが実行されます。

:::details 自動フォーマットする拡張子を限定したい場合

自動フォーマットする拡張子を限定したい場合は次のように記述できます。

```json:.vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": false,
  "[javascript]": {
    "editor.formatOnSave": true
  },
  "[json]": {
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.formatOnSave": true
  },
  "[typescriptreact]": {
    "editor.formatOnSave": true
  }
}
```

こうすると`.js` `.json` `.ts` `.tsx` の保存時のみ自動フォーマットが実行されます。

または全ファイルに対して`"editor.formatOnSave": true`とした上で、無効化したい拡張子に`"editor.formatOnSave": false`を指定するというブラックリスト方式でもOKです。

:::

## ESLintの設定を追加してTypeScriptコードの品質を高める

TypeScript（JavaScript）の構文チェックのため[ESLint](https://eslint.org/)の設定を行います。
コードの品質を高く保つためにもここはできるだけ丁寧に設定しておきます。
https://eslint.org/

### デフォルトの設定を確認

まずは `create-next-app` で作成された `package.json` と `.eslintrc.json` を見てみましょう。

```json:package.json
{
  "scripts": {
    ...
    "lint": "next lint"
  },
  ...
  "devDependencies": {
    ...
    "eslint": "^8",
    "eslint-config-next": "14.1.0"
  }
}
```

```json:.eslintrc.json
{
  "extends": "next/core-web-vitals"
}
```

すでに

- npm-scripts（`lint`）の設定
- `eslint`と`eslint-config-next`のインストール
- 設定ファイル（`.eslintrc.json`）の`extends`に`next/core-web-vitals`を指定

までセットアップされていることが分かります。

ちなみに`next/core-web-vitals`は主にReactやNext.js独自のルールを適用するものとなっています。（詳細について知りたい方は[Next.jsの公式ドキュメント](https://nextjs.org/docs/pages/building-your-application/configuring/eslint#eslint-config)を参照してください。）

ただ、正直なところこれ**だけ**では全く安心して開発ができません。
そのため、これからプラグインや設定を追加していきます。

:::details 【補足】next/core-web-vitals について
`next/core-web-vitals` の内部では最終的に次の4つのプラグインをインポートしています。

- `eslint-plugin-react`
- `eslint-plugin-react-hooks` （extends経由で追加）
- `eslint-plugin-jsx-a11y`
- `eslint-plugin-import`

そして、次の3つのプラグインの推奨ルールセット(`recommended`)を適用しています。

- `eslint-plugin-react`
- `eslint-plugin-react-hooks`
- `eslint-plugin-next`

深掘りしていくと少しややこしいですが、ESLintの[pluginsとextendsの関係性を理解](https://blog.ojisan.io/eslint-plugin-and-extend/)した上で[eslint-config-nextのリポジトリ](https://github.com/vercel/next.js/blob/canary/packages/eslint-config-next/index.js)を眺めてみると仕組みがおおよそ把握できると思います。

https://blog.ojisan.io/eslint-plugin-and-extend/
:::

:::details 【補足】next lint の対象ディレクトリについて
`next lint`はデフォルトでは`pages/` `app/` `components/` `lib/` `src/`のみをLintの対象とします。

これ以外のディレクトリを使用したり、逆にLintの対象ディレクトリを制限したい場合は`next.config.js`にて設定が必要です。

例）ESLintの対象を `app/` `components/` `utils/` `hooks/` の4ディレクトリとする

```js:next.config.js
const nextConfig = {
  eslint: {
    dirs: ["app", "components", "utils", "hooks"],
  },
};

module.exports = nextConfig;
```

https://nextjs.org/docs/pages/building-your-application/configuring/eslint#linting-custom-directories-and-files
https://www.mizdra.net/entry/2023/01/13/013855
:::

### ESLintの推奨ルールを設定

`next/core-web-vitals` には **JavaScriptの基本的な構文に関するルールが含まれていません**。
そこで、まずはESLint本体の推奨ルールセット（`eslint:recommended`）を適用しましょう。

`.eslintrc.json`を次のように更新してください。

```diff json:.eslintrc.json
 {
-  "extends": "next/core-web-vitals"
+  "extends": ["eslint:recommended", "next/core-web-vitals"]
 }
```

これにより JavaScript の基本的な構文エラーが検出できるようになります。

https://eslint.org/docs/latest/rules/
https://www.tam-tam.co.jp/tipsnote/javascript/post11934.html

### TypeScriptに関するルールの追加

続いてTypeScript用のESLintルール [typescript-eslint](https://typescript-eslint.io/) を追加しましょう。
https://typescript-eslint.io/

まずは次の2パッケージをインストールしてください。

```sh
npm i -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

| パッケージ名                     | 概要                                                                       |
| -------------------------------- | -------------------------------------------------------------------------- |
| @typescript-eslint/parser        | TypeScriptの構文を解析するパーサー                                         |
| @typescript-eslint/eslint-plugin | Typescript用のESLintプラグイン<br>（推奨ルールセットもここに含まれている） |

そして`.eslintrc.json` を次のように更新します。

```diff json:.eslintrc.json
 {
   "extends": [
     "eslint:recommended",
+    "plugin:@typescript-eslint/recommended-type-checked",
     "next/core-web-vitals"
   ],
+  "parser": "@typescript-eslint/parser",
+  "parserOptions": {
+    "project": "./tsconfig.json"
   }
 }
```

追加した内容は以下の通りです。

#### plugin:@typescript-eslint/recommended-type-checked

`typescript-eslint` には次の[6つのルールセット](https://typescript-eslint.io/linting/configs/)があります。

|                      | 型情報を利用しない | 型情報を利用する         |
| -------------------- | ------------------ | ------------------------ |
| 厳格なルール         | strict             | strict-type-checked      |
| 推奨ルール           | recommended        | recommended-type-checked |
| 構文を統一するルール | stylistic          | stylistic-type-checked   |

これだけではどれを使えばよいか迷ってしまいますね。

完全に私感ですが、これらのルールについて調べた結果として、私は次のような印象を抱きました。

- strict は厳格すぎて採用の是非について意見が分かれる
- stylistic は癖が強く一部の挙動に影響がある
- type-checked のルールは入れておいて困ることは無さそう

そのため、ここでは`recommended-type-chekced`（推奨&型情報を利用する）のみを使用しています。

:::message
全てのルールを詳しく確認したわけではないので間違っている部分があるかもしれません。
誤りに気づいた方はコメントくださると嬉しいです。
:::

https://typescript-eslint.io/linting/configs/
https://zenn.dev/cybozu_frontend/articles/ts-eslint-v6-guide#v6-からのルールセット
https://zenn.dev/nekoya/articles/34e7002b8f164d

#### "parser": "@typescript-eslint/parser"

TypeScript の構文を解析するためのパーサーを指定しています。

#### parserOptions.project

extendsで`recommended-type-chekced`（型情報を利用するルール）を指定すると、この`parserOptions.project`の指定が必須となります。
ここでプロジェクトの`tsconfig.json`のパスを指定することで、ESLintにプロジェクトのTypeScript設定を伝えることができます。

ちなみにパスではなく`true`を渡すと自動的に一番近い`tsconfig.json`を探してくれるそうです。（ここではわかりやすさのため明示的にパスを指定しました）

https://typescript-eslint.io/packages/parser/#project

### 不要importの削除

続いて、コード内で使用していないimportの自動削除をしてくれるプラグイン [eslint-plugin-unused-imports](https://github.com/sweepline/eslint-plugin-unused-imports) を追加します。
未使用のimport文を手動で消す手間が省けるため、入れておくとかなり便利です。

https://github.com/sweepline/eslint-plugin-unused-imports

こちらはパッケージをインストールして`plugins`と`rules`に追加するだけで設定完了です。

```sh
npm i -D eslint-plugin-unused-imports
```

```diff json:.eslintrc.json
 {
   ...
+  "plugins": ["unused-imports"],
+  "rules": {
+    "unused-imports/no-unused-imports": "error"
+  }
 }
```

### importの順番を整理

続いてimportの順番を自動で整列してもらうように設定を追加していきます。
これは `eslint-plugin-import` の [import/order](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md) を使って設定ができます。
@[card](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md)

`eslint-plugin-import` は `eslint-config-next` にすでに含まれているため、ここではパッケージのインストールや `plugins` の指定は不要です（明示的にインストール&指定しても問題はありません）。
並べ方はそれぞれの好みやディレクトリ構成によりますが、私はとりあえず次のような設定としています。

```diff json:.eslintrc.json
   "rules": {
     ...
+    "import/order": [
+      "error",
+      {
+        "groups": ["builtin", "external", "internal", "parent", "sibling", "index", "object", "type"],
+        "newlines-between": "always",
+        "pathGroupsExcludedImportTypes": ["builtin"],
+        "pathGroups": [
+          {
+            "pattern": "@/components/**",
+            "group": "internal",
+            "position": "before"
+          }
+        ],
+        "alphabetize": {
+          "order": "asc"
+        }
+      }
+    ]
   }
```

こだわりたい方は[公式ドキュメント](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md)や以下の記事などを参考にしてみてください。
https://zenn.dev/knowledgework/articles/0994f518015c04#import%E3%81%AE%E8%87%AA%E5%8B%95%E6%95%B4%E5%88%97%EF%BC%88import%2Forder%EF%BC%89
https://zenn.dev/riemonyamada/articles/02e8c172e1eeb1

### JSXのPropsの順番をソート【2024/02 追記】

JSXコンポーネントのPropsの順番も自動でソートしてもらいましょう。
これは `eslint-plugin-react` の [jsx-sort-props](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/jsx-sort-props.md) を有効にするだけで可能です。

@[card](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/jsx-sort-props.md)

`eslint-plugin-react` は `eslint-config-next` に含まれているため、ここでもパッケージのインストールや `plugins` の指定は不要です。

```diff json:eslintrc.json
   "rules": {
     ...
+    "react/jsx-sort-props": "error",
   }
```

これにはビルドサイズを減らす効果もあるようです。コードもきれいになって一石二鳥ですね 🙌

https://zenn.dev/team_zenn/articles/sorting-props-reduces-gzip-size

### Tailwind CSSに関する設定を追加

Tailwind CSSを使っていて「クラス名の順番がバラバラでわかりにくい」「`pr-1 pl-1` と `px-1`のような書き方が混在していて気持ち悪い」などと思ったことはないでしょうか？
私はあります。

そんな悩みを解決するために、プラグイン [eslint-plugin-tailwindcss](https://github.com/francoismassart/eslint-plugin-tailwindcss) を追加しましょう。
https://github.com/francoismassart/eslint-plugin-tailwindcss

このプラグインによって次のようなことが可能になります。（詳しくは[ドキュメント](https://github.com/francoismassart/eslint-plugin-tailwindcss)を参照）

- クラス名の順序の整列
- 負の値の書き方の統一（`-top-[5px]` を `top-[-5px]`にする）
- ショートハンドの強制（`pr-1 pl-1` を `px-1`にする）
- 重複するプロパティ指定の警告（`p-2 p-3` のような重複に警告を出す）

まずはパッケージをインストールしてください。

```sh
npm i -D eslint-plugin-tailwindcss
```

そして `.eslintrc.json` に推奨ルールをセットすれば設定完了です。

```diff json:.eslintrc.json
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended-type-checked",
    "next/core-web-vitals",
+   "plugin:tailwindcss/recommended"
  ],
```

:::message
Tailwind CSS公式のPrettierプラグイン（[prettier-plugin-tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)）も存在するのですが、こちらはクラス順のソートにしか対応していません。そのためここでは機能が充実しているESLintのプラグインを採用しています。
:::

### Prettierと衝突するルールを無効化

ESLintでフォーマットに関するルールを設定していると、Prettierとバッティングしてしまう可能性があります。そんなバッティングを回避するためのルール [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier) を追加します。
https://github.com/prettier/eslint-config-prettier

まずはパッケージをインストールしてください。

```sh
npm i -D eslint-config-prettier
```

そして `.eslintrc.json` の `extends` にこの設定を追加すればOKです。

`eslint-config-prettier`は「Prettierと衝突する可能性のある全てのESLlintルールをオフにする」という設定なので、extendsの一番最後に指定するのがよいでしょう。（`extends`では後に指定したルールが前のルールを上書きします）

```diff json:.eslintrc.json
   "extends": [
     "eslint:recommended",
     "plugin:@typescript-eslint/recommended-type-checked",
     "next/core-web-vitals",
     "plugin:tailwindcss/recommended",
+    "prettier"
   ],
```

:::details .eslintrc.json に関する補足
ここまで `plugins` に `@typescript-eslint` や `tailwindcss`、`import` を指定していないことに疑問を持った方がいるかも知れません。
実を言うとこれらは `plugins` に書いても書かなくても同じように動作します。

- `@typescript-eslint` や `tailwindcss`は extends で適用しているため指定不要
- `import` は `next/core-web-vitals` 内で読み込まれているため指定不要

分かりやすさのためこれらを明示的に指定しても良いですが、今回は
**`extends`で使用しているプラグインは `plugins` に指定しない**
という方針で記述しました。

詳しく知りたい方はESLintの extends と plugins について調べてみてください。
https://blog.ojisan.io/eslint-plugin-and-extend/
https://zenn.dev/kimromi/articles/b7cf98005f3193
:::

### fix用のコマンドを追加

npm-scriptsにESLintのfix用コマンドを追加します。

```diff json:package.json
   "scripts": {
     ...
     "lint": "next lint",
+    "lint:fix": "next lint --fix",
     ...
   },
```

この状態で

```sh
npm run lint:fix
```

を実行すると、`next lint`の対象ファイルをチェックした上で、可能な箇所は自動修正してくれます。

### VSCodeのESLint拡張機能と設定の追加

VSCodeにESLintの拡張機能を追加しましょう。これによりエディタ上にESLintの警告を出したり、ファイル保存時の自動修正を行うことが可能になります。

`dbaeumer.vscode-eslint` をインストールして有効化してください。
https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint

これでエディタにESLintのエラーが表示されるようになります。
さらに `.vscode/settings.json` を更新します。

```diff json:.vscode/settings.json
 {
+  "editor.codeActionsOnSave": {
+    "source.fixAll.eslint": "explicit" // ファイル保存時にESLintの自動修正を実行する
+  },
   ...
 }
```

これにより、ファイル保存時にESLintによる自動修正が行われるようになります。

## MarkuplintでHTMLの構文をチェックする

続いてHTMLの構文チェックをするため [Markuplint](https://markuplint.dev/ja/) を導入していきましょう。
アクセシビリティ向上のためにもぜひ入れておくことをおすすめします。
https://markuplint.dev/ja/

### Markuplintの導入

導入用のコマンドがあるのでそちらを使用します。

```sh
npx markuplint --init
```

対話形式の質問に答えることで、設定ファイルの作成から必要なパッケージのインストールまでを自動実行してくれます。

```sh
✔ Which do you use template engines? · React (JSX)
✔ May I install them automatically? (y/N) · true
✔ Do you customize rules? (y/N) · false
✔ Does it import the recommended config? (y/N) · true
```

以上の内容で回答を進めると `.markuplintrc` という設定ファイルが作成され、さらに次の3パッケージがインストールされます。

| パッケージ名           | 概要                                                           |
| ---------------------- | -------------------------------------------------------------- |
| markuplint             | Markuplint本体                                                 |
| @markuplint/jsx-parser | JSX用のパーサプラグイン                                        |
| @markuplint/react-spec | React用のスペックプラグイン（React特有の属性を解釈してくれる） |

### 設定ファイルの更新

作成された`.markuplintrc`を確認してみましょう。

```json:.markuplintrc
{
  "specs": {
    "\\.[jt]sx?$": "@markuplint/react-spec"
  },
  "parser": {
    "\\.[jt]sx?$": "@markuplint/jsx-parser"
  },
  "extends": [
    "markuplint:recommended"
  ]
}
```

インストールしたパーサーとスペックプラグインが指定され、推奨プリセットルールも設定済みとなっていることが分かります。
このファイルに少しだけ修正を加えます。

```diff json:.markuplintrc
 {
   ...
   "extends": [
-    "markuplint:recommended"
+    "markuplint:recommended-react" // recommended ＋ React用の追加ルール
   ]
 }
```

`extends`の設定を`markuplint:recommended-react`（React用の推奨プリセット）に変更しました。これは「基本プリセットを全て読み込んだ上で React 用のルールを追加する」という設定[^1]です。

[^1]: 現状では`No hard coding ID`というルールのみが追加で有効になるようです。これはIDの重複を防ぐために、要素のid属性をハードコードしている場合に警告を表示するルールです。これに関して制作者の[ゆうてん](https://zenn.dev/yusukehirao)さんが[解説記事](https://zenn.dev/yusukehirao/articles/id-attribute-strategy)を書いてくれています。

:::message
Markuplintの[ドキュメント](https://markuplint.dev/ja/docs/rules)には一度目を通しておくのがおすすめです。それほどたくさんルールがあるわけではないので、わりとサクッと読めます。
:::

:::details 【補足】Markuplintのプリテンダー機能について
Markuplintの[プリテンダー](https://markuplint.dev/ja/docs/guides/besides-html#pretenders)という機能を利用すると、コンポーネントごとに対応するHTML要素を定義することができます。

できる限り多くのコンポーネントにプリテンダーを設定するのが望ましいですが、まずは頻出コンポーネントから優先的に設定しておくだけでも効果が期待できそうです。

私自身まだきちんと使えていませんが、Nextの`<Image>`をimg要素、`<Link>`をa要素に紐付ける設定をプロジェクトにデフォルトで入れておくのも良いかもしれません。
https://markuplint.dev/ja/docs/guides/besides-html#pretenders
:::

### Markuplint用のコマンドを追加

Markuplintについても npm-scripts を追加しておきます。

```diff json:package.json
   "scripts": {
     ...
+    "markuplint": "markuplint \"./**/*.tsx\"",
   },
```

この状態で

```sh
npm run markuplint
```

を実行すると、プロジェクト内の`.tsx`ファイルに対してマークアップの構文チェックを行ってくれます。[^2]

[^2]: markuplint には fix コマンドも存在しているのですが、私の環境ではうまく動作させられなかったため今回は使っていません。原因が調査できたら追記したいです。（[CLI のドキュメント](https://markuplint.dev/ja/docs/guides/cli)）

### VSCodeのMarkuplint拡張機能と設定の追加

MarkuplintにもVSCode拡張機能があるので入れておきましょう。
これを入れておくとエディタ上で Markuplint の警告が表示されるようになります。

`yusukehirao.vscode-markuplint`をインストールして有効化してください。
https://marketplace.visualstudio.com/items?itemName=yusukehirao.vscode-markuplint

さらに`.vscode/settings.json`を更新します。

```diff json:.vscode/settings.json
 {
   ...
+  "markuplint.enable": true // Markuplintを有効化する
 }
```

これによってエディタ上で Markuplint のエラーを確認できるようになります。

## Secretlintで秘匿情報の漏洩を防ぐ

続いてSecretlintの導入を進めていきましょう。

[Secretlint](https://github.com/secretlint/secretlint) はAPIトークンや秘密鍵など、リポジトリにコミットしてはいけないデータがファイルに含まれていないかをチェックするツールです。
https://github.com/secretlint/secretlint
https://efcl.info/2020/03/24/secretlint/

「APIへのアクセストークンを誤って公開してしまった」というような事故を防ぐためにもぜひ導入しておきましょう。（もちろんこれを入れておけば絶対安心、というものではないのでご注意ください。）

### Secretlintとルールセットのインストール

まずは本体と推奨ルールセットをインストールします。

```sh
npm i -D secretlint @secretlint/secretlint-rule-preset-recommend
```

### 設定ファイルの作成

設定ファイル作成のためのコマンドが用意されているので、そちらを使用します。

```sh
npx secretlint --init
```

実行すると`.secretlintrc.json`というファイルが作成されます。
ファイルの中身は次のようになっていて、推奨ルールがセットされていることが分かります。

```json:.secretlintrc.json
{
  "rules": [
    {
      "id": "@secretlint/secretlint-rule-preset-recommend"
    }
  ]
}
```

### Secretlint用のコマンドを追加

チェック実行用のnpm-scriptsを追加します。

```diff json:package.json
   "scripts": {
     ...
+    "secretlint": "secretlint --maskSecrets --secretlintignore .gitignore '**/*'",
   },
```

`--secretlintignore` は除外ファイルを指定するオプションで、ここでは `.gitignore`に書かれたファイルを対象外としています。

`--maskSecrets` はエラーメッセージに出力するシークレット情報を `***` でマスクしてくれるオプションです。（CIで実行した際のログなどに残らないよう、有効にしておくのがおすすめです）

この状態で

```sh
npm run secretlint
```

を実行すると、対象ファイル内にシークレットデータが無いかチェックしてくれます。
試しに次のようなコードを追加してみてください。

```ts:secret.ts
const AWS_KEY = 'AKIAXXXXXXEXAMPLEKEY';
```

この状態で`npm run secretlint` を実行すると次のようなエラーが表示されます。

```sh
1:24  error  [AWSAccessKeyID] found AWS Access Key ID: ********************  @secretlint/secretlint-rule-preset-recommend > @secretlint/secretlint-rule-aws
```

SecretlintがAWSのアクセスキーを検出し、エラーを出してくれていることが分かりますね。

## Husky + lint-staged でコミット前にコードをチェックする

ここまでコードチェックのためにさまざまな設定を追加してきました。
ここからは[Husky](https://typicode.github.io/husky/)と[lint-staged](https://github.com/lint-staged/lint-staged)を使ってGitのコミット前にコードチェックを実行し、問題のあるコードがコミットされないような環境を整えていきます。

| ツール名        | 概要                                                                                                                 |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Husky**       | Gitのコミットやプッシュ実行時に処理を呼び出すツール<br>（ここではpre-commitフックの際に`lint-staged`を呼び出します） |
| **lint-staged** | Gitでステージされたファイルに対して処理を呼び出すツール                                                              |

https://typicode.github.io/husky/
https://github.com/lint-staged/lint-staged

### Huskyとlint-stagedのインストール

まずはパッケージをインストールします。

```sh
npm i -D husky lint-staged
```

### Husky の初期化と設定ファイルを作成

続いて、Huskyの初期化と設定ファイルの作成を行います。
次のコマンドを実行してください。

```sh
npx husky init
```

`husky init` はHusky v9から追加されたコマンドで、以下の処理を実行してくれます。

- husky の初期化
- npm-scriptsの `prepare` コマンドに `husky` を追加（＝ `npm install` 時にHuskyの初期化を実行する設定）
- `pre-commit` 用の設定ファイル（`.husky/pre-commit`）を作成

作成された `.husky/pre-commit` を次のように更新します。

```diff :.husky/pre-commit
- npm run test
+ npx lint-staged
```

これでコミット前に `lint-staged` が呼び出されるようになります。

https://typicode.github.io/husky/get-started.html

### lint-staged の設定

続いて`lint-staged`の設定を進めていきます。

プロジェクトルートに `.lintstagedrc.js` を作成してください。
まずは[Next.jsのドキュメント](https://nextjs.org/docs/pages/building-your-application/configuring/eslint#lint-staged)にしたがって、以下の内容を記述します。

```js:.lintstagedrc.js
const path = require("path");

const buildEslintCommand = (filenames) =>
  `next lint --fix --file ${filenames
    .map((f) => path.relative(process.cwd(), f))
    .join(" --file ")}`;

module.exports = {
  "*.{js,jsx,ts,tsx}": [buildEslintCommand],
};
```

これでコミット時、ステージされたファイルに対して`next lint --fix`が実行されることになります。

ここからさらに手を加えて次のようにします。

```diff js:.lintstagedrc.js
 const path = require("path");

 const buildEslintCommand = (filenames) =>
   `next lint --fix --file ${filenames
     .map((f) => path.relative(process.cwd(), f))
     .join(" --file ")}`;

 module.exports = {
+  "*": ["secretlint --maskSecrets"],
+  "*.{js,cjs,mjs,json,ts,tsx,css}": ["prettier --write"],
+  "*.{ts,tsx}": ["bash -c tsc --noEmit", buildEslintCommand, "markuplint"],
 };
```

ここで設定した内容は次の通りです。

| 対象ファイル                                    | 実行する内容                                                                       |
| ----------------------------------------------- | ---------------------------------------------------------------------------------- |
| 全ファイル                                      | Secretlintの秘匿情報チェック                                                       |
| `.js` `.cjs` `.mjs` `.json` `.ts` `.tsx` `.css` | Prettierのフォーマット                                                             |
| `.ts` `.tsx`                                    | TypeScriptのコンパイルチェック<br>ESLintの構文チェック<br>Markuplintの構文チェック |

コミット前チェックの設定は以上です。

この状態でファイルをコミットすると、ステージされた各ファイルに対してチェックを実行し、もしエラーがあればコミットを中止してくれます。

チェックを無視してコミットしたいときは、コミット時に `-no-verify` か `-n` フラグをつけることでスキップができます。

:::details 【補足】コミット時 `command not found` のエラーが出る場合

Volta や nvm などの Node.js バージョン管理ツールを使用しているとGUI（VSCodeのGitやSourceTreeなど）からコミットしたとき `command not found` のエラーが出るかもしれません。

その場合、Huskyの実行前にPATHを通す設定が必要になります。

私はVoltaを使用しているため、次のような内容で `~/.config/husky/init.sh` を作成しました。

```sh:~/.config/husky/init.sh
export VOLTA_HOME="$HOME/.volta"
export PATH="$VOLTA_HOME/bin:$PATH"
```

詳しくは[Huskyのドキュメント](https://typicode.github.io/husky/how-to.html#node-version-managers-and-guis)や参考記事を参照ください。

https://typicode.github.io/husky/how-to.html#node-version-managers-and-guis
https://zenn.dev/cureapp/articles/f2722061739b51
:::

## VSCode の設定

ここまででかなり開発環境が整ってきましたね。
ここからVSCodeに関する設定をもう少しだけ追加していきます。

### Tailwind CSS IntelliSense（VSCode の拡張機能）を追加

Tailwind CSSのクラス名の自動補完をしてくれる、とても便利な拡張機能があるので導入しておきましょう。
`bradlc.vscode-tailwindcss` をインストールして有効化するだけでOKです。

https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss
https://tailwindcss.com/docs/editor-setup

### プロジェクトで使用している拡張機能を設定ファイルに記載

`.vscode/extensions.json` というファイルに、必要なVSCode拡張機能を指定しておきましょう。

```js:.vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint", // ESLint
    "esbenp.prettier-vscode", // Prettier
    "yusukehirao.vscode-markuplint", // Markuplint
    "bradlc.vscode-tailwindcss" // Tailwind CSS IntelliSense
  ]
}
```

こうすることでVSCodeの拡張機能検索欄に`@recommended`と入力するだけで、必要な拡張機能が一覧表示されるようになります。（メンバー間で環境を揃える際にとても便利）

### 未importモジュールの自動追加

ファイル保存時に未importのモジュールを自動で探してimportしてくれる便利な機能があるので有効化しておきましょう。
`.vscode/settings.json`を次のように更新します。

```diff json:.vscode/settings.json
 {
   "editor.codeActionsOnSave": {
     "source.fixAll.eslint": "explicit", // ファイル保存時にESLintの自動修正を実行する
+    "source.addMissingImports": "explicit" // 未importのモジュールを自動で探してimport文を追加する
   },
   ...
 }
```

これだけでファイル保存時、VSCodeが未importモジュールを探してimport文を追加してくれるようになります。

## プロジェクトに応じて追加する設定

最後に、プロジェクトに応じて設定しておくと便利な機能を紹介します。

### import の依存関係をルール化

[eslint-plugin-strict-dependencies](https://www.npmjs.com/package/eslint-plugin-strict-dependencies) というプラグインを利用すると、「このコンポーネントはこのファイルからしか読み込めない」というような依存関係のルールを決めることができます。

https://www.npmjs.com/package/eslint-plugin-strict-dependencies

まずはパッケージをインストールします。

```sh
npm i -D eslint-plugin-strict-dependencies
```

例えば「`next/link` を直接使わず、ラップしたコンポーネント `components/Link.tsx` を必ず使うようにしたい」といった場合、次のような設定となります。

```diff json:.eslintrc.json
+  "plugins": ["unused-imports", "strict-dependencies"],
   ...
   "rules": {
     ...
+    // import先を制限する
+    "strict-dependencies/strict-dependencies": [
+      "error",
+      [
+        {
+          "module": "next/link",
+          "allowReferenceFrom": ["components/Link.tsx"],
+          "allowSameModule": false
+        },
+      ],
+      {
+        "resolveRelativeImport": true
+      }
+    ]
   }
```

こうすることで「`components/Link.tsx` 以外からは`next/link` をimportできない」という制約が作れます。

設定はプロジェクトに応じてさまざまだと思うのでお好みで色々と設定してみてください。

https://zenn.dev/knowledgework/articles/0994f518015c04#%E4%BE%9D%E5%AD%98%E9%96%A2%E4%BF%82%E3%81%AE%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF%EF%BC%88strict-dependencies%EF%BC%89

## まとめ

開発環境を整える過程で新しい知識をたくさん得ることができました。
特にESLintに関してはこれまでかなり雰囲気で使っていたところがあったので、整理できてスッキリしています。

ただ、ESLintに関しては[v9以降 Flat Config という新たな設定方法へ移行することがアナウンス](https://eslint.org/blog/2023/10/flat-config-rollout-plans/)されていたり、ESLint + Prettier を代替する [Biome](https://biomejs.dev/ja/) といったツールも出てきているので、これからもまだまだキャッチアップが必要そうですね。。😱

ここからさらにMarkuplintのプリテンダーの設定をしたり、Secretlintのルールをカスタマイズしたりすることで、さらに便利な環境にしていくことができそうな気がしています。

また、今回はテストツールや CI の設定は行いませんでしたが、今後はこれらも整えてより堅牢な環境を構築できるようになりたいです。
コミット時はシンプルにPrettierやSecretlintのみを、CIでは各種Lintやコンパイラチェックを実行する、というようなフローにしても良いかもしれませんね。

長い記事でしたが最後までお読みいただき、本当にありがとうございました！

## 参考記事

本文中で紹介しきれなかった参考記事です。
https://qiita.com/tksst/items/bf62d50b25af69505e8e#9-typescriptの設定変更推奨度3
https://zenn.dev/chida/articles/bdbcd59c90e2e1
https://zenn.dev/t_keshi/scraps/9ddb388bc6975d
https://zenn.dev/rena_h/scraps/fd330154d02f76
https://zenn.dev/chot/articles/bcc178c481a9c2
https://zenn.dev/azukiazusa/articles/start-syntax-checking-with-markuplint
https://zenn.dev/risu729/articles/latest-husky-lint-staged
