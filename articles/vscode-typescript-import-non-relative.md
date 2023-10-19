---
title: "絶対パスで自動インポートしてくれないときの対処"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "vscode"]
published: true
---

tsconfigのbaseUrlやpathsを設定すれば、`@/xxx/yyy`のように絶対パスを指定できます。  
しかし、VSCodeの自動インポートを使用すると`./hoge`のようにパスを指定されてしまいます。絶対パスで統一されていた方が気持ちいので統一してももらいましょう。

## VSCodeの設定

`setting.json`の`typescript.preferences.importModuleSpecifier`を`non-relative`にしてください。

## オプションの詳細

VSCodeのポップアップに出てきたものそのままです。(2023/08/10)

| オプション         | 動作                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| shortest (Default) | 相対インポートよりもパス セグメント数が少なくなる場合にのみ、非相対インポートを優先します。                               |
| relative           | インポートされたファイルの場所への相対パスを優先します。                                                                  |
| non-relative       | `jsconfig.json` または `tsconfig.json` に構成されている `baseUrl` または `paths` に基づいて非相対インポートを優先します。 |
| project-relative   | 相対インポート パスでパッケージまたはプロジェクト ディレクトリが提供される場合にのみ、非相対インポートを優先します。      |
