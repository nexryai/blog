---
title: "ActivityPubのリレーを簡単に建てる"
date: 2022-12-28
slug: "activitypub-relay"
description: "ActivityPubリレーの建て方"
keywords: ["fediverse", "misskey", "selfhost"]
draft: false
tags: ["tech"]
math: false
toc: true
---

## 概要
Misskeyインスタンスはそれなりにあるがリレーはあんまりない気がするのでサクッと建てる方法を紹介します。  
色々あるみたいですが今回はRustで書かれている[AodeRelayのフォーク](https://github.com/Interstellar-Relay-Community/aode-relay)を使います。

### 宣伝
この方法で実際に[relay.sda1.net](https://relay.sda1.net/)を構築しました。  
自由に追加できるのでインスタンスをお持ちの方は`https://relay.sda1.net/inbox`を追加することをご検討ください。（宣伝）

## 前提条件
 - Linux
 - docker-composeが使えるかdockerにcomposeプラグインが入っている状態
 - 安定したネットワーク環境

## Docker composeの設定
いい感じのイメージがなかったので私が作りました。

### `docker-compose.yml`

```
version: '3'
services:
  relay:
    image: nexryai/relay:latest
    volumes:
      - ./data:/mnt/
    ports:
      - 8080:8080
    environment:
      - ADDR=0.0.0.0
      - SLED_PATH=/mnt/sled/db-0.34
      - HOSTNAME=[リレーのドメイン]
      - PUBLISH_BLOCKS=false
      - RESTRICTED_MODE=false
    restart: always

```

あとはいい感じにリバースプロキシを設定してください。
