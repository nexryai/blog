---
title: "tpmを使ってルートファイルシステムと外付けドライブのLUKSを自動アンロックする"
date: 2023-06-22
draft: false
slug: 5fbde8b3-dc1e-4376-ad72-5b017269d325
keywords: ["LUKS", "Linux", "tpm", "cryptsetup", "security"]
description: "BitLocker的なのをLinuxでもやりたかった"
---

### 前提条件
 - ルートファイルシステムがLUKSで暗号化されたUbuntu 22.04
 - tpm2.0搭載PC

### やり方
tpmでルートファイルシステムを復号化し、ルートファイルシステムに格納した鍵で外付けドライブの復号化を行います。


#### Step1. ルートファイルシステムを自動復号化する
[このサイト](https://zenn.dev/walkmana_25/articles/ubuntu2204-tpm2)のやり方が一番わかりやすいかも

```
sudo apt install clevis-tpm2 clevis-luks clevis-initramfs -y
sudo clevis luks bind -d /dev/sda2 tpm2 '{"hash":"sha256","key":"rsa"}'
sudo update-initramfs -u
```

#### Step2. 外付けドライブをフォーマット、暗号化する

```
# ディスクにパーティションが存在しない場合はここで作る
sudo fdisk /dev/sdb

# 復号化鍵を作成（必ずバックアップするべきかも）
sudo dd bs=2048 count=4 if=/dev/urandom of=/root/sdb.key

# パーティションを暗号化
sudo cryptsetup luksFormat /dev/sdb1 /root/sdb.key
sudo cryptsetup open /dev/sdb1 cryptdata_sdb1  --key-file /root/sdb.key

# ファイルシステム作成
sudo mkfs -t ext4 /dev/mapper/cryptdata_sdb1
```

#### Step3. 外付けドライブの自動復号化を設定する

まず暗号化されたパーティションのUUIDを取得

```
# ディスクのUUID取得
sudo cryptsetup luksDump /dev/sdb1 | grep "UUID"
```

続いてそれぞれ次の設定ファイルに適切な設定を追記する

##### `/etc/crypttab`
```
cryptdata_sdb1 UUID=[取得したUUID] /root/sdb.key
```

##### `/etc/fstab`
```
/dev/mapper/cryptdata_sdb1 /var/nas/storage2 ext4 defaults 0 2
```

#### Step4. 再起動して確認
実際に再起動して復号化されるか確認する。LUKSのパスワードを聞かれるプロンプトが出るがしばらくすると進むはずなので気長に待つべし。

#### 最後に（重要）
**必ず暗号化に使ったキーファイル（ここでは`/root/sdb.key`）を安全な場所にバックアップしてください。**  
これがないと万一キーファイルが飛んだときに復号化できなくなります。
