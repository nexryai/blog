---
title: "PostgreSQL15に非同期レプリケーションを設定してDBサーバーをリアルタイムでバックアップ"
date: 2023-02-25
slug: "postgresql-async-replica.md"
description: "PostgreSQL15に非同期レプリケーションを設定してDBサーバーをリアルタイムでバックアップ"
keywords: ["postgresql", "repkication", "database"]
tags: ["tech"]
draft: false
math: false
toc: false
---


## 概要
PostgreSQL15に非同期レプリケーションを設定してDBサーバーをリアルタイムでバックアップする方法です。
 - 非同期なので速度には影響しない
 - 主にDBを遠隔地にあるサーバーにリアルタイムバックアップしたいときなどに使う

## やり方
`pg_hba.conf`とかは普通のDBのレプリケーションと基本は同じですが以下の点が違う

### `/etc/postgresql/15/main/postgresql.conf`
```
# こうしないとDBが巨大な場合、転送中にWALの期限が切れてbasebackupコマンドが通らなくなる
archive_mode = on

# 同期しない設定
synchronous_commit = off
synchronous_standby_names = ''
```

後は普通のレプリケーションと同じように`pg_basebackup`を使ってDBをコピーして起動させるだけ。レプリカ側の`hot_standby`とかはお好みで。
