---
title: oracleからjdkなどをcliで入手するためのやり方
tags: Java cli memo
author: n_slender
slide: false
---
oracleからjdkなどをcliで入手するためのコマンド。
忘れたころに必要になるのでメモ。（調べたら同様の記事がありました... ）

`wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie"  http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.rpm`

webサイトから入手するときはこの辺から
http://www.oracle.com/technetwork/java/javase/downloads/index.html

# 参考サイト

* http://www.task-notes.com/entry/20150602/1433214000
* [Linuxでjdkをwgetする方法](http://qiita.com/hajimeni/items/67d9e9b0d169bf68d1c9)

