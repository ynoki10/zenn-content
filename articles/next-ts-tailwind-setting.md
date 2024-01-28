---
title: "Next.js + TypeScript + Tailwind CSS の開発環境をできるだけ丁寧に構築する【2024年】"
emoji: "🪴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "eslint", "prettier", "secretlint", "markuplint"]
published: false
---

## はじめに

先日 Next.js + TypeScript + Tailwind CSS を利用して[技術ブログ](https://www.yoshinoki.dev/)を作りました。（まだ全然更新できていません。。）

開発環境を整えようと頑張った結果、それなりに満足のいく環境ができたので記事にしてみました。できるだけ丁寧な解説を心がけたためボリュームが多くなっていますが、よろしければお付き合いください。

https://github.com/ynoki10/next-ts-tailwind-starter
また、記事内で解説している内容をセットアップしたリポジトリを公開しています。
Next.jsのボイラープレートとしても活用可能ですので、興味のある方はぜひ覗いてみてください。

:::message
内容に関してはできる限り調査して情報をまとめたつもりですが、もし間違いがあった場合はコメントで教えていただけると幸いです。
アドバイスやご感想のコメントもお待ちしております！
:::

## 概要

### 使用ツール・パッケージ

:::details 使用ツール・パッケージ一覧

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

※ Next.jsはApp routerで進めていますが、Pages routerでも特に設定に影響はありません。
:::

### 設定したこと

- TypeScript と ESLint のルール設定
- import 順の整列
- Tailwind CSS の記法の統一
- マークアップ（HTML）の構文チェック（Markuplint）
- API トークンや秘密鍵などのコミットを防止（Secretlint）
- ファイル保存時に自動フォーマット&エラー修正
- コミット前に自動フォーマット
- コミット前にコンパイラチェックと各種Lintを実行（エラーがあればコミットを中止）
- 各種 VSCode 拡張機能の導入

### 設定していないこと

- CIでのチェック
- テストツールの導入

## Next.jsのプロジェクトを作成する

まずは [create-next-app](https://nextjs.org/docs/app/api-reference/create-next-app) を利用して新規プロジェクトを作成します。

```sh
npx create-next-app@latest
```

対話形式の質問に答えると、回答に応じたプロジェクトが作成されます。

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

TypeScriptに関する設定を行います。
`create-next-app`で作成されたデフォルトの`tsconfig.json`でも十分な印象はありますが、少しだけ設定を追加しておきます。

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

追加・変更した内容を確認していきましょう。

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

Prettier はプロジェクトの`.gitignore`で指定されたファイルを自動的にフォーマット対象外としますが、追加で除外したいファイルがある場合`.prettierignore`というファイルで指定が可能です。
https://prettier.io/docs/en/ignore.html

ここでは`package-lock.json` を指定しておきます。（`yarn`や`pnpm`を利用している場合はそれぞれの lock ファイルを指定してください）

```ignore:.prettierignore
/package-lock.json
```

### フォーマット用のコマンドを追加

npm-scriptsにコマンドを追加しましょう。

```diff json:package.json
"scripts": {
   ...
+  "prettier": "prettier --write ."
},
```

この状態で

```sh
npm run prettier
```

を実行すると、プロジェクト内の全て対象ファイルをフォーマットしてくれます。

### VSCode拡張機能と設定の追加

VSCodeに拡張機能を入れることでファイル保存時の自動フォーマットが可能になります。

まずは`esbenp.prettier-vscode`をVSCodeにインストールして有効化してください。
https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

さらに`.vscode/settings.json`を作成して、次のように記述します。

```json:.vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode", // Prettierをフォーマッターとして設定
  "editor.formatOnSave": true // ファイル保存時にフォーマットする
}
```

これにより、ファイル保存時に自動でPrettierによるフォーマットが実行されます。

:::details 自動フォーマットの対象を限定したい場合

対象ファイルを限定したい場合は次のように記述できます。

```json:.vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
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

:::

## ESLintの設定を追加する

TypeScript（JavaScript）の構文チェックのため[ESLint](https://eslint.org/)の設定を行います。
https://eslint.org/

### デフォルトの設定を確認

`create-next-app` で作成された `package.json` と `.eslintrc.json` を見てみましょう。

```json:package.json
{
  "scripts": {
    "lint": "next lint"
  },
  "devDependencies": {
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

- `eslint`と`eslint-config-next`のインストール
- npm-scripts（`next lint`）の設定
- `.eslintrc.json`の`extends`に`next/core-web-vitals`を設定

までセットアップ済みであることが分かります。

このうち`next/core-web-vitals`には主にReactやNext.js独自のルールが含まれています。
（詳細について知りたい方は[Next.jsの公式ドキュメント](https://nextjs.org/docs/pages/building-your-application/configuring/eslint#eslint-config)を参照してください。）

:::details 【補足】next/core-web-vitals について
`next/core-web-vitals` の内部では最終的に次の4つのプラグインをインポートしています。

- `eslint-plugin-react`
- `eslint-plugin-react-hooks` （extends経由で追加）
- `eslint-plugin-jsx-a11y`
- `eslint-plugin-import`

さらに次の3つのプラグインの推奨ルールセット(`recommended`)を適用しています。

- `eslint-plugin-react`
- `eslint-plugin-react-hooks`
- `eslint-plugin-next`

深掘りしていくと少しややこしいですが、ESLintの[pluginsとextendsの関係性を理解](https://blog.ojisan.io/eslint-plugin-and-extend/)した上で[eslint-config-nextのリポジトリ](https://github.com/vercel/next.js/blob/canary/packages/eslint-config-next/index.js)を眺めてみると仕組みはおおよそ把握できると思います。

https://blog.ojisan.io/eslint-plugin-and-extend/
:::

:::details 【補足】next lint の対象ディレクトリについて
`next lint`はデフォルトでは`pages/` `app/` `components/` `lib/` `src/`のみをLintの対象とします。

これ以外のディレクトリを使用したり、逆Lintの対象ディレクトリを制限したい場合は`next.config.js`にて設定が必要です。

例）

```js:next.config.js
const nextConfig = {
  eslint: {
    dirs: ["app", "components", "utils", "hooks"], // 'app', 'components', 'utils', 'hooks' ディレクトリに対して ESLint を実行する
  },
};

module.exports = nextConfig;
```

https://nextjs.org/docs/pages/building-your-application/configuring/eslint#linting-custom-directories-and-files
https://www.mizdra.net/entry/2023/01/13/013855
:::

### ESLintの推奨ルールを設定

`next/core-web-vitals`には**JavaScriptの基本的な構文に関するルールが含まれていません**。
そこで、まずはESLint本体の推奨ルールセット（`eslint:recommended`）を設定します。

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

[typescript-eslint](https://typescript-eslint.io/)を導入してTypeScript用のESLintルールを追加します。
https://typescript-eslint.io/

まずは次の 2 パッケージをインストールしてください。

```sh
npm i -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

| パッケージ名                     | 概要                                |
| -------------------------------- | ----------------------------------- |
| @typescript-eslint/parser        | TypeScript の構文を解析するパーサー |
| @typescript-eslint/eslint-plugin | Typescript 用の拡張ルールと推奨設定 |

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

追加した内容について確認していきましょう。

#### plugin:@typescript-eslint/recommended-type-checked

typescript-eslint には次のように[6 つのルールセット](https://typescript-eslint.io/linting/configs/)があります。

|                      | 型情報を利用しない | 型情報を利用する         |
| -------------------- | ------------------ | ------------------------ |
| 厳格なルール         | strict             | strict-type-checked      |
| 推奨ルール           | recommended        | recommended-type-checked |
| 構文を統一するルール | stylistic          | stylistic-type-checked   |

どれを使えばよいのか迷いますが、調べる中で

- strict は厳格すぎて採用の是非について意見が分かれる
- stylistic は癖が強く一部の挙動に影響がある
- type-checked のルールは入れておいて困ることは無さそう

ということが分かってきました。
そのため、ここでは`recommended-type-chekced`（推奨&型情報を利用する）のみを使用しています。

:::message
全てのルールを詳しく確認したわけではないので間違っている部分があるかもしれません。
誤りに気づいた方はコメントくださると嬉しいです。 🙇
:::

https://typescript-eslint.io/linting/configs/
https://zenn.dev/cybozu_frontend/articles/ts-eslint-v6-guide#v6-からのルールセット
https://zenn.dev/nekoya/articles/34e7002b8f164d

#### "parser": "@typescript-eslint/parser"

TypeScript の構文を解析するためのパーサーを指定しています。

#### parserOptions.project

extendsで`recommended-type-chekced`（型情報を利用するルール）を指定したため、`parserOptions.project`の指定が必須となります。
プロジェクトの`tsconfig.json`の場所を指定することで、ESLintがTypeScriptに関する設定を解釈してくれます。

ちなみにパスではなく true を渡すと自動的に一番近い tsconfig.json を探してくれるそうです。（ここではわかりやすさのため明示的にパスを指定しました）

https://typescript-eslint.io/packages/parser/#project

### 不要importの削除

コード内で使用していないimportの自動削除をしてくれるプラグイン [eslint-plugin-unused-imports](https://github.com/sweepline/eslint-plugin-unused-imports) を追加します。未使用のインポートを手動で消す手間が省けるので、入れておくとかなり便利です。

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

importの順番をルール化してESLintに自動で並びかえてもらいましょう。
これは `eslint-plugin-import`の[import/order](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md)を使って設定できます。

並べ方やはそれぞれの好みやディレクトリ構成に応じて設定してください。
私はあまりこだわりがないので、次のような設定としています。

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

https://zenn.dev/knowledgework/articles/0994f518015c04#import%E3%81%AE%E8%87%AA%E5%8B%95%E6%95%B4%E5%88%97%EF%BC%88import%2Forder%EF%BC%89
https://zenn.dev/riemonyamada/articles/02e8c172e1eeb1

:::message
`eslint-plugin-import` は `eslint-config-next`に含まれているためパッケージのインストールや`plugins`の指定は不要です。（明示的にインストール&指定しても問題はありません）
:::

### Tailwind CSSに関する設定を追加

「Tailwindのクラス名の順番を揃えたい」「`pr-1 pl-1`を自動で`px-1`に変換して欲しい」などと思ったことはないでしょうか？
そんな悩みを解決するためのプラグイン[eslint-plugin-tailwindcss](https://github.com/francoismassart/eslint-plugin-tailwindcss) を追加します。
https://github.com/francoismassart/eslint-plugin-tailwindcss

これを入れることで

- クラス名の順序の整列
- 負の値の書き方の統一（`-top-[5px]` を `top-[-5px]`にする）
- ショートハンドの強制（`pr-1 pl-1` を `px-1`にする）
- 重複するプロパティ指定の警告（`p-2 p-3` のような重複に警告を出す）

などができるようになります。（詳しくは[ドキュメント](https://github.com/francoismassart/eslint-plugin-tailwindcss)を参照）

パッケージをインストールして、推奨ルール（`recommended`）をセットすれば設定が完了します。

```sh
npm i -D eslint-plugin-tailwindcss
```

```diff json:.eslintrc.json
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended-type-checked",
    "next/core-web-vitals",
+   "plugin:tailwindcss/recommended"
  ],
```

これでルールに従わない並び順や書き方をしているときにエラーや警告を出してくれるようになります。

:::message
Tailwind CSS公式のPrettierプラグイン（[prettier-plugin-tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)）も存在するのですが、こちらはクラス順のソートにしか対応していません。そのためここでは機能が充実しているESLintのプラグインを使っています。
:::

### Prettierと衝突するルールを無効化

ESLintでフォーマットに関するルールを設定していると、Prettierとバッティングしてしまう可能性があります。そんなバッティングを回避するため`eslint-config-prettier`を追加します。

```sh
npm i -D eslint-config-prettier
```

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
ここまで `plugins`に`@typescript-eslint`や`tailwindcss`、`import`を指定していないことに疑問を持った方がいるかも知れません。
実を言うと、これらは `plugins` に書いても書かなくても同じように動作します。

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

を実行すると、`next lint`の対象ファイルをチェックし、可能な箇所は自動修正してくれます。

### VSCode拡張機能の追加

VSCodeにESLintの拡張機能を追加します。これによりエディタ上に警告を出したり、ファイル保存時に自動修正が行えるようになります。

`dbaeumer.vscode-eslint` を VSCode にインストールして有効化してください。
https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint

これでエディタに ESLint のエラーが表示されるようになります。
さらに`.vscode/settings.json`を更新します。

```diff json:.vscode/settings.json
 {
+  "editor.codeActionsOnSave": {
+    "source.fixAll.eslint": "explicit" // ファイル保存時にESLintの自動修正を実行する
+  },
+  ...
 }
```

これでファイル保存時に自動修正が行われるようになります。

## MarkuplintでHTMLの構文をチェックする

HTMLの構文チェックをするため[Markuplint](https://markuplint.dev/ja/)を導入します。
アクセシビリティ向上のためにもぜひ入れておくことをおすすめします。
https://markuplint.dev/ja/

### Markuplintの導入

導入用のコマンドがあるのでそちらを使用します。

```sh
npx markuplint --init
```

対話形式の質問に答えることで、設定ファイルの作成から必要なパッケージのインストールまでを自動で実行してくれます。

```sh
✔ Which do you use template engines? · React (JSX)
✔ May I install them automatically? (y/N) · true
✔ Do you customize rules? (y/N) · false
✔ Does it import the recommended config? (y/N) · true
```

以上の内容で回答を進めると`.markuplintrc`という設定ファイルが作成され、次の3パッケージがインストールされます。

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
   "specs": {
-    "\\.[jt]sx?$": "@markuplint/react-spec"
+    ".tsx": "@markuplint/react-spec" // tsxのみの指定にする
   },
   "parser": {
-    "\\.[jt]sx?$": "@markuplint/jsx-parser"
+    ".tsx": "@markuplint/jsx-parser" // tsxのみの指定にする
   },
   "extends": [
-    "markuplint:recommended"
+    "markuplint:recommended-react" // recommended ＋ React用の追加ルール
   ]
 }
```

今回はJSファイルを使わない方針なので対象ファイルは`.tsx`のみとしました。（特に変更しなくても問題はありませんが一応）

また`extends`の設定を`markuplint:recommended-react`（React用の推奨プリセット）に変更しました。これは「基本プリセットを全て読み込んだ上で React 用のルールを追加する」という設定[^1]です。

:::message
Markuplintにはそれほどたくさんのルールがあるわけではないので、一度[ドキュメント](https://markuplint.dev/ja/docs/rules)に目を通しておくのがおすすめです。わりとサクッと読めます。
:::

:::details Markuplintのプリテンダー機能
Markuplintでは[プリテンダー](https://markuplint.dev/ja/docs/guides/besides-html#pretenders)という機能を利用して、コンポーネントごとに対応するHTML要素を定義することができます。頻出コンポーネントをできるだけ設定しておくことでMarkuplintにチェックしてもらえるので便利そうです。

まだきちんと使っていませんが、Nextの`<Image>`をimg要素、`<Link>`をa要素に紐付けるような設定をプロジェクトにデフォルトで入れておいても良いかもしれません。
https://markuplint.dev/ja/docs/guides/besides-html#pretenders
:::

[^1]: 現状では`No hard coding ID`というルールのみが追加で有効になるようです。これは要素のid属性をハードコードしている場合に警告が表示されるもので、IDの重複を防ぐためのルールです。これに関して制作者の[ゆうてん](https://zenn.dev/yusukehirao)さんが[解説記事](https://zenn.dev/yusukehirao/articles/id-attribute-strategy)を書いてくれています。

### Markuplint用のコマンドを追加

npm-scripts を追加しておきます。

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

### VSCode拡張機能と設定の追加

MarkuplintにもVSCode拡張機能があるので入れておきましょう。
`yusukehirao.vscode-markuplint`をVSCode にインストールして有効化してください。
https://marketplace.visualstudio.com/items?itemName=yusukehirao.vscode-markuplint

さらに`.vscode/settings.json`を更新します。

```diff json:.vscode/settings.json
 {
   ...
+  "markuplint.enable": true // Markuplintを有効化する
 }
```

これによってエディタ上で Markuplint の警告が表示されるようになります。

## Secretlintで秘匿情報の漏洩を防ぐ

[Secretlint](https://github.com/secretlint/secretlint)はAPIトークンや秘密鍵など、リポジトリにコミットしてはいけないデータがファイルに含まれていないかをチェックするツールです。
https://github.com/secretlint/secretlint
https://efcl.info/2020/03/24/secretlint/

「APIへのアクセストークンを誤って公開してしまった」というような事故を防ぐためにも導入しておきましょう。（これを入れておけば絶対安心というものではないのでご注意ください。念のため。）

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

`--secretlintignore .gitignore` で `.gitignore`に書かれたファイルを対象外としています。
`--maskSecrets` はエラーメッセージ内のシークレットを `***` でマスクしてくれるオプションです。

この状態で

```sh
npm run secretlint
```

を実行すると、対象ファイルにシークレットデータが無いかチェックしてくれます。
試しに次のようなコードを追加して secretlint を実行してみます。

```ts:secret.ts
const AWS_KEY = 'AKIAXXXXXXEXAMPLEKEY';
```

すると次のようなエラーが表示されます。

```sh
1:24  error  [AWSAccessKeyID] found AWS Access Key ID: ********************  @secretlint/secretlint-rule-preset-recommend > @secretlint/secretlint-rule-aws
```

SecretlintがAWSのアクセスキーを検出し、エラーを出してくれていますね。

## Husky + lint-staged でコミット前にコードをチェックする

ここまでコードチェックのためにさまざまな設定を追加してきました。
ここからは[Husky](https://typicode.github.io/husky/)と[lint-staged](https://github.com/lint-staged/lint-staged)を使ってコミット前にコードチェックを実行し、問題のあるコードがコミットされないような環境を整えていきます。

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

Huskyを初期化して、`pre-commit`用の設定ファイルを作成します。

```sh
npx husky init
```

`init`はHusky v9から追加されたコマンドで、以下の処理を実行してくれます。

- husky の初期化
- npm-scriptsの`prepare`コマンドに`husky`を追加（`npm install`時にHuskyの初期化を実行する設定）
- `pre-commit` 用の設定ファイル（`.husky/pre-commit`）を作成

作成された `.husky/pre-commit` を次のように更新します。

```sh
npx lint-staged
```

これでコミット前に `lint-staged` が呼び出されるようになります。

https://typicode.github.io/husky/get-started.html

### lint-staged の設定

プロジェクトルートに `.lintstagedrc.js` を作成し[Next.js のドキュメント](https://nextjs.org/docs/pages/building-your-application/configuring/eslint#lint-staged)にしたがってまずは以下の内容を記述します。

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

これでコミット前にステージされたファイルに対して`next lint --fix`が実行されることになります。
ここではさらに手を加えて次のようにします。

```diff js:.lintstagedrc.js
 const path = require("path");

 const buildEslintCommand = (filenames) =>
   `next lint --fix --file ${filenames
     .map((f) => path.relative(process.cwd(), f))
     .join(" --file ")}`;

 module.exports = {
+  "*": ["secretlint"],
+  "*.{js,cjs,mjs,json,ts,tsx,css}": ["prettier --write"],
+  "*.{ts,tsx}": ["bash -c tsc --noEmit", buildEslintCommand, "markuplint"],
 };
```

設定した内容は次の通りです。

| 対象ファイル                                    | 実行する内容                                                                                   |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| 全ファイル                                      | Secretlintの秘匿情報チェック                                                              |
| `.js` `.cjs` `.mjs` `.json` `.ts` `.tsx` `.css` | Prettierのフォーマット                                                                    |
| `.ts` `.tsx`                                    | TypeScriptのコンパイルチェック<br>ESLintの構文チェック<br>Markuplintの構文チェック |

この状態でファイルをコミットしようとするとそれぞれのファイルに対してチェックが実行され、エラーがある場合はコミットを中止してくれます。
（もしチェックを無視してコミットしたい場合、コミット時に `-no-verify` か `-n` フラグをつけることでスキップができます。)

:::details コミット時 `command not found` のエラーが出る場合

Volta や nvm などの Node.js バージョン管理ツールを使用しているとGUI（VSCodeのGitやSourceTreeなど）からのコミットの際 `command not found` のエラーが出るかもしれません。

その場合はHuskyの実行前にPATHを通す設定が必要になります。

筆者はvoltaを使用しているため、次のような内容で`~/.config/husky/init.sh`を作成しています。

```bash
export VOLTA_HOME="$HOME/.volta"
export PATH="$VOLTA_HOME/bin:$PATH"
```

詳しくは[Huskyのドキュメント](https://typicode.github.io/husky/how-to.html#node-version-managers-and-guis)を参照ください。

https://typicode.github.io/husky/how-to.html#node-version-managers-and-guis
https://zenn.dev/cureapp/articles/f2722061739b51
:::

## VSCode の設定

VSCodeに関する設定をさらにいくつか追加しておきます。

### Tailwind CSS IntelliSense（VSCode の拡張機能）を追加

`bradlc.vscode-tailwindcss` をインストールして有効化しておきましょう。
エディタ上での自動補完が効くようになってとても便利です。

https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss
https://tailwindcss.com/docs/editor-setup

### 拡張機能を設定ファイルに記載

`.vscode/extensions.json`というファイルに拡張機能のリストを指定しておきます
この状態で拡張機能の検索欄に`@recommended`と入力すると指定したの拡張機能が一覧で表示されます。
チームメンバー全員で環境を揃える際に便利なので設定しておきます。

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

### 未importモジュールの自動追加

ファイル保存時に未importのモジュールを自動で探してimportしてくれる便利な機能があるので有効化しておきましょう。
`.vscode/settings.json`に追記します。

```diff json:.vscode/settings.json
 {
   "editor.codeActionsOnSave": {
     "source.fixAll.eslint": "explicit", // ファイル保存時にESLintの自動修正を実行する
+    "source.addMissingImports": "explicit" // 未importのモジュールを自動で探してimport文を追加する
   },
   ...
 }
```

## プロジェクトに応じて追加する設定

### import の依存関係をルール化

「このコンポーネントはこのファイルからしか読み込めない」というような依存関係のルールを決めることができます。
[eslint-plugin-strict-dependencies](https://www.npmjs.com/package/eslint-plugin-strict-dependencies) というプラグインを利用して設定が可能です。
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

きちんと環境を整える中で多くのことを学んだ。

ESLint に関しては今後 flat config という新しい設定方法に変わっていく（？）ようなので、キャッチアップが必要そう。最近では rome や biome といったツールも出てきている。

さらにテストツールや CI の設定も学んで、より堅牢な環境を構築できるようになりたい。

コミット時は Prettier と Secretlint、CI で各種リントとコンパイラチェック、というように分けても良いかもしれない。

まだまだやれることはありそうなので引き続き高めていきたい。

Secretlint と Markuplint は情報少ないのでもっとできることありそう。

## 参考記事

本文中で紹介しきれなかった参考記事です。
https://qiita.com/tksst/items/bf62d50b25af69505e8e#9-typescriptの設定変更推奨度3
https://zenn.dev/chida/articles/bdbcd59c90e2e1
https://zenn.dev/t_keshi/scraps/9ddb388bc6975d
https://zenn.dev/rena_h/scraps/fd330154d02f76
https://zenn.dev/chot/articles/bcc178c481a9c2
https://zenn.dev/azukiazusa/articles/start-syntax-checking-with-markuplint
https://zenn.dev/risu729/articles/latest-husky-lint-staged
