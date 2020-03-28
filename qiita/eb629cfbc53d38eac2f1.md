---
title: standaloneのsolrのインデックスのお手軽バックアップ
tags: Solr
author: n_slender
slide: false
---
スタンダアローンのsolrにも、backup APIがあるので、それを定期的に実行すれば簡易的なバックアップがとれます。ちょっと必要になったので振り返れるようにメモ。

numberToKeepのパラメタを渡すことで世代管理もできるので、一週間分を残したいなら

`curl http://localhost:8983/solr/collectionName/replication?command=backup&numberToKeep=7`

を、cronで毎日実行とすれば十分です。

https://cwiki.apache.org/confluence/display/solr/Making+and+Restoring+Backups

レストアの仕方は以前に書いたのでこちら参考に。
http://qiita.com/n_slender/items/93216bae005ac96b75c4

