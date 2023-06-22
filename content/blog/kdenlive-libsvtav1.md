---
title: "libsvtav1を使ったらffmpegでのAV1エンコードが超高速になった話"
date: 2023-06-22
draft: false
slug: 34f1d47b-74df-44b5-b4f5-e548486f2eab
keywords: ["ffmpeg", "av1", "avt-av1", "libsvtav1", "codec", "kdenlive"]
description: "超お得"
---

### AV1のエンコード遅すぎ問題
AV1、法的関係の面倒な面がなくサイズも小さくなるという素晴らしいコーデックですがエンコードが非現実的なまで遅いという致命的な欠点があります。  
これを解決するべく試行錯誤した結果たどり着いたのがlibsvtav1です。

### 使い方

#### インストール
`--enable-libsvtav1`オプションと共にビルドされたffmpegが必要です。Fedoraなら[このページ](https://computingforgeeks.com/how-to-install-ffmpeg-on-fedora/)で紹介されているような一般的な方法でインストールすれば有効になっています。  
Archの公式リポジトリでも有効でした。Windowsの場合は[この辺](https://ffmpeg.org/download.html#build-windows)のバイナリを入れるといいかも。

#### 使ってみる
crfはお好みで32とかに上げるとサイズ削減できるかも  
`ffmpeg -i test.mp4 -c:v libsvtav1 -crf 30 -b:v 0 -c:a libopus -b:a 256k -ac 2 -ar 48000 test.webm`

### Kdenliveで使う
Linux環境なら複雑な設定無しでKdenliveでも使えます。適当なパッケージマネージャでkdenliveと`--enable-libsvtav1`なffmpegを入れてあることが前提条件です。

#### プロファイル追加
レンダリング設定を開きます  

<img src="https://s3.sda1.net/misskey/contents/bf560d70-6843-4d90-97fa-2845efbcb1c3.png"></img>

新規プロファイルを作成し手動編集します。  

<img src="https://s3.sda1.net/misskey/contents/fb337078-911e-4041-88c0-6abd359d5684.png"></img>

以下の設定を貼り付け

```
ab=%audiobitrate+'k' acodec=libopus ar=48000 channels=2 crf=32 f=webm strict=experimental vcodec=libsvtav1
```

あとは適当な名前で保存してレンダリングします。
