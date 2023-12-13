---
title: "調布祭にモバイルオーダーを導入した話"
emoji: "📱"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "firebase", "strapi"]
published: false
---

この記事は [UEC 2 Advent Calendar 2023](https://adventar.org/calendars/8704) 15日目の記事です。

https://adventar.org/calendars/8704

昨日はKanaruさんの小さなコンピュータ入門でした。

この記事では調布祭のteam411の店舗で導入したモバイルオーダーシステム（以下MO）について~~宣伝~~紹介します。見かけた方は多いのではないでしょうか。これです。

![](https://storage.googleapis.com/zenn-user-upload/0df110fbb308-20231213.jpg)

ベイクドモチョチョについては深く触れないこととします。

## 経緯

元々はU☆PoCで応募したものでした。

https://www.uec.ac.jp/research/venture/contest.html

![](https://storage.googleapis.com/zenn-user-upload/af632ae9e8d8-20231213.png =250x)
_U☆PoCで使用したポスター_

なんとチームメンバーが考えてくれたアイディアが天才的なこともあって企業賞を受賞することができました。はい、つまり実装はほぼ私がやりました。~~チーム開発とは。~~ 今回のMO導入はU☆PoCの一環でもありました。

## プロトタイプの作成

先ほどのU☆PoCも諸々関係してきますが、細かいことは飛ばしてとりあえずプロトタイプを作成しました。9月の話ですね。

Firebase愛好家（依存症）の私が実装したこともあり、Firebase Auth、Firestoreなどで実装。言ってしまえばMOはFirebaseを装飾しただけです。フロントはNext.js on Vercel。この時代はまだPages Routerが主流だった気がする。もうApp Routerが浸透してるっていうんだからフロントの進化の速度は怖い。デザインはできる訳もなくMUIを採用。というかデザイナー募集。MUIいいですよね。同じ世界線に生きる人に「うわ、これMUIじゃん」って思われること以外は悪いことはないと思います。

プロトタイプで作成したのはこんなやつですね。ポスター切り抜き。商品の画像が酷いですが、これは私の仕業です。よく見ると後で受け取るとかも選べるようになってますね。消えた機能です。

![](https://storage.googleapis.com/zenn-user-upload/7844d2a477d6-20231213.png)

## いざ実装へ

プロトタイプで得た知見を参考にしつつ調布祭で使用するシステムの開発に移りました。構成は以下のようにしました。ツッコミどころが多いかと思いますが。