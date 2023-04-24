---
title: "インフラ紹介"
date: 2022-5-19
slug: "servers"
description: "こんなページ見てて面白い？？"
keywords: ["about"]
draft: false
tags: ["about"]
math: false
toc: false
---

## インフラ紹介
sda1.net のインフラである私の自宅サーバーの紹介です。


### Lenovo ThinkServer TS140

#### 役割
 - ゲートウェイサーバー
 - Dockerホスト（freasearch.org、misskey.sda1.net、mi.ablaze.one、git.sda1.net、mtrx.sda1.net etc...）
 - httpサーバー（nexryai.online、sda1.netディレクトリインデックス）

#### 性能
CPU: Intel(R) Xeon(R) CPU E3-1225 v3 @ 3.20GHz<br>
RAM: 16GB (innodisk 8GB DDR3 ECC DIMM ×12)

#### OS
‌‌Ubuntu Server 22.04


### DELL PowerEdge R230

#### 役割‌‌
 - Podmanホスト（beta.misskey.sda1.net、個人用のKeycloakとoutlineサーバー、藍ちゃんbot etc...）

#### 性能‌‌
CPU: Intel(R) Celeron(R) CPU G3900 @ 2.80GHz<br>
RAM: 16GB (innodisk 8GB DDR4-2666 ECC DIMM ×2)

#### OS‌
‌‌Ubuntu Server 22.04


### Raspberry Pi 4 Model B
#### 役割
‌‌LDNAサーバー ‌‌、その他開発時とかに使う雑多用途

#### 性能
CPU: Broadcom BCM2711 (Cortex-A72 ×4)<br>
RAM: 4GB

#### OS
‌‌Ubuntu Server 22.04


### その他

スイッチングハブとかは金欠なのでTP-Linkの安いやつ使ってますが品質は十分で特に問題は起きていません。

お守り程度の雷サージを付けてます。

