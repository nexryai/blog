---
title: "AVIF画像をFedoraで扱う"
date: 2023-03-13
slug: "avif-on-fedora"
description: "題名通り"
keywords: ["avif", "fedora", "linux", "image"]
draft: false
tags: ["tech"]
math: false
toc: false
---

私は個人的にAVIFという画像フォーマットを推しています。しかし何故かfedoraのデフォルトだと扱えないようです。

### 対処報
libheif を入れる
```
sudo dnf install libheif
```
