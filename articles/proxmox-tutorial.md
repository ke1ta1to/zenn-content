---
title: "Proxmoxの運用Tips【自宅鯖】"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["proxmox", "linux"]
published: true
---

この記事はUEC Advent Calendar 2024の11日目の記事です。その2も合わせてどうぞ。

https://adventar.org/calendars/10127

https://adventar.org/calendars/10198

10日目の記事はあづみさんの「[Windowsのワザップ](https://azumino.pages.dev/blog/20240012-windows-tips/)」です。弊学はWindows逆張りLinuxユーザーがとても多いですが、もちろんWindowsは使いこなせるってことですよね？やべっと思ったら今すぐこの記事を読みにいきましょう。特に最後の方、私も知らない知見が多くて勉強になりました。

11日目はProxmoxの知見を共有したい記事です。

## Proxmoxについて少し

私の周りに自宅サーバーを始める人類がとっても多い。嬉しい限りですがインストールして適当に使っているだけでは勿体無いので、有益な情報を共有できればと思います。一応インストールの仕方も書きましたが、ポイントだけ説明しているのでLinuxとかサーバー初心者向けの記事ではありません。

## Proxmoxをとりあえずインストール

ここは読めば進められるところなので要点だけを説明します。

https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso
から最新のisoをダウンロードしてください。

RufusなどでUSBメモリに書き込んで、そこからブートしたらインストーラーが起動すると思います。

![](https://storage.googleapis.com/zenn-user-upload/03956ed9e9d9-20241212.png)

特別な理由がなければ、`Install Proxmox VE (Graphical)`を選択。あとはインストーラーに従ってネットワークの設定まで進んでください。

![](https://storage.googleapis.com/zenn-user-upload/bafb5dc6e4ba-20241212.png)

ネットワークの設定ですが、注意が必要です。IPの変更はかなり手間がかかります。特にクラスターを組んでいる時のIP変更は修正箇所が多くバグる可能性も高いのでおすすめしません（n敗）。なのでIPを事前に決めておくことをおすすめします。ここでは私の例を紹介します。

デフォルトゲートウェイ: `172.16.0.1`  
サブネットマスク: `255.240.0.0`

| IP           | 接続先 |
| ------------ | ------ |
| `172.17.1.0` | pve 01 |
| `172.17.1.1` | VM 101 |
| `172.17.1.2` | VM 102 |
| `172.17.2.0` | pve 02 |
| `172.17.2.1` | VM 201 |
| `172.17.2.2` | VM 202 |

一部ですが、このようにわかりやすいIPを振った方がいいです。ただし、ライブマイグレーションなど頻繁にする場合はこの方法は意味をなさないことには注意してください。DHCPでプライベートDNSなどでやりくりしたほうが良いでしょう。ただしPVE本体はDHCPでは基本的に対応できません。
ネットワークのIP構成の変更は痛い目を見るので、しっかり検討してください。
[RFC1918: Address Allocation for Private Internets](https://datatracker.ietf.org/doc/html/rfc1918)を参考に`/8`~`/24`で切ればいいと思います。ただしホスト部が大きすぎるとルーターへの負荷が否定できないのでバランスを見てください。

インストールが終了したら画面に表示されているURLへブラウザから入ってください。ユーザーネームは`root`です。
言語は日本語じゃなくて英語の方がいいです。理由はここらへんの分野で英語の方が良いとされているのと同じです。

## クラスターの構築

クラスターの構築は複数台のVMを立てるときに建てる必要がありますが、1台でも建てれるのでやって損はないと思います。
メリットとしては

> - Centralized, web-based management
> - Multi-master clusters: each node can do all management tasks
> - Use of pmxcfs, a database-driven file system, for storing configuration files, replicated in real-time on all nodes using corosync
> - Easy migration of virtual machines and containers between physical hosts
> - Fast deployment
> - Cluster-wide services like firewall and HA

があるようです。 [参照元: Cluster Manager](https://pve.proxmox.com/wiki/Cluster_Manager)

Server ViewからDatacenterを選択して、Cluter Informationに行ってください。
![](https://storage.googleapis.com/zenn-user-upload/d4e0fe955c03-20241211.png)

Create Clusterボタンから、クラスターを作成します。Cluster Nameは何でも構いません。Proxmox Wikiではprod-centralとしている例を見かけたので自分はdev-centralとしています。

![](https://storage.googleapis.com/zenn-user-upload/a087c4f10c12-20241211.png)

こんな感じですね。TASK OKになったら大丈夫です。
![](https://storage.googleapis.com/zenn-user-upload/9613182b2b32-20241211.png)
左上がDatacenter (指定した名前)になっていると思います。

サーバーが1台の場合はこれで大丈夫です。2台目以降ある場合はここからの手順でクラスターに参加してください。

まずクラスターを作成した方のコンソールからJoin Informationをクリックしてください。Copy Informationをクリックして参加情報をコピーしてください。

![](https://storage.googleapis.com/zenn-user-upload/2b63c85646be-20241212.png)

2台目以降もProxmoxを同じ手順でインストールした後、Join Clusterボタンから先ほどコピーしたJoin Informationをペーストし、パスワード等必要な情報を入力すれば、完了です。

![](https://storage.googleapis.com/zenn-user-upload/ea5299420758-20241212.png)

TASK OKの文字が出なくても大丈夫です。一度リロードした後、画面下のログのJoin Clusterのものを開いたら完了しているのがわかると思います。

ここまでの手順でどのPVEのIPでコンソールに入ってもDatacenterに全てのサーバー機が登録されていたら成功です。

![](https://storage.googleapis.com/zenn-user-upload/dbd1db3abf05-20241212.png)

## パッケージのアップデート

こんな感じで、ログがUpdate package databaseで真っ赤になっている人、対象です。

![](https://storage.googleapis.com/zenn-user-upload/3117eb86aeb9-20241212.png)

Proxmoxには[サブスクリプションプラン](https://www.proxmox.com/en/proxmox-virtual-environment/pricing)があります。デフォルトではEnterprise版が指定されていて、かつプロダクトキーを入力していない状態ということになりますので、このようなエラーが出ています。

無償版に切り替えが可能で、それによってこのエラーを消せるので必ずやってください。やらないとパッケージがアップデートできない気がします。
作業対象は全てのPVEです。複数機ある方は全てにおいて行なってください。

Server Viewから、PVEを選択し、Updates、Repositoriesを開くとこのような画面になると思います。

![](https://storage.googleapis.com/zenn-user-upload/212f72d07c8e-20241212.png)

The enterprise repository is enabled, but there is no active subscription!
"Enterpriseなのに有効なサブスクリプションがないよ！"って怒られていますね。

まず、不要なものを無効化します。簡単です。enterpriseって書いてあるやつを無効化すればいいですね。
`https://enterprise.proxmox.com/debian/ceph-quincy`と`https://enterprise.proxmox.com/debian/pve`です。
選択してDisableを押すだけです。

無効化した分、無償版をこれから追加します。Addから、`No-Subscription`がついてあるものを全て追加してください。
`No-Subscription`, `Ceph Quincy No-Subscription`, `Ceph Reef No-Subscription`, `Ceph Squid No-Subscription`です。Cephを組まない場合は最初の1つだけで良い気もします。

![](https://storage.googleapis.com/zenn-user-upload/fda5e11d6e4a-20241212.png)

Statusがこんな感じになっていたらOK！内容は、大丈夫そうですね。

Updatesタブから、パッケージの操作ができます。Refreshボタンは`apt update`、Upgradeボタンは`apt upgrade`をしてくれます。Upgradeに関しては手動で行わないといけないようなので、定期的に実行が必要です。

![](https://storage.googleapis.com/zenn-user-upload/5bb599837179-20241212.png)

以上です。忘れずに全てのPVEで行ってください。同クラスターなのにProxmoxのバージョンが違うとか、とっても怖いので。

## Cloudflareからのコンソールアクセス

[Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)を使用すれば、http(s)はポート解放をせずに外部から利用できるのとても便利です。ここではCloudflare Tunnelのはじめ方は説明しません。1つだけ伝えたいことがあります。
ネットワーク内、もしくはPVE内にVMを立てて`cloudflared`をインストールして、準備完了。
Public Hostnameでhttpsの`localhost:8006`または`PVEのIP:8006`を設定します。httpsなので間違いないように。

![](https://storage.googleapis.com/zenn-user-upload/f99c6c3f56ad-20241212.png)

あれ。原因は簡単です。httpsの証明書がないからです。どこにも書いてないのでわかりにくいですね。
ここで[オレオレ証明書](https://www.gmo.jp/security/ciphersecurity/http-https/blog/oreore-certificate/)を作成したり、nginxなどでリバプロ運用でも良いですが、個人利用なら以下の方法で対処すれば良いと思います。

Public Hostnameから、Additional application settings>TLS>No TLS VerifyをONにしてください。証明書の認証を無効にしてくれます。

![](https://storage.googleapis.com/zenn-user-upload/5c13e31902f9-20241212.png)

以上です。登録したドメインでアクセスできるようになります。

## Cloudflare Tunnelの運用

全てのVMに`cloudflared`を入れるという選択肢もありますが、クラスター内で1つインストールしてprivate ipを入れてやれば良いと思います。

![](https://storage.googleapis.com/zenn-user-upload/5382fc25c2f9-20241212.png)

自分はこんな感じです。ポート番号で何してるのかバレて怖いですね

## Proxmox on Proxmox

この記事を書くためにProxmox on Proxmoxをしたのでメモしておきます。

普通にVMを立てますが、CPUの設定の時に`type`を`host`にしてやってください。これだけでいけます。

![](https://storage.googleapis.com/zenn-user-upload/f4d73fd8a2b3-20241212.png)

Proxmox on Proxmox on Proxmox on Proxmox on Proxmox on Proxmox on Proxmox on...
はい、やめておきましょう。

Proxmox内で仮想ネットワークを立ててやればさらにProxmoxの検証が捗る気がします。成功したら追記するかもです。

## まとめ

自宅鯖仲間が増えることはとっても良いこととされているので、少しでも興味を持ったらとりあえずインストールしてください。時間がないとか言い訳はいいです。一旦インストールしましょう一旦ね。
明日の記事はInさんによる「木に関するアルゴリズムを実装したい。」です。木に関するアルゴリズム、木になります。何かと逃れられないところですからね、楽しみであります。

<!-- ## Ceph? -->
