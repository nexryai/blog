---
title: "最も簡単で安全にSSH公開鍵認証を設定する方法 【令和最新永久保存版 】"
date: 2022-10-13
slug: "easy-ssh"
description: "A brief guide to setup KaTeX"
keywords: ["ssh", "gpg", "pubkey", "auth"]
draft: false
tags: ["tech"]
math: true
toc: false
---



SSHの公開鍵認証って面倒だし何となく怖いと思っている方もそうでない方にも、今回はSSHの公開鍵認証を設定する最も簡単で安全な方法を手短に紹介します。

## 1. 鍵を作る

以下のコマンドを実行。
```
ssh-keygen -t ed25519
```
パスやらパスワードを聞かれるので適当に答える。 某大国のアホが戦争始めたせいで悪化した 国際情勢も相まってサイバー攻撃が増加しているのでアルゴリズムには安全で高速なed25519を使用。
## 2. サーバーに送りつける

以下のコマンドを実行。
`ssh-copy-id -i [先程作成した鍵のパス + ".pub"] [サーバーのユーザー名]@[サーバーのアドレス]`  

[先ほど作成した鍵のパス + ".pub"]は何も指定していなければ`~/.ssh/id_ed25519.pub`になります。

## 3. 公開鍵認証を試す

普通にSSH接続をしてログインパスワードを聞かれず鍵のパスワードを聞かれる画面が出れば成功。また鍵のパスワードは一回入力すればログアウトしない限り入力せずにSSH接続できるはずなのでそうなるか試す。

### 上手く行かない場合
sshでリモートマシンにアクセスし、`/etc/ssh/sshd_config`を開く。  
`PubkeyAuthentication`が`yes`になっているか確認する。修正したら`sudo systemctl restart sshd`でsshを再起動。


## 4. SSHを公開鍵認証限定にする

SSHでリモートマシンにアクセスし、お好みのエディターで`/etc/ssh/sshd_config`を編集。
`PasswordAuthentication` という項目を探す。環境によってはコメントアウトされているので注意。
見つけたらその行を次のように書き換える。

```
PasswordAuthentication no
```

また次の項目も同じように設定することを推奨

```
# rootでのsshログインを拒否
PermitRootLogin no
```

最後に`sudo systemctl restart sshd`を実行し設定を反映。

