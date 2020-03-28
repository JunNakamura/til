---
title: Jenkins+Slackで超簡易版ヘルスチェック
tags: Jenkins Slack healthcheck
author: n_slender
slide: false
---
自前のサーバにアプリを展開しているとき、死活監視を簡単にしようと思ってつくった際のメモ。

Site Monitor pluginとSlack notification pluginを使う。


## slack連携の設定

システムの設定で、グローバル設定ができる。
チームのサブドメインとトークンを貼りつけて、通知先のチャンネルをいれるだけ。

## ヘルスチェック用のジョブ作成

### slackの通知設定

ジョブの設定画面にslackの通知設定項目がある。死んだときと復活したときに通知してもらいたいので下記のようにする。

![image](https://qiita-image-store.s3.amazonaws.com/0/9880/aa96605b-b45a-4299-1dd7-678ce59fec5d.png)

### トリガの設定

アプリのビルドが成功した後と、定期的に実行してもらいたいので、つぎの二つにチェックをいれる。

* 他プロジェクトの後にビルド（job名を入力）
* 定期的に実行



### ビルド後の処理

サイトモニターのURLとSlack通知を追加して完了

![image](https://qiita-image-store.s3.amazonaws.com/0/9880/f3df1441-194f-c09c-e023-c44de498cabe.png)

### ローテーション

ビルドとその成果物をいつまでも残す必要がないので、適当な間隔でローテーションさせる。

## 参考サイト

http://qiita.com/sqrtxx/items/d8751e18c549a41f6276

http://qiita.com/key/items/b38066de6de3b8ea3483

