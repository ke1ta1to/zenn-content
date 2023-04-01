---
title: "AWS Route53でマイクラサーバーのDNS設定"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "route53", "minecraft"]
published: true
---

# はじめに

１つのサーバーで複数のマイクラサーバーを開いてるとき、25565 以外のポートを利用することになりますが、IP が example.com:xxxxx なんてダサいですよね。
test1.domain.com -> 25566 ポート
test2.domain.com -> 25567 ポート
にアクセスすることを目標とします。
:::message
BungeeCord は利用しないこととします
:::
:::message
マイクラサーバーの構築方法などは詳しく解説しません
:::

# server.properties の設定

それぞれのサーバーで設定してください

```:server.properties
server-port=25566
```

```:server.properties
server-port=25567
```

# 接続確認

マイクラを開いて、`サーバーIP:25566`、`サーバーIP:25567`でそれぞれアクセスできるか確認してください。
できない場合はポート開放ができているか確認してください。

# Route53 での設定

ホストゾーンを作成して、ドメインの設定が利用可能になっていることとして進めます。
対象のホストゾーンを選択して、`レコードの作成`を選択します。
以下の３つのレコードを作成します。
:::message
自分が設定したいドメインやポート番号に置き換えてください
:::

## proxy.domain.com

| レコード名 | レコードタイプ | 値                                         |
| ---------- | -------------- | ------------------------------------------ |
| `proxy`    | A              | xxx.xxx.xxx.xxx（サーバーのグローバル IP） |

![](https://storage.googleapis.com/zenn-user-upload/f1086468fe73-20230401.png)

## test1.domain.com

| レコード名              | レコードタイプ | 値                            |
| ----------------------- | -------------- | ----------------------------- |
| `_minecraft._tcp.test1` | SRV            | `1 10 25566 proxy.domain.com` |

![](https://storage.googleapis.com/zenn-user-upload/64c43796de41-20230401.png)

## test2.domain.com

| レコード名              | レコードタイプ | 値                            |
| ----------------------- | -------------- | ----------------------------- |
| `_minecraft._tcp.test2` | SRV            | `1 10 25567 proxy.domain.com` |

![](https://storage.googleapis.com/zenn-user-upload/d307b64fc72a-20230401.png)

# 動作確認

マイクラから、`test1.domain.com`、`test2.domain.com`でアクセスできるか確認してください。

以上で設定完了です。
