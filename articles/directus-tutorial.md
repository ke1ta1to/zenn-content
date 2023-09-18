---
title: "DirectusをSelf-Hostedしてみる"
emoji: "🐰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["directus", "cms", "docker"]
published: true
publication_name: "team411"
---

## はじめに

セルフホスティング型のHeadlessCMSとして[Strapi](https://github.com/strapi/strapi)、[Directus](https://github.com/directus/directus)、[Ghost](https://github.com/TryGhost/Ghost)あたりが候補にあると思います。しかし日本語のDirectusの情報がほぼないのと、使ってみて結構良かったので環境構築までをまとめます。Dockerの基本的な知識はあるものとして進めます。

## Directusについて軽く

[Quickstart Guide](https://docs.directus.io/getting-started/quickstart.html)から分かるように、Directus CloudというCasS型とSelf-Hosted型でDirectusを利用できます。今回はSelf-Hosted型で進めていきます。

[公式ドキュメント](https://docs.directus.io/self-hosted/quickstart.html)ではDockerで進めていますが、ソースからビルドして実行もできるようです。基本的にはExtensionsを作成してカスタマイズすれば良さそうなのでその必要はなさそうです。

## 最小構成のdocker-compose

公式ドキュメントの[Create a Docker Compose File](https://docs.directus.io/self-hosted/quickstart.html#create-a-docker-compose-file)に載っていますが、これを実行すれば最小構成で起動できます。

```yaml:docker-compose.yml
version: '3'
services:
  directus:
    image: directus/directus:latest
    ports:
      - 8055:8055
    volumes:
      - ./database:/directus/database
      - ./uploads:/directus/uploads
    environment:
      KEY: 'replace-with-random-value'
      SECRET: 'replace-with-random-value'
      ADMIN_EMAIL: 'admin@example.com'
      ADMIN_PASSWORD: 'd1r3ctu5'
      DB_CLIENT: 'sqlite3'
      DB_FILENAME: '/directus/database/data.db'
      WEBSOCKETS_ENABLED: true
```

`localhost:8055`にアクセスして、`admin@example.com`と`d1r3ctu5`でログインできます。今回は環境構築の話なので使用方法は飛ばしますが、設定から日本語表示にすることができるので難しいことはないと思います。翻訳が若干甘いので気をつけてください。

## データベースを変更する

デフォルトではSQLiteが使用されます。手軽に使える利点もありますが、セキュリティ面やパフォーマンスに限界があるのでMySQLやPostgreSQLを使用するよう変更します。

PostgreSQLの例を示します。その他DBの場合は適宜合わせてください。

docker-composeでPostgreSQLを起動します。docker-composeの例を示しておきます。

```yaml:docker-compose.yml
version: '3.8'

services:

# ・・・

  db:
    image: postgres:latest
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_USER: 'postgres'
      POSTGRES_DB: 'postgres'
    ports:
      - 5432:5432

# ・・・

volumes:
  postgres-data:
```

また、以下をdocker-composeのdirectusのenvironmentに追加してください。公式ドキュメントの [Database](https://docs.directus.io/self-hosted/config-options.html#database)を参考にしてください。

| Key         | Value                                        |
| ----------- | -------------------------------------------- |
| DB_CLIENT   | `postgres`                                   |
| DB_HOST     | `db`（サービス名に合わせる）                 |
| DB_PORT     | `5432`（公開したポートに合わせる）           |
| DB_DATABASE | `postgres`（dbの`POSTGRES_DB`に合わせる）    |
| DB_USER     | `postgres` （dbの`POSTGRES_USER`に合わせる） |
| DB_PASSWORD | `postgres`（dbの`POSTGRES_USER`に合わせる）  |

ここまでのdocker-compose.ymlは以下のようになります。

```yaml:docker-compose.yml
version: '3.8'

services:
  directus:
    image: directus/directus:latest
    ports:
      - 8055:8055
    volumes:
      - ./extensions:/directus/extensions
      - ./uploads:/directus/uploads
    depends_on:
      - db
    environment:
      DB_CLIENT: 'postgres'
      DB_HOST: 'db'
      DB_PORT: '5432'
      DB_DATABASE: 'postgres'
      DB_USER: 'postgres'
      DB_PASSWORD: 'postgres'

  db:
    image: postgres:latest
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_USER: 'postgres'
      POSTGRES_DB: 'postgres'
    ports:
      - 5432:5432

volumes:
  postgres-data:
```

`docker compose up`で起動し、DBのマイグレーションが終了し正常に起動することを確認してください。

ここまででDirectusのデータベースにPostgreSQLを利用できるようになりました。

## ファイルストレージにS3を利用する

デフォルトではサーバー内にファイルが保存されます。`S3`、`Google Cloud Storage`、`Azure`、`Cloudinary`、`Supabase`などのストレージも利用できるようです。ここでは`S3`を利用する例を示しておきます。公式ドキュメントは[File Storage](https://docs.directus.io/self-hosted/config-options.html#file-storage)です。

公式ドキュメントにもありますが、**環境変数の大文字小文字が区別されます。**`s3`と`S3`の箇所がありますので注意してください。また、複数のファイルストレージを利用できます。詳細は公式ドキュメントを参考にしてください。

AWSのコンソールからS3バケットとそれにアクセスできるロールを持ったIAMユーザーが必要です。以下のサイトなどが参考になるかと思います。必要があれば詳細を追記します。

https://www.jpcyber.com/support/minimal-iam-policy-to-access-amazon-s3

以下を`directus`のenvironmentに追加してください。

| Key               | Value                                                    |
| ----------------- | -------------------------------------------------------- |
| STORAGE_LOCATIONS | `s3`                                                     |
| STORAGE_S3_DRIVER | `s3`                                                     |
| STORAGE_S3_KEY    | IAMのアクセスキー                                        |
| STORAGE_S3_SECRET | IAMのSecret                                              |
| STORAGE_S3_BUCKET | S3のバケット名                                           |
| STORAGE_S3_REGION | S3のバケットがあるリージョン名 <br> `ap-northeast-1`など |

また、`STORAGE_S3_ROOT`を設定すればサブディレクトリ以下にファイルが保存されるので環境別に分けたい場合などに利用できます。

ファイルライブラリーからファイルをアップロードして、S3のダッシュボードに表示されれば成功です。うまくポリシーが当たってないとエラーが出ます。公式ドキュメントには特に記載がありませんでしたので、必要に応じて割り当ててください。

ここまででdocker-compose.ymlは以下のようになります。volumesでuploadsフォルダをマウントしていましたが、S3を利用する場合は不要です。

```yaml:docker-compose.yml
version: '3.8'

services:
  directus:
    image: directus/directus:latest
    ports:
      - 8055:8055
    volumes:
      - ./extensions:/directus/extensions
    depends_on:
      - db
    environment:
      DB_CLIENT: 'postgres'
      DB_HOST: 'db'
      DB_PORT: '5432'
      DB_DATABASE: 'postgres'
      DB_USER: 'postgres'
      DB_PASSWORD: 'postgres'
      STORAGE_LOCATIONS: 's3'
      STORAGE_S3_DRIVER: 's3'
      STORAGE_S3_KEY: '[IAMのアクセスキー]'
      STORAGE_S3_SECRET: '[IAMのSecret]'
      STORAGE_S3_REGION: '[バケットがあるリージョン]'
      STORAGE_S3_BUCKET: '[バケット名]'

  db:
    image: postgres:latest
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_USER: 'postgres'
      POSTGRES_DB: 'postgres'
    ports:
      - 5432:5432

volumes:
  postgres-data:
```

## メールサーバーを設定する

アカウントのパスワードをリセットしたいときなど、メールサーバーを設定する必要があります。公式ドキュメントは [Email](https://docs.directus.io/self-hosted/config-options.html#email)です。

sendmailなどでサーバーを立てても良いですが、今回は簡単に実装できるGmailを利用します。 [他のメール プラットフォームで Gmail のメールをチェックする](https://support.google.com/mail/answer/7126229?hl=ja#zippy=%2C%E3%82%B9%E3%83%86%E3%83%83%E3%83%97-imap-%E6%A9%9F%E8%83%BD%E3%81%8C%E6%9C%89%E5%8A%B9%E3%81%AB%E3%81%AA%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%81%93%E3%81%A8%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B%2C%E3%82%B9%E3%83%86%E3%83%83%E3%83%97-%E3%83%A1%E3%83%BC%E3%83%AB-%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E3%81%A7-smtp-%E3%81%A8%E3%81%9D%E3%81%AE%E4%BB%96%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%82%92%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B)を参考に、必要な情報を得てください。

必要な環境変数は以下の通りです。directusのenvironmentに追記してください。

| Key                 | Value                  |
| ------------------- | ---------------------- |
| EMAIL_FROM          | 送信元のメールアドレス |
| EMAIL_TRANSPORT     | `smtp`                 |
| EMAIL_SMTP_HOST     | `smtp.gmail.com`       |
| EMAIL_SMTP_PORT     | `456`                  |
| EMAIL_SMTP_USER     | 送信元のメールアドレス |
| EMAIL_SMTP_PASSWORD | アカウントのパスワード |

ここまででdocker-compose.ymlは以下のようになります。

```yaml:docker-compose.yml
version: '3.8'

services:
  directus:
    image: directus/directus:latest
    ports:
      - 8055:8055
    volumes:
      - ./extensions:/directus/extensions
    depends_on:
      - db
    environment:
      DB_CLIENT: 'postgres'
      DB_HOST: 'db'
      DB_PORT: '5432'
      DB_DATABASE: 'postgres'
      DB_USER: 'postgres'
      DB_PASSWORD: 'postgres'
      STORAGE_LOCATIONS: 's3'
      STORAGE_S3_DRIVER: 's3'
      STORAGE_S3_KEY: '[IAMのアクセスキー]'
      STORAGE_S3_SECRET: '[IAMのSecret]'
      STORAGE_S3_REGION: '[バケットがあるリージョン]'
      STORAGE_S3_BUCKET: '[バケット名]'
      EMAIL_FROM: '[送信元のメールアドレス]'
      EMAIL_TRANSPORT: 'smtp'
      EMAIL_SMTP_HOST: 'smtp.google.com'
      EMAIL_SMTP_PORT: '456'
      EMAIL_SMTP_USER: '[送信元のメールアドレス]'
      EMAIL_SMTP_PASSWORD: '[Gmailのパスワード]'

  db:
    image: postgres:latest
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_USER: 'postgres'
      POSTGRES_DB: 'postgres'
    ports:
      - 5432:5432

volumes:
  postgres-data:
```

## 本番環境で利用する

本番環境で利用するためには環境変数に`PUBLIC_URL`を設定する必要があります。公開するURLを設定してください。

## `docker-compose.yml`の完成系

以上を踏まえて以下のようになりました。

```yaml:docker-compose.yml
version: '3.8'

services:
  directus:
    image: directus/directus:latest
    ports:
      - 8055:8055
    volumes:
      - ./extensions:/directus/extensions
    depends_on:
      - db
    environment:
      DB_CLIENT: 'postgres'
      DB_HOST: 'db'
      DB_PORT: '5432'
      DB_DATABASE: 'postgres'
      DB_USER: 'postgres'
      DB_PASSWORD: 'postgres'
      STORAGE_LOCATIONS: 's3'
      STORAGE_S3_DRIVER: 's3'
      STORAGE_S3_KEY: '[IAMのアクセスキー]'
      STORAGE_S3_SECRET: '[IAMのSecret]'
      STORAGE_S3_REGION: '[バケットがあるリージョン]'
      STORAGE_S3_BUCKET: '[バケット名]'
      EMAIL_FROM: '[送信元のメールアドレス]'
      EMAIL_TRANSPORT: 'smtp'
      EMAIL_SMTP_HOST: 'smtp.google.com'
      EMAIL_SMTP_PORT: '456'
      EMAIL_SMTP_USER: '[送信元のメールアドレス]'
      EMAIL_SMTP_PASSWORD: '[Gmailのパスワード]'

  db:
    image: postgres:latest
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_USER: 'postgres'
      POSTGRES_DB: 'postgres'
    ports:
      - 5432:5432

volumes:
  postgres-data:
```

## 終わりに

他にも変更できるところが多々あるので、公式ドキュメントを熟読する必要がありそうです。英語の情報もあまりないよう（調べ方が悪い？）なので試行錯誤して解決していく必要がありそうです。

Gitなどに上げる場合はシークレットキーなどが公開されないよう、docker-composeの`env_file`を利用すれば良さそうです。

Directus、とても良いHeadless CMSなのでぜひ国内でも広まってほしいですね。
