---
title: "misskey.sda1.net 透明性レポート"
date: 2022-5-19
slug: "misskey-tr"
description: "Transparency Report"
draft: false
math: false
toc: false
---
## 当インスタンスでブロックしているインスタンス一覧
GTLをより健全で安全なものにするために、misskey.sda1.net では一部のインスタンスをブロックしています。  
ブロックされたインスタンスのドメインは単にMisskey上でブロックされるだけでなく、WAFやファイアウォールによりアクセスが制限されサーバー側からの名前解決も行われなくなります。（ブロックしているインスタンスのURLを貼り付けてもプレビューされないのはこのため）

### 陰謀論や過激思想の放置、連合サーバーへの攻撃、スパム拡散、限度を超えたR18画像の許容等が理由でのブロック
```
mstdn.love
freespeech.gaac2.com
poa.st
mk.f72u.net
mastodon.cf
baraag.net
mstdn.jp
m.tkw.fm
libera.tokyo
songbird.cloud
union.place
socialism.social
kolektiva.social
mastodon.lol
gnusocial.jp
freespeechextremist.com
kiwifarms.cc
botsin.space
baraag.net
shitposter.club
gameliberty.club
spinster.xyz
cliterati.club
develop.gab.com
etiketi.de
gab.com
neenster.org
glindr.org
social.illegalpornography.com
social.quodverum.com
switter.at
artalley.porn
mi.tkw.fm
activitypub-troll.cf
misskey-forkbomb.cf
repl.co
freespeech.group
cum.salon
freethought.online
freecumextremist.com
tw.9tail.net
asbestos.cafe
```

### Twitterから無断転載を行ってるためブロックしているインスタンス
```
bird.makeup
gateway.occm.cc
beta.mstdn.cf
beta.birdsite.live
twtr.plus
twtr.disqordia.space
bird.samtherapy.net
twtr.wappie.land
birdsite.slashdev.space
birdsite.wilde.cloud
bird.froth.zone
news.twtr.plus
tweet.pasture.moe
twtr.carnivore.social
bird.im-in.space
twtr.vrij.social
twotter.waifuism.life
birdsite.nytpu.com
birb.elfenban.de
bird.evilcyberhacker.net
birdbots.leptonics.com
twitter.grants.cafe
bird.lgbt.pm
birdsite.thorlaksson.com
bridge.birb.space
birdsite.platypush.tech
politbird.twexodus.top
birdsite.blazelight.dev
tweets.icu
bird.istheguy.com
twitter.doesnotexist.club
test.misskey.cf
birdsite.lakedrops.com
relay.scottsmall.org
twttr.yukineko.me
lol.lamp.wtf
tw.9tail.net
```

## 連合制限について
以下のドメインでホストされるインスタンスとは連合されず、以下のドメインで運営されているサイトのURLを貼り付けてもプレビューは表示されません。

 - NextDNSやその他フィルターにて脅威、トラッカー、広告、アダルトサイト、犯罪関係、過激と指定されているドメイン
 - ホワイトリストにない無償ドメイン（.cf、.tkなど）

## IP制限について
インスタンスを保護し、正当な利用者が快適に利用可能な環境にするために、以下に該当するアクセスは制限を受ける可能性があります。  
特に児童ポルノに関してはアップロードされると運営者にも被害が出てしまいます。

### 防弾ホストのASN
対象: 連合、タイムラインの閲覧など全てのアクセスが不可能  
説明: DDoS攻撃の防止の為、基本的に防弾ホストからはアクセスできません。

### AbuseIPDBによって危険とマークされているIPアドレス
対象: 連合、タイムラインの閲覧など全てのアクセスが不可能   
説明: 公開プロキシやVPNを使用している場合でも引っかかるかもしれません。

### 過去に運営者（nexryai）のサーバーに攻撃を行ったIPアドレス
対象: 連合、タイムラインの閲覧など全てのアクセスが不可能

### Tor
対象: 連合が不可能  
説明: AbuseIPDBでIPがマークされている場合はアクセスすらできない可能性があります。
<br>

**検閲されている国に居るなどの理由からTorを使用する必要がある方へ**  
管理人に連絡してもらえれば個別に対応します。

### 運営に損害を与える目的（ブロックしているインスタンスの不必要な晒し上げなど）を持つクローラーのIPアドレス
対象: 連合、タイムラインの閲覧など全てのアクセスが不可能
