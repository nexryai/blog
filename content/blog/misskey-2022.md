---
title: "快適にMisskeyのお一人さまインスタンスを運営する"
date: 2022-12-22
slug: "misskey-advent-2022"
description: "Misskeyのインスタンスを安全で快適に運用して幸せになろう"
keywords: ["Misskey"]
draft: false
tags: ["Misskey"]
math: false
toc: true
---


## 概要
元々Twitterに生息していましたが今年の4月からMisskeyを始めたnexryai（ネクスライと読みます）です。厳密には前にも登録したことはありますが本格的に始めたのは今年です。<br><br>
今年は個人的に、健康面や人間関係で厄災が続き散々な一年でしたが、Misskeyはそんな私の心の支えでした。<br>
MisskeyはOSSとして開発されており、誰でもインスタンスを建てることができます。個人用のインスタンスをお一人さまインスタンスと呼んだりします。<br>
今回はお一人さまインスタンスをサクッと構築し、安全快適に運用する方法をご紹介します。

#### お知らせ
この記事は[Misskey Advent Calendar 2022](https://adventar.org/calendars/7354) 22日目の記事です。<br>
アドベントカレンダーに出るのは初めてなのでこんな内容でいいのか正直不安ですかご容赦ください。

#### 補足
この記事では私が実際に運用している方法で紹介しています。<br>
ご自身の環境で動かす場合は適切に読み替える必要がある場所が一部というかそれなりにあると思われるのでご注意ください。

#### 注意1
MisskeyインスタンスはMisskeyのみならずMastodonなどのあらゆるサーバーと連合します。<br>
サーバーが長時間ダウンしたり一日中不安定になるような環境では連合先に迷惑をかける（投稿を配送できないとジョブキューに負担がかかる）可能性があるのでなるべく安定して運用できるようにしてください。自宅回線が不安な方はVPSをオススメします。<br>
またインスタンスを閉鎖する場合はなるべく`410 gone`などを返すようにしてインスタンスが恒久的に閉鎖されたことを示すようにしてください。（ドメインを売りに出したとかで不可な場合を除く。502とか返されると閉鎖したのか落ちてるのか判断できないため。）


#### 注意2
ActivityPubの仕様上（？）データベースを消失してしまうと同じドメインでインスタンスを作ることはできなくなります。<br>
データベースのバックアップは取るようにしてください。

<br>

長過ぎる補足系はこれくらいにして本題に行きます。

## お一人さまインスタンスの利点
お一人さまインスタンスを建てる理由は人それぞれですが、サブ垢として使ったりするのもありです。
 - 絵文字とか好きにできる
 - 自己満足
 - サーバー運用のスキル向上
 - ~~独裁できる~~

## セットアップ
ここではdocker-composeとCaddyを使いサクッと構築します。もちろんCaddy以外のhttpサーバーでも可能なのでそこら辺は適当に読み替えてください。

### 前提条件
 - ubuntu 22.04 （他のディストリビューションでも可能。ただfirewalldを使用しているFedoraやopenSUSEだとネットワーク系が正しく動かないことがあり後述の対処法が必要）
 - 推奨: Btrfsでフォーマットされたドライブ（ext4などでも可能だがスナップショット機能を活用するとバックアップが飛躍的に便利になるので）
 - [このページの方法](https://caddyserver.com/docs/install)でCaddyをインストール済み（nginxなど別のhttpサーバーでも可能。）
 - 80、443ポートが開いており、外部にも開放されている。
 - `sudo apt install docker-compose`でDockerとdocker-composeをインストール済み
 - ドメインを持っていて、DNSのAレコードを自宅のIPへ向けている
 - Cloudflareなどはお好みで。証明書関係が面倒くさくなる上に個人用のインスタンスが攻撃されることはまずないので個人的には導入しなくてもいいと思う。ただこれを使わないと同じLANからアクセスするときにhostsを弄る必要が出てくる（詳細は後述）のでその辺をやりたくない人は使うといいかも。この記事では使わないことを前提に説明します。

### Step 1
Misskey用のディレクトリを作成します。この記事では`/var/servers/docker/misskey`を使用します。今後の解説でもこのパスを使用するので適時読み替えてください。<br>
Btrfsを使用すると後述するバックアップのところで非常に役立つのでおすすめです。<br>
#### btrfsの場合
```
sudo btrfs subvolume create /var/servers/docker
sudo mkdir  /var/servers/docker/misskey
```

#### その他ファイルシステムの場合
`sudo mkdir -p /var/servers/docker/misskey`

作成したディレクトリに移動し以下の2つのファイルを作成します。<br>

#### `docker-compose.yml`
```
version: "3"

services:
  web:
    image: misskey/misskey:latest
    restart: always
    links:
      - db
      - redis
    ports:
      - "127.0.0.1:3000:3000"
    networks:
      - misskey_db
      - external_network
    volumes:
      - ./files:/misskey/files
      - ./misskey.yaml:/misskey/.config/default.yml:ro

  redis:
    restart: always
    image: redis:4.0-alpine
    networks:
      - misskey_db
    volumes:
      - ./var/redis:/data

  db:
    restart: always
    image: postgres:14.6
    networks:
      - misskey_db
    environment:
      - POSTGRES_PASSWORD=[DBのパスワードを適当に設定]
      - POSTGRES_USER=misskey
      - POSTGRES_DB=misskey
    volumes:
      - ./db:/var/lib/postgresql/data

networks:
  misskey_db:
    internal: true
  external_network:
```

#### `misskey.yaml`
```
url: https://[インスタンスのドメイン]/
db:
  host: db
  port: 5432
  db: misskey
  user: misskey
  pass: [docker-compose.ymlで設定したDBのパスワード]
redis:
  host: redis
  port: 6379
id: 'aid'
port: 3000
```
##### 補足
同じローカルネットワークで別のインスタンスを実行している場合、`misskey.yaml`に以下を追記します。
```
allowedPrivateNetworks: [
  '127.0.0.1/32',
  '[LANの範囲 (192.168.0.0/24 など)]'
]
```
<small>~~これの存在を知らずGitHubでissueを建てたアホがここにいます~~ ←申し訳ありませんでした</small>

### Step 2
`docker-compose up -d`で起動します。<br><br>
FedoraやopenSUSEなどfirewalldを使用している環境の場合、ここでDBに繋がらないという趣旨のエラーが出ることがあります。その場合は`docker-compose.yml`を次のように書き換える必要があります。<br>（これは恐らくDockerのバグでfirewalldの設定と干渉しているのが原因と思われます。ただFedoraとかSUSEはDockerよりPodmanの方との相性を重視してるイメージがあるので何とも言えない。）
```
version: "3"

services:
  web:
    image: misskey/misskey:latest
    restart: always
    links:
      - db
      - redis
    ports:
      - "127.0.0.1:3000:3000"
    volumes:
      - ./files:/misskey/files
      - ./misskey.yaml:/misskey/.config/default.yml:ro

  redis:
    restart: always
    image: redis:4.0-alpine
    volumes:
      - ./var/redis:/data

  db:
    restart: always
    image: postgres:14.6
    environment:
      - POSTGRES_PASSWORD=[DBのパスワードを適当に設定]
      - POSTGRES_USER=misskey
      - POSTGRES_DB=misskey
    volumes:
      - ./db:/var/lib/postgresql/data
```

### Step 3
Caddyを設定します。`/etc/caddy/Caddyfile`を次のように書き込みます。
```
(logging) {
	log {
		output file /var/log/caddy/access.log {
			roll_size 1mb
			roll_keep 5
			roll_keep_for 720h
		}
	}
}

misskey.example.com (インスタンスのドメイン) {
        import logging
        encode zstd gzip
        reverse_proxy http://localhost:3000
        @login {
                path /api/signin*
                not remote_ip 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 224.0.0.0/4
        }
        # セキュリティ強化の為ログインをローカルIPのみからに限定する。ただしCloudflareを使う場合は使用不可
        # VPNなどがあり外からでもアクセスできるネットワークであれば有効にするとより安全になる。
        # respond @login "Blocked" 403
}

```

設定したら`sudo systemctl reloda caddy`で反映。<br>
Caddy君は極めて優秀なのでnginxみたいに長ったるい設定を書く必要はなく、証明性も勝手に取得して更新してくれます。リバースプロキシ関係のヘッダーもデフォルトで付けてくれる。

### Step 4 (CDNを使っていないかつ自宅のネットワークにインスタンスを立てている場合のみ)
CDNを使っていないかつ自宅のネットワークにインスタンスを立てている場合、このままだとインスタンスのアドレスにアクセスしてもルーターの管理画面に辿り着いてしまうのでhostsを弄る必要があります。<br>
<b>そうでない場合（VPSなどに建てている場合やCloudflareを使用している場合）はこのセクションは読み飛ばしてください。</b>
<br>
方法はいくつかあります。<br>

#### カスタムDNSサーバーを使う方法
AdGuard DNSやAdGuard Home、Next DNSなどに登録し、ルーターでそれを使うように設定します。DNSを書き換える機能が存在すると思うので、それを使用して自分のドメインへはサーバーのローカルIPを返すようにします。<br><br>
<small>Next DNSの場合</small><br>
<img src="/images/img1-3.png"><br>
<small>AdGuard Homeの場合</small><br>
<img src="/images/img1-4.png"><br><br>
この方法を個人的には推しています。ルーターで設定すればデバイスの設定を変更する必要はなく（ただしブラウザがDoHやカスタムDNSを使う設定になっている場合はルーターの設定を使うようにする必要あり）、設定次第ではサーバーが危険なドメインに通信するのを防いだりなどのおいしい副産物があります。

#### デバイスのHostsを設定する方法
これはPCでのみ有効でデバイスごとの設定が必要なので面倒かもしれません。OSのhostsファイルを開き以下の内容を追記します。（Linuxの場合`/etc/hosts`、~~その他OSは知らん~~）
```
# 192.168.1.13 bar.mydomain.org
[サーバーのローカルIP] [インスタンスのドメイン]
```

### Step 5
お疲れ様でした。お好みのブラウザでインスタンスのアドレスにアクセスしてみましょう。初期設定ページが出ていれば成功。<br>
あとは画面の指示に従いアカウントを作成します。

### Step 6
「コントロールパネル→リレー」からリレーを追加します。これをしないと他のサーバーを見つけられません。<br>
リレーの一覧は[このサイト](https://hisubway.online/blog/fediverse_relay/)にいい感じにまとまってるので参考にどうぞ<br><br>
<img src="/images/img1-5.png">

### Step 7
サーバーのディスクは基本的に有限です。リモートファイルのキャッシュが有効になってると長く運用するにつれてどんどんストレージが食いつぶされるのでこれを無効にします。<br>
「コントロールパネル→全般→リモートのファイルをキャッシュする」を無効にします。<br><br>
<img src="/images/img1-6.png">
### Done!
これで基本的な設定は終わりです。後はフォローやらミュートやらブロックをインポートしましょう。<br><br>
他には...
 - 絵文字の追加などはコントロールパネルから行えます。リモートの絵文字のインポートも可能です。
 - 「コントロールパネル→全般」からインスタンスの名称や説明、テーマや背景画像が設定できます。
 - お一人さまインスタンスではあまり必要ないと思ってるのでオブジェクトストレージやメール配信の設定はしていませんが、必要に応じて行ってください。

<img src="/images/img1-11.png">

## フォロー機能の活用
リレーに参加しててもリモートの投稿が飛んでこないことが多々ありますが、フォローしてるユーザーの投稿は同じインスタンスに居るかのように飛んできます。気になるユーザーを積極的にフォローすればTLも賑やかになります。

## 快適なTLを作る
お一人さまインスタンスでは最初は主にGTLを眺めることになるでしょう。しかしFediverseには色んなインスタンスが存在しており、中には適切にモデレーションされてないものやヘイトスピーチが活発に行われている危険なインスタンスも存在します。GTLはこれらの投稿も拾ってしまうので精神衛生上よろしくありません。  
そこで凍結機能とインスタンスブロックの出番です。冒頭でも述べた通りお一人さまインスタンスには基本的にあなたか親しい人しかいないので独裁できます。（もちろんインスタンスを登録開放にする場合は規約を作りそれに則った運営を行いましょう）

### 凍結機能
Misskeyではリモート・ローカル関係なくアカウントの凍結が可能です。リモートユーザーを凍結すると凍結を行ったインスタンス上ではそのユーザーは凍結された扱いになります。危険人物は躊躇なく凍結してTLの治安を保ちましょう。<br><br>
凍結したいユーザーのページを開き、メニューから凍結を選択することで凍結できます。
<br><br>
<img src="/images/img1-2.png">

### インスタンスブロック
インスタンス自体が過激な政治的主張やヘイトスピーチを行うためのものだったり、R18画像を投稿するためのものであることがあります。またそうでなくても適切にモデレートされていないインスタンスも存在します。そのようなインスタンスはインスタンスごとブロックしてしまいましょう。<br>
方法としては二通りあります。<br><br>１つ目は「コントロールパネル→インスタンスブロック」からブロックしたいインスタンスのドメインを入れる方法です。<br>２つ目は「コントロールパネル→連合」からブロックしたいインスタンスを選択し、「このインスタンスをブロック」を有効にする方法です。<br><br>
<img src="/images/img1-1.png">

<br>

`https://[インスタンスのドメイン]/about`を開き「連合タブ→状態」をブロック中にするとそのインスタンスがブロックしているインスタンスを見れるので参考になるかもしれません。

## バックアップ関係
サーバーにあるデータはいつ消失するか分かりません。誤操作で消した、地震でサーバーが壊れた、ディスクが突然死した、ミサイルが飛んできて家に命中したなどの理由で予期せずにデータベースをすっ飛ばす可能性があります。これでは恐ろしくて夜も眠れません。  
ここで私の書いた [bunker backup](https://git.sda1.net/nexryai/bunker) の出番です。こいつを使えばサーバーのデータを簡単にオブジェクトストレージにバックアップできます。しかも暗号化されるのでプライバシー的にもセキュリティ的にも安心です。ステマはこれくらいにしてバックアップ方法を書いていきます。

### バックアップ
#### Step 0
適当なオブジェクトストレージを用意します。VPSにMinioを入れるかCloudflare R2（10GBまで無料）をオススメします。

#### Step 1
[このページ](https://git.sda1.net/nexryai/bunker#%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)に従い秘密鍵を生成しサーバーに送りつけbunkerをインストールします。<br><br>
`config.yml`に以下の内容を設定します。<b>btrfsを使ってない場合はスナップショットを作成するコマンドをcpなど適当なコマンドに置き換えてください。</b>

#### ⚠注意
docker-composeで動いてるDBのバックアップのやり方として以下のようなやり方が紹介されることが多々あります。  
`docker-compose exec db pg_dump -Fc -U misskey -d misskey > misskey.dump`<br><br>
しかしこのやり方でやると<b>正常にダンプできないことがある（ダンプ自体は一見成功するがインポートできない）</b> ので注意してください。[非ASCIIのバイトが破損する](https://stackoverflow.com/a/63934857/16903706)みたいです。またこのやり方でも確実にバックアップできるとは限らないので必ずバックアップが復元できるかを確認してください。

<br>

```
server_name: misskey_server

# S3 Object storage config
s3_endpoint: https://<accountid>.r2.cloudflarestorage.com
s3_access_key: <access_key>
s3_secret_access_key: <access_key_secret>
s3_bucket_name: <bucket_name>

# backup config

# Number of backups to keep
save_backups: 2

backup_dirs:
  - /var/servers/backup

pre_backup_exec:
  # Btrfsのスナップショットを作成（btrfsを使ってない場合はcpなど適当なコマンドに置き換えてください）
  - btrfs subvolume snapshot /var/servers/docker /var/servers/backup
  # MisskeyのDBダンプ
  - docker-compose -f /var/servers/docker/misskey/docker-compose.yml exec -T db bash -c "pg_dump -Fc -U misskey -d misskey > /var/lib/postgresql/data/misskey_db.dump"
  - mv /var/servers/docker/misskey/db/misskey_db.dump /var/servers/backup/misskey/
  - rm -rf /var/servers/backup/misskey/db

encrypt_keys:
  - [先程生成した鍵のID]

post_backup_exec:
  - rm -rf /var/servers/backup
```

#### Step 2
`sudo bunker backup`を実行し試しにバックアップします。

#### Step 3
バックアップを定期的に行うようにします。この例ではアクセスが少なそうな毎日午前3時に実行します。生活リズムが壊滅的な方は日中にバックアップする設定の方がいいかも。<br><br>
`sudo crontab -e`を実行し以下の内容を追記します。
```
# m h  dom mon dow   command
0 3 * * * bunker backup
```

### リストア

#### Step 1
秘密鍵を生成したマシン（もしくはバックアップの暗号化に使った公開鍵の秘密鍵をインポートしたマシン）にもサーバーで行ったように`bunker`をインストールします。  
`config.yml`を設定したら、適当な一時ディレクトリに移動し`bunker restore`を実行します。（リストアと書いてますがカレントディレクトリにダウンロード、復号化されるだけです。）

#### Step 2
完了したら`var`のようなルートディレクトリ直下にありそうな名前のディレクトリができてるかと思います。中身を適当に漁ってMisskeyの`docker-compose.yml`があるディレクトリをサーバーへ転送します。

#### Step 3
転送したディレクトリの中身をMisskeyが動いていたディレクトリ（例では`/var/servers/docker/misskey`）へ移動します。<br><br>
続いてDBを復元します。<br>
まずDBを起こします
```
sudo docker-compose up -d db
```

続いてPostgreSQLのDBが動いているコンテナのIDを調べます。
```
docker ps
```

最後にダンプされたデータをコピーしインポートします。
```
docker cp misskey_db.dump [先程調べたpostgresのコンテナID]:/db.dump
docker-compose exec -T db pg_restore -U misskey -d misskey /db.dump
```

あとは`docker-compose up`すれば復元されます。

## botを導入する
優秀なMisskeyのbotである藍ちゃんを導入してみます。他にも [NullcatChan!](https://github.com/nullnyat/NullCatChan) などのフォークもあるので気になったら試してみるといいかもしれません。

### Step 1
インスタンス上でbotとして動かす用のアカウントを作成しプロフィールを設定します。また必ずプロフィールから「botとして設定」を有効にしましょう。お好みでcatなどを設定してください。<br><br>
<img src="/images/img1-7.png">
<br>
続いてアクセストークンを発行します。<br><br>
<img src="/images/img1-8.png"><br>
発行されたトークンを安全な場所にメモしておきましょう。

### Step 2
サーバーに入り適当なディレクトリを作成し移動します。

#### フォントのダウンロード
弊サーバーからダウンロードしていますが別にどこから落としても変わりません。
```
wget https://sda1.net/cdn/fonts/NotoSansCJKjp-Regular.ttf
mv NotoSansCJKjp-Regular.ttf font.ttf
```

#### config.json の作成
[このページ](https://github.com/syuilo/ai#docker%E3%81%A7%E5%8B%95%E3%81%8B%E3%81%99)の通りに`config.json`を作成します。

#### docker-compose.yml の設定
以下の内容で`docker-compose.yml`を作成します。ビルド済みのイメージが見つからなかったので自分がアップしたのを使っていますが自分が見つけられてないだけで公式のがどっかにあるかもです。
```
version: '3'
services:
  app:
    image: nexryai/misskey-ai:latest
    volumes:
      - './config.json:/ai/config.json:ro'
      - './font.ttf:/ai/font.ttf:ro'
      - './data:/ai/data'
    restart: always
```

### Step 3
`docker-compose up -d`で実行します。<br>
ログに以下のように`[AiOS]: Ai am now running!`と出ればok<br><br>
<img src="/images/img1-9.png"><br>

### 生存確認
`@ai ping`と投稿し`PONG!`と返してくれれば正常に動いている証拠です。<br><br>
<img src="/images/img1-10.png">
<br><br>
どのような単語に反応してくれるかなどは[こちら](https://github.com/syuilo/ai/blob/master/torisetu.md)のページが参考になるでしょう。

## docker-composeをsystemdで制御する
ここでは説明を簡略化するために`docker-compose up -d`を使ってバックグラウンドプロセスを動かしていますが、私はバックグラウンドプロセスはなるべくsystemdで制御したい人間なのでdocker-composeをsystemdで制御しています。やり方に関しては`docker-compose systemdで制御`などと調べれば出てくるのでここでは省略します。

## まとめ
以上が自分なりのインスタンスを快適に運営する方法でした。何かコメント、疑問、質問、苦情、言いがかり等あれば`nexryai@misskey.sda1.net`までお気軽にどうぞ。<br><br>
最後まで読んでいただきありがとうございました。それでは良いMisskeyライフを。
