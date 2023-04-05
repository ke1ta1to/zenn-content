---
title: "prettier-plugin-tailwindcssと他プラグインの競合"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tailwindcss", "prettier"]
published: true
---

# エラー状況

`prettier-plugin-tailwindcss`と以下のプラグイン等を導入した際に、一部もしくは全てのプラグインが動作しないことが確認できています。

- `@prettier/plugin-php`
- `@prettier/plugin-pug`
- `@shopify/prettier-plugin-liquid`
- `@ianvs/prettier-plugin-sort-imports`
- `@trivago/prettier-plugin-sort-imports`
- `prettier-plugin-astro`
- `prettier-plugin-css-order`
- `prettier-plugin-import-sort`
- `prettier-plugin-jsdoc`
- `prettier-plugin-organize-attributes`
- `prettier-plugin-organize-imports`
- `prettier-plugin-style-order`
- `prettier-plugin-svelte`
- `prettier-plugin-twig-melody`

# 解決方法

`prettier-plugin-tailwindcss`を最後に読み込む必要があります。

`pluginSearchDirs`オプションをオフにして、プラグイン読み込み設定を`.prettierrc`に追記する必要があります。

```json:.prettierrc
{
  // ...
  "plugins": [
    "prettier-plugin-svelte",
    "prettier-plugin-organize-imports",
    "prettier-plugin-tailwindcss" // 最後に読み込む
  ],
  "pluginSearchDirs": false
}
```

# 参考

https://github.com/tailwindlabs/prettier-plugin-tailwindcss#compatibility-with-other-prettier-plugins
https://tailwindcss.com/blog/automatic-class-sorting-with-prettier
