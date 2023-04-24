---
title: "GStreamerとFFmpeg、libavcodecの関係性"
date: 2023-03-12
slug: "gstreamer-and-ffmpeg"
description: "ffmpegとgstreamerの関係性"
keywords: ["ffmpeg", "gstreamer", "libavcodec", "libvlc", "vlc"]
draft: false
math: false
toc: false
---
気になって調べてみたので備忘録として残します。間違ってたらお気軽に指摘してください。

## libavcodec・GStreamer
別物ですが、この2つは組み合わせて使われることが多々あります。

### FFmpeg + libavcodec
FFmpegは広く知られている動画や音声をエンコード、デコードするためのソフトです。様々な場面で使われています。  
libavcodecはFFmpegプロジェクトの一部であり、名前の通りエンコードやデコードを行うライブラリです。FFmpegはlibavcodecを使用してエンコードやデコードを行っています。

### GStreamer
あまりこちらは知名度は高くありませんが、FFmpeg同様様々な場面で使われています。GStreamerはプラグインによって拡張可能なマルチメディアフレームワークです。  
GStreamerは後述するx265などを使ったプラグインだけでなく、FFmpegの基礎となっているlibavcodecもプラグインとして使用できます。  
GnomeやKDEなどのメディアプレーヤーにもGStreamerは使われています。

## GStreamerやlibavcodecに使われているライブラリ
GStreamerやlibavcodecもまた、特定のコーデックのエンコードやデコードのために libx264・libde265・libmpeg2・libvpx・libaom などのライブラリを使用しています。  
これらのライブラリは特定のコーデックに特化したライブラリです。

## その他
### libvlcとは
libvlcはVLCが使っているライブラリです。libavcodecに依存しています。VLCがFFmpegを使用していると言われるのはこのためです。

### ハードウェアアクセラレーション
`gstreamer-vaapi`などをインストールすることでGPUによるハードウェアアクセラレーションを使えます
