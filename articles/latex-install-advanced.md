---
title: "少し頑張るLaTeX環境構築"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["latex", "devcontainer"]
published: true
---

## はじめに

入学して早々LaTeXのインストールを急かされる大学があるそうですが．
世の中にはLaTeXの記事が無限にあって，初心者お断り仕様です．さらにこの記事が投稿されてもっと複雑に！
さて，この記事の主題は「Dev ContainerでLaTeXしよう」です
LaTeXはMacとかWindowsとかでインストール方法が異なって，さらに個人のパソコンの状態によってインストールに失敗したりします！やだ！
ところでDev Containerを使うと「デバイスに依存しない」「ローカル環境を汚染しない」開発ができます．
おっと．つまりそういうことですね．やるしかない

## 対象読者

ちょっと待った．本当にこの記事に沿ってインストールする？
**とにかくLaTeXが書ければいい！** という方はこっちの記事をおすすめします
https://zenn.dev/e_chan1007/articles/8029f3f9dff2be

それでもこっちを読んでくれるあなたに，この記事のタイトル「少し頑張る」をすると以下のメリットがあるよ

- PCのローカル環境が汚れない
- インストールに失敗しない（Docker，VSCodeとかのインストールは知らん）
- なんかかっこいい！！（重要）

また，パソコン初心者向けに書いても需要が空集合になると察したので，VSCodeとか触れるよ！そんな人向けに書きます

## 必要なインストール

[Dev Containers tutorial](https://code.visualstudio.com/docs/devcontainers/tutorial)ができればOK！

### インストール詳細

[Dev Containers tutorial](https://code.visualstudio.com/docs/devcontainers/tutorial)ができてたら必要はないけど一応．

- [Docker Desktop](https://www.docker.com/ja-jp/products/docker-desktop/)
  - ターミナルから`docker --version`を実行して確認
- [Visual Studio Code](https://code.visualstudio.com/Download)
  - [Dev Containers拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)を入れる

## 構成ファイルを入れる

Dev Containerのテンプレートを入れよう！

1. 適当にディレクトリを作って，VSCodeでそこを開いた状態にしておいてください．
2. VSCodeの左下の「><」みたいなのを押して，「開発コンテナー構成ファイルを追加...」を選択
3. 「ワークスペースに構成ファイルを追加」の方がおすすめ
4. 一番上の入力ボックスに`ghcr.io/eguchi1611/texlive-sci-ja/template`を入力
5. 機能の選択画面はそのままスルー
6. こんな感じになったらOK！
   ![](https://storage.googleapis.com/zenn-user-upload/1e8b1a55def5-20240329.png =400x)
7. 右下の「コンテナーで再度開く」を選択

以上ですね．VSCodeでターミナルを開いて`uplatex -version`で確認してください

## 快適なLaTeXライフを

- 適当にディレクトリを作って複数のドキュメントを管理
- Gitを入れて管理([.gitignore](https://github.com/github/gitignore/blob/main/TeX.gitignore))

## 最後に

ここまで読んでいただきありがとうございます！役に立っていたら幸いです．質問等はコメントへ
詰まったらサンプルを作成しておいたので参考にしてください
https://github.com/eguchi1611/latex-devcontainer-sample

## 雑談

これから書く以下

- なんでこんな複雑なんだよ
- Dockerイメージの開発
- 開発コンテナのテンプレートの作成
