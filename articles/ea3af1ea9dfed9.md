---
title: "Rails7にてesbuildとtailwind使用時にカスタムCSSを導入するには"
emoji: "🍃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['jsbundling-rails', 'esbuild', 'cssbundling-rails', 'tailwind', 'rails7']
published: true
---

## 前提となる使用技術
- Rails7
- jsbundling-rails with esbuild
- cssbundling-rails with tailwind

## 話したいこと
cssbundling-railsにてtailwindモードを利用している場合、`application.tailwind.css`の内容をバンドルして`app/assets/builds/application.css`にまとめてくれる。しかし、初期状態では例えばオリジナルであなたが`app/assets/stylesheets`配下に新たに`custom.css`を追加して`application.tailwind.css`に`@import 'custom'`としてもオリジナルのcssファイルを読み込むことはできない。

ref: [github cssbundling-rails](https://github.com/rails/cssbundling-rails#how-do-i-import-relative-css-files-with-tailwind)

> If you want to use @import statements as part of your Tailwind application.js file, you need to configure Tailwind to use postcss and then postcss-import. But you should also consider simply referring to your other CSS files directly, instead of bundling them all into one big file. It's better for caching, and it's simpler to setup. You can refer to other CSS files by expanding the stylesheet_link_tag in application.html.erb like so: <%= stylesheet_link_tag "application", "other", "styles", "data-turbo-track": "reload" %>.


```css:application.tailwind.css
@tailwind base;
@tailwind components;
@tailwind utilities;
@import 'custom' // custom.cssを読み込もうとしても読み込めない
```

そこで本記事では、上の技術スタックの時に、いかにしてtailwindとオリジナルCSSを両立させるかを話したい。
また、本記事は[Setup TailwindCSS and esbuild on Rails 7](http://www.ahmednadar.com/posts/setup-tailwindcss-and-esbuild-on-rails-7)を参照している。

## なぜ読み込めないか、どのように解決するか
cssbundling-railsのtailwindモードを利用している時、nodeベースのtailwindCSSはcssbundling-railsによって提供されているのだが、このcssbundling-railsがデフォルトではcssのインポートを許容していないからである。  
そこで、postcssを使ってこの問題を解決することができる。

## ステップ
### 1. postcssとその関連ライブラリのインストール

```shell:terminal
> yarn add postcss postcss-flexbugs-fixes postcss-import postcss-nested 
```

### 2. postcss.config.jsファイルを作成&編集

```shell:terminal
> touch postcss.config.js
```

```js:postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss'),
    require('autoprefixer'),
    require("postcss-nested"),
    require("postcss-flexbugs-fixes"),
  ]
}
```

### 3. `app/assets/stylesheets/application.tailwind.css`を編集

```css:application.tailwind.css
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';
/* app components */
@import './custom.css';
```

以上のステップであなたが作成したオリジナルのCSSもバンドルされてbuilds配下の`application.css`にまとめられて、CSS適用が可能になるはずである。

ありがとうございました！
