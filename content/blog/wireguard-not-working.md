---
title: "Wireguardを有効化するとインターネットに繋がらなくなるときの対処法"
date: 2022-10-13
slug: "wireguard-not-working"
description: "題名通り"
keywords: ["wireguard", "vpn"]
draft: false
tags: ["tech"]
math: false
toc: false
---


Wireguardは素晴らしいソフトウェアですが一つだけ初見殺しな仕様があります。

## 前提条件

VPN経由でインターネットに接続したいのではなくあくまでもサーバー間の通信としてVPNを使用したい。

```
eth0 (インターネット )          eth0 (インターネット)
 ↕                              ↕
サーバーA ← wg0 (vpn) wg0 → サーバーB
```
## 症状

 - Wireguardを有効にするとインターネットに繋がらなくなる
 - 無効化すると戻る
 - ip routeでデフォルトルートを調べてもちゃんとeth0など本来のインターフェイスが設定されている

## 対処法

まずip route get 1.1.1.1を実行してみる。すると驚くべきことにip routeではeth0がデフォルトになっているのにwg0経由になっている。

```
suser@localhost:~> ip route
default via 192.168.0.1 dev eth0 proto dhcp [←ちゃんとeth0がデフォルトルートなのに]
10.0.0.0/24 dev wg0 proto kernel scope link src 10.0.0.2 
192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.200 

suser@localhost:~> ip route get 1.1.1.1
1.1.1.1 dev wg0 table 51820 src 10.0.0.2 uid 1000  [←何故かwg0経由になっている]
    cache 
```

実はこれWireguardの仕様らしい。こんなん気づく訳ないやろという感じですが仕様なので仕方ありません。
しかし対処法は簡単なのでご安心ください。
`/etc/wireguard/wg0.conf` に `Table = off`を追加するだけです。

`sudo nano /etc/wireguard/wg0.conf`

```
[Interface]
Address = 10.0.0.2/24
ListenPort = ポート
PrivateKey = 秘密鍵
Table = off [←これを追加する]


[Peer]
PublicKey = 公開鍵
PresharedKey = 事前共有鍵（任意）
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = サーバーのIP:ポート
PersistentKeepAlive = 30
```
後は`sudo systemctl restart wg-quick@wg0`なり何なりで再起動して変更を適用しましょう。

