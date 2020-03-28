---
title: play + heapstats
tags: Java PlayFramework heapstats monitoring 監視
author: n_slender
slide: false
---
Play Java のアプリを監視するために、HeapStatsを導入した際のメモ。

## HeapStatsのインストール

playがあるサーバにエージェントをインストール。
詳しくは公式サイトを参照。
http://icedtea.classpath.org/wiki/HeapStats/jp

収集データのローテーションや出力先の設定例も公式サイトにあるので、実際使う場合はそれも設定すること。

## 起動スクリプトでagentを指定

`play dist`で作成された起動スクリプトの実行時に、bin/app_nameにagentlibの値を指定する。

```
./bin app_name -J-agentlib:heapstats
```

## まとめ

HeapStatsの良いところは、データの収集と解析が切り分けられているところ。
本番環境が顧客のところにあっても、収集データを連携してもらえれば、手元で解析できる。
http://www.slideshare.net/chi9rin/heap-stats-35885661

Javaアプリの監視手段の候補としてもっておくと、良さそう。

