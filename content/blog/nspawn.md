---
title: "Dockerやハイパーバイザーの変わりにsystemd-nspawnを使う"
date: 2022-03-29T09:42:16+09:00
draft: false
---


ハイパーバイザーの変わりにDockerを使おうと思ったがハイパーバイザー的な使い方はできないと知り代替を探しているとSystemd-nspawnというものに出会った。オーバーヘッドがほぼないハイパーバイザー的な感じで使えるchrootの強化版らしい。

<br>

### Dockerとの比較

- Dockerは基本的に一つのアプリを動かす目的で使われるのでハイパーバイザー的な使い方はできない（コンテナ内からsystemctlがそもそも使えない）
- systemd nspawnは仮想マシンのような感覚で使える。
- ただしsystemd nspawnの方がディスクを多く使う上コンテナは配布されていないので自力で構築する必要がある


### ハイパーバイザーとの比較
- systemd nspawnの方がオーバーヘッドがかなり少ない（仮想化技術を使っているのではなくchrootなので）
- ただしsystemd nspawnはゲストとして使えるOSが限られている（ubuntuなどメジャーなLinuxディストリビューションなら動くがWindows serverやTrueNASなどは動かない）


## 実際に使ってみる

### systemd nspawnのインストール

標準で入っているらしいがopenSUSEだと入っていなかった。ディストリビューションによってパッケージ名が違うのでそこは各自調べてください（cnfやcommnad-not-foundコマンドを使うと便利）  
続いて以下を実行 <br>

```
sudo systemctl enable machinectl.target
sudo systemctl start machinectl.target
```

<br>
 
### ゲストOSを構築

今回はゲストとしてubuntuかdebianを採用。まずdebootstrapをインストールする。

続いて

`sudo mkdir -p /var/lib/machines/server1`

 

#### ubuntuの場合
`sudo debootstrap --arch=amd64 focal /var/lib/machines/server1 http://archive.ubuntu.com/ubuntu/`

#### debianの場合
`sudo debootstrap --arch=amd64 bullseye /var/lib/machines/smb https://ftp.riken.jp/Linux/debian/debian/`

<br>
 
### 初期設定

`sudo systemd-nspawn -D /var/lib/machines/server1` でコンテナに入りpasswdでパスワードを設定する。ここまでくればもう完成したも同然。

<br>

#### 設定ファイル記述

`sudo machinectl enable server1`

続いて `/etc/systemd/nspawn/server1.nspawn` を作成し以下の内容を書き込む

```
[Network]
MACVLAN=em1
```

ホストのディレクトリをコンテナにマウントしたい場合は以下も追記

```
[Files]
Bind=[マウントしたいディレクトリのパス]:[コンテナ内でのマウント先のパス]

[Exec]
PrivateUsers=no
```

<br>

#### コンテナ起動
`sudo machinectl start server1`  
`sudo machinectl login server1`

<br>

#### ネットワークを開通させる
`ip a` でネットワークインターフェースの名前を調べ `ip l set [インターフェース名] up` で起こす。ipが何故か降ってこないので `dhclient -v` でipを取得


※抜けるにはctrlキーを押しながら" ] "キーを3回押す

※macvlanオプションを付けているのはホストと同じip空間に接続したいから

### 各種セットアップ

#### 不足しているaptのリポジトリを追加

###### ubuntu
```
apt install software-properties-common
apt-add-repository universe
```

###### debian
`apt install sudo`


#### ユーザー追加

```
adduser user
gpasswd -a user sudo
```
 
#### ネットワークの設定の永続化とip固定

1. `/etc/systemd/network/[インターフェース名].network` を作成し以下を追記 (アドレス等は適当に読み替えて)
```
[Match]
Name=[インターフェース名]

[Network]
Address=192.168.0.100/24
Gateway=192.168.0.1
DNS=192.168.0.1
```
 

2. systemd-networkdの有効化
```
ip l set [インターフェース名] down
systemctl enable systemd-networkd
systemctl start systemd-networkd
```

<br>

#### ファイアウォールの有効化 （重要）
ファイアウォールの有効化は絶対に必須。ファイアウォールなしでサーバーを公開するのは何も持たずに普段着で戦場に飛び出すのと同じ。

```
sudo apt install ufw
sudo ufw default deny
```

このままだとsshが使えないのでsshを使いたい場合
```
sudo ufw allow from [192.168.0.0/24 などの許可するIPの範囲] to any port 22
```

最後に有効化
```
sudo ufw enable
```

<br>

#### sshの有効化

`sudo apt install openssh-server`

 
## おまけ（備忘録）
### smbサーバーの構築
```
sudo apt install samba
sudo chown <ユーザー名>:<グループ名> /var/storage
sudo chmod 770 /var/storage
```

`/etc/samba/smb.conf` に追記

```
[storage] # 共有で表示される共有名
path = /var/storage
writeable = yes
guest ok = no
create mode = 0775
directory mode = 0775
valid users = @<グループ名>
```

後は `sudo systemctl restart smbd`でsamba起動


<br>

## まとめ

普通の仮想マシンのような使い勝手なので個人的にはDockerより使いやすい。ディレクトリをコピーすればそのままバックアップできるので移行も簡単。リソース消費量が少なくラズパイのような非力な環境でも動く。 systemd最高！

