---
title: "少し頑張る理数系LaTeX環境構築 (Dev Container)"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["latex", "devcontainer"]
published: true
publication_name: "team411"
---

お急ぎの方 → [構成ファイルを入れる](#構成ファイルを入れる)

## はじめに

入学して早々LaTeXのインストールを急かされる大学があるそうですが．
世の中にはLaTeXの記事が無限にあって，初心者お断り仕様です．さらにこの記事が投稿されてもっと複雑に！
さて，この記事の主題は「Dev ContainerでLaTeXしよう」です（もっというと自分用にイメージを作ろうデス）
LaTeXはMacとかWindowsとかでインストール方法が異なって，さらに個人のパソコンの状態によってインストールに失敗したりします！やだ！
ところでDev Containerを使うと「デバイスに依存しない」「ローカル環境を汚染しない」開発ができます．
おっと．つまりそういうことですね，やるしかない

## 対象読者

ちょっと待った．本当にこの記事に沿ってインストールする？
**とにかくLaTeXが書ければいい！** という方はこっちの記事をおすすめします
https://zenn.dev/e_chan1007/articles/8029f3f9dff2be

それでもこっちを読んでくれるあなたに，この記事のタイトル「少し頑張る」をすると以下のメリットがあるよ

- PCのローカル環境が汚れない
- インストールに失敗しない（Docker，VSCodeとかのインストールは知らん）
- インストール時間が短い^[TeX Liveのミラーは主に激遅だが，これはGHCRからダウンロードするため]
- なんかかっこいい！！（重要）

また，パソコン初心者向けに書いても需要が空集合になると察したので，VSCodeとかある程度触れるよ！そんな人向けに書きます

## 必要なインストール

この記事通りに進めればOK！（英語に負けるな）

https://code.visualstudio.com/docs/devcontainers/tutorial

## 構成ファイルを入れる

Dev Containerの構成ファイルを選ぶフェーズで以下を入力してください

```
ghcr.io/ke1ta1to/texlive-sci-ja/template
```

![](https://storage.googleapis.com/zenn-user-upload/c5d9466c8d8a-20240401.png)

以上ですね．Dev Containerを開いてターミナルから`uplatex -version`で確認してください

## 最後に

思ったより書くことがありませんでした．Dev Containerはシンプルでいいね！
意外と洗練されたLaTeXのDev Containerが見つからなかったのでこの機会に作成しました．
この開発過程も下に記しておくので興味ある方は是非．
ここまで読んでいただきありがとうございます！役に立っていたら幸いです．質問等はコメントへ

## 番外編

### Dockerイメージの開発

イメージの開発にかなり時間を割きました．私の結論（思い出）はこのリポジトリに詰まっています．
https://github.com/ke1ta1to/texlive-sci-ja
せっかくなので解説します．私の本当の環境構築のおすすめは，自分好みのイメージを作ることです．これを参考にどうですか？

まずベースイメージの選定ですが，[`buildpack-deps:bookworm-scm`](https://hub.docker.com/_/buildpack-deps/)です．[デフォルトテンプレ群](https://hub.docker.com/_/microsoft-vscode-devcontainers)のイメージのベースがこれだからです．
具体的には`mcr.microsoft.com/devcontainers/base`がベースですが，これはただ`buildpack-deps`と，後述するfeaturesを合体させてるだけです．

次はDockerfileの作成です．世の中にはTeX LiveのDockerイメージが大量に落ちています．それを片っ端から読み漁りました．
こうなりました．

```dockerfile:Dockerfile
FROM buildpack-deps:bookworm-scm

ARG TEXLIVE_MIRROR=https://tug.ctan.org/systems/texlive/tlnet

ENV PATH=/usr/local/texlive/bin:$PATH

WORKDIR /texlive-install

COPY ./texlive.profile .

# texliveをインストール
RUN wget ${TEXLIVE_MIRROR}/install-tl-unx.tar.gz \
  && tar -xf install-tl-unx.tar.gz --strip-components 1 \
  && ./install-tl -profile texlive.profile --location ${TEXLIVE_MIRROR} \
  && ln -sf /usr/local/texlive/*/bin/* /usr/local/texlive/bin

# latexmkのインストール
RUN tlmgr install latexmk

# GhostscriptとGnuplotをインストール
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
  && apt-get -y install --no-install-recommends ghostscript gnuplot \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/texlive/texmf-local/tex/latex/gnuplot

# gnuplot-lua-tikzをインストール
RUN gnuplot -e 'set term tikz createstyle' \
  && mktexlsr

WORKDIR /

RUN rm -rf /texlive-install
```

以下ポイント

- TeX Liveはミラーによって速度が変わりまくるのでARGでミラーを選択できるように
- インストールされるパスに2023などのバージョンが入るのでシンボリックリンクと環境変数でいい感じに調整

さてインストールプロファイルを作成しましょう．イメージのサイズを下げるために厳選してあります．理数系と言っている根拠はここなので，自分好みに変えるチャンスポイントです．私のイメージではこんな感じです．

```:texlive.profile
# https://tug.org/texlive/doc/install-tl.html#PROFILES
selected_scheme scheme-custom

collection-basic 1
collection-fontsrecommended 1
collection-langcjk 1
collection-langjapanese 1
collection-latex 1
collection-latexextra 1
collection-latexrecommended 1
collection-mathscience 1
collection-pictures 1

tlpdbopt_install_docfiles 0
tlpdbopt_install_srcfiles 0
```

ここまででDockerのイメージは完成ですね．でもDev Containerのイメージの作成は一工夫必要です．
`Common Utilities`という[Dev Container Features](https://containers.dev/features)が必要です．これを入れるとターミナルがいい感じに今のブランチ名を出してくれたり，良い恩恵が得られます．公式のDev Containerイメージはこれが入っています．こいつも含めた状態でイメージをビルドしたいですね．GitHub Actionsで済ませます．
Actionsでイメージのビルドといったら！`docker/build-push-action`ではありません．Dev Containerのイメージのビルド用Actions^[[Dev Container CLI](https://github.com/devcontainers/cli)のbuildをActionsにしてくれたやつ]があります．`devcontainers/ci`ですね．これを使ってビルドすると，Dev ContainerのFeatures，もっというとVSCodeの拡張機能と設定も全部まとめて1つのイメージにしてくれます！ナイス！
私のは以下のdevcontainer.jsonが適用された状態のイメージが出てきますよ．

```json:devcontainer.json
{
  "name": "TeX Live",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "features": {
    "ghcr.io/devcontainers/features/common-utils:2": {
      "username": "vscode"
    },
    "ghcr.io/devcontainers/features/git:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": ["James-Yu.latex-workshop"],
      "settings": {
        "latex-workshop.latex.recipe.default": "latexmk (latexmkrc)"
      }
    }
  },
  "remoteUser": "vscode"
}
```

Actionsファイルはこうです

```yaml:build.yml
name: "Pre-build TeX Live Dev Container Image"

on:
  push:
    branches:
      - "main"
  schedule:
    - cron: "0 0 * * 0" # 毎週日曜 (UTC)
  workflow_dispatch:

env:
  REGISTRY_IMAGE: ghcr.io/ke1ta1to/texlive-sci-ja

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Pre-build dev container image
        uses: devcontainers/ci@v0.3
        env:
          BUILDX_NO_DEFAULT_ATTESTATIONS: true
        with:
          imageName: ${{ env.REGISTRY_IMAGE }}
          cacheFrom: ${{ env.REGISTRY_IMAGE }}
          platform: linux/amd64,linux/arm64
          subFolder: ./src
          push: always
```

これでイメージが完成です！

### Dev Containerのテンプレートの作成

いざイメージができても，毎回`devcontainer.json`ファイルを手書きするのはスマートではないですね．公式のテンプレートは自動で色々ファイルを吐き出してくれます．それを作ろう．
GitHubのActions，`devcontainers/action`です．こいつがいい感じにしてくれる．
公式が分かりやすいので貼っておきますね
https://github.com/devcontainers/template-starter

以上！番外編の方が長くなってしまった．
いい感じのLaTeXライフが送れますように．
