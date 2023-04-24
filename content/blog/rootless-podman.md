---
title: "RootlessなPodmanでcomposeする"
date: 2023-03-21
slug: "rootless-podman-compose"
description: "RootlessなPodmanでcomposeする方法です。"
keywords: ["podman", "docker", "server", "compose"]
draft: false
tags: ["tech"]
math: false
toc: true
---

### 概要
RootlessなPodmanでcomposeする方法です。

#### Why podman
 - デーモンレス
 - ルートレスにしやすい
 - RHEL系のOSとの高い親和性

### やり方
```
# 基本パッケージを入れる
sudo dnf -y install podman podman-docker

# compose入れる（pip使うのはおすすめしない）
curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose
chmod 755 /usr/bin/docker-compose

# Podmanをユーザー権限で動かす
systemctl --user enable --now podman.socket

# ユーザー権限で動いてるPodmanを使うように指定
echo "export DOCKER_HOST="unix:$XDG_RUNTIME_DIR/podman/podman.sock"" >> .bash_profile
```

あとは`docker-compose up`で動くようになる。既存のボリュームがあると権限関係で詰むので注意
