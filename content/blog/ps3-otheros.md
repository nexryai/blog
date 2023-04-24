---
title: "OFW4.88のPS3を3.15までダウングレードしてOtherOSを復活させる　[2021年版]"
date: 2021-07-22
slug: "ps3-otheros"
description: "OFW4.88のPS3初期型が余ってたからLinuxを動かそうと思い色々試行錯誤した結果。"
keywords: ["firefox", "linux", "openSUSE"]
draft: false
tags: ["Tech"]
math: false
toc: false
---

OFW4.88のPS3初期型が余ってたからLinuxを動かそうと思い色々試行錯誤した結果。<br>

実はPS3にはOtherOSというLinuxやBSDを入れられる機能があったのだが3.21以降は廃止されてしまっている。OtherOS++という最新のCFWで復活させるプロジェクトもあるのだが制約も多く正規の機能を使いたかったのでこの方法で3.15までダウングレードした。

### 注意
 - 失敗すればブリックします。自己責任でお願いします
 - **導入の前に手持ちのPS3の初期FWが3.15以下であることを確認してください。初期FWのバージョン未満にダウングレードするとブリックして起動できなくなります。[このサイト](https://www.psdevwiki.com/ps3/SKU_Models)で確認できます。**

### 追記（2022/2/15）
参考にした情報が古かったせいで余計な手順を踏んでいる可能性があります。CFW4.88からQAフラグを有効にして直接3.15までダウングレードできる可能性もあります。ただし手元に改造用のPS3が無いため検証できません。これは理論上は可能ですが未検証ですので試すのは自己責任です。また4.87以降だとQAフラグからのダウングレードが機能しないとの報告もあります。もし結果が得られたら`contact[at]nexryai.online`まで報告お願いします。

### 導入

#### Step1
CFWを導入。やり方は[このページ](https://yyoossk.blogspot.com/2021/06/ps3488cfw.html)参照

#### Step2
3.55までダウングレードする。   
まず[このファイル](https://mega.co.nz/#!rYVhnAYI!vxg9QBSDXMDTW0HXnRli6h_4nUOlTq2e2ungYnqi-hE)をダウンロード。  
ダウンロードしたZIPの中に "3.55 Rogero Downgrader RSOD" というフォルダーがあるのでその中にあるPS3UPDAT.PUPをインストール。

##### リンク切れの場合
ミラーが[archive.org](https://archive.org/details/ps3-downgrade-cfws)にあります。

#### Step3
続いてQAフラグを弄る。[このpkgファイル](https://store.brewology.com/ahomebrew.php?brewid=219)をダウンロードしUSBの直下にコピー。<br>

このUSBをPS3に入れる。本来Package Managerからインストールするのだが何故か上手く行かない。  
そこでPS3のXMBでPackage Managerの上で△を押してオプションを開く。 "全てのパッケージをインストール" 的な英語のオプションがあるのでそれを実行。  
インストールが完了したらXMBからQA Toggleを実行する。画面が真っ黒になりビープが数回なって再びXMBが表示されれば成功。念の為PS3を再起動する。<br><br>
XMBでネットワーク設定のところまで行き<br><br>
L1 + L2 + L3 (左スティックを押す) + R1 + R2 + ↓ <br><br>
を同時に押す。  
Debug Settingなどが出てきたら上手く行ってる証拠。

##### リンク切れの場合
ミラーが[archive.org](https://archive.org/details/toggle_qa)にあります。

#### Step4
CFW3.55をインストールする。Step2でダウンロードしたZIPの "3.55.4 Rogero CFW 3.7" 内のPS3UPDAT.PUPをインストール。  
**続いて設定の中にある”Debug Settings”を選び、一覧にある "System Update Debug" をONにする。**

#### Step5
いよいよ最終工程のOFW3.15へのダウングレード。  
OFW3.15のPUPをセーフモードからインストールする。完全に動作させるには2回インストールしないといけないようです。

### Done
お疲れ様でした。あとは古き良きOtherOSを楽しむだけです。
