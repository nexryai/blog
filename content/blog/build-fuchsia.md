---
title: "Google独自のOS 「Fuchsia」 をビルドしよう"
date: 2021-09-01
slug: "build-fuchsia"
description: "Google独自の新しいOSであるFuchsiaをビルドする方法"
keywords: ["google", "fuchsia", "OS"]
draft: false
tags: ["Tech"]
math: false
toc: true
---

Google独自の新しいOSであるFuchsiaをビルドする方法です。FuchsiaはLinuxカーネルの代替ともなるOSで、組み込みに使いにくいGPLでなくBSDやMIT、Apache2ライセンスで頒布されています。<br><br>
ビルドは非常に簡単です。AndroidやChromeOSのビルドより圧倒的に楽です。

### 必要なもの
 - x86_64のCPUで動いてるLinux環境（どのディストロでも可能）
 
 Macでも可能らしいですが持ってないので解説できません。Windowsでは不可能です。（そもそもWindowsはこんなことやるOSではないけど...）  
この記事読んでビルドしようとするような方なら実機Linux or Mac環境くらいあると思われるのでそれを使ってください。

 - 時間と翻訳ソフトを使う能力
 
ビルドにはかなり時間がかかります。またFuchsiaは発展途上なので日本語での情報はほとんどありません。

### いざビルド

#### Step1
ビルドに必要なパッケージを入れます
 - curl
 - unzip
 - git
 - go
 - Build Essential  (OpenSUSEの場合は `sudo zypper install --type pattern devel_basis` で入る)
 - kvm

#### Step2
Fuchsiaのソースコードを持ってきます
```
curl -s "https://fuchsia.googlesource.com/fuchsia/+/HEAD/scripts/bootstrap?format=TEXT" | base64 --decode | bash
```

#### Step3
ビルドの準備
```
export PATH="[先程のコマンドを実行した場所]/.jiri_root/bin:$PATH"
cd fuchsia
```
ビルドに必要なツールのパスを通しFuchsiaのソースコードが入ったディレクトリへ移ります。`export PATH="[先程のコマンドを実行した場所]/.jiri_root/bin:$PATH"`は.bashrcに追加しときましょう。

#### Step4
ターゲットを決めビルドします。  
Fuchsiaはまだ開発中のOSなのでPixel bookなどの限られたハードでしか動きません。今回はエミュレータで動かそうと思います。（残念ながらラズパイでは動かないそうです...）<br><br>
ビルド自体はコマンド一行です。CPUは100%に張り付くので不要なソフトは終了しましょう。時間がかかるのでゲームでもして気長に待ちましょう。香川県民の方は知りません（）
```
fx set workstation.x64 --release
fx build
```

#### Step5
待ちに待った起動の時間です。と言いたいところですが初回は少し準備が必要です。   
KVMの有効化をします。  
```
sudo usermod -a -G kvm ${USER}
```
このコマンドを実行したら再ログインします。

### 起動！！！
```
fx vdl start --software-gpu 
```
上手く行けばこんな感じで起動します。

<img src="/images/20210901210853.png">
