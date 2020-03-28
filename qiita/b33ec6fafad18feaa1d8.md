---
title: Symfony2.xの開発経験から得たtips
tags: PHP Symfony2
author: n_slender
slide: false
---
# はじめに

[Symfony Advenet Calander 2016](http://qiita.com/advent-calendar/2016/symfony)の21日目の記事です。

symfonyについてのちょっとしたHow Toものです。開発しているときに、ちょっとしたことですが「こういうことがやりたい」と思ったことがあって、そのときに編み出した小技です。逆引き集といえるほどでもないですが、ノウハウの共有や別のやり方のもとになれればと思います。

# Profiler

symfonyのprofilerは便利で、かなりの情報がここから得られます。まだ使いきれていないですが、自分の経験で役に立ったと思うものをピックアップします。

## 使用されているテンプレート

profiler画面のtwigより、使用されているtwigテンプレートやどういう順番でレンダリングがされているかが一覧できます。どんなテンプレートを呼び出しているのか、何回繰り返されているのか、レンダリングかかる時間を確認したいときに便利です。

![profiler_twig.png](https://qiita-image-store.s3.amazonaws.com/0/9880/598e3bf8-8766-fdc9-272c-b5c6fc3eabe4.png)


## 表示されるまでのタイムライン

profiler画面のtimelineより、リクエストから画面返却までのタイムラインが表示されます。
画面表示が遅いとき、どこで時間がかかっているかの大枠をおさえることができます。例えば、controller内の処理とテンプレートでのレンダリングのどっちに時間がかかっているかくらいであれば、この機能で十分測定できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/9880/a509a3fe-038a-3286-7b01-e92c798463a2.png)


## 実行されたクエリ(doctrineを使っている場合のみ)

profilerのdoctrineより、その画面で使用されたSQLの情報が表示されます。
prepared sql文に渡されたパラメタ、実際に実行されたSQL文、explainなども見れるので至れり尽くせりです。

![profiler_doctrine.png](https://qiita-image-store.s3.amazonaws.com/0/9880/d00bdab9-f582-21b4-5349-b4344d2f4de8.png)


## dumpによるデバッグ

書こうと思ったら、すでに[去年の記事](http://symnote.kwalk.jp/public/debug) にありました...
いわゆる、プリントデバッグの延長にあるテクニックですが、

ポイントは、

* profilerからすぐに確認できる
* var_dump()はもういらない
* twigでも使える

です。controllerで、意図した分岐に入っているか、想定どおりの値かを確認するのによく使っています。

## マッチしたルーティングの確認

profilerのroutingより、どのルーティングにマッチしたかどうかが確認できます。
ルーティングに正規表現やドメイン制限してる場合は、この画面で想定どおりの動きかを確認するのに役立ちます。

![profiler_routes.png](https://qiita-image-store.s3.amazonaws.com/0/9880/3c499bdf-a3a7-213e-d87f-be0b7324de3f.png)


# ルーティング

## ホスト名での制限

URLのルールは同じだけど、サブドメインによって処理を分けたいとき

```php
@Route("/test/{pageNumber}", host="dev1.example.com")

@Route("/test/{pageNumber}", host="dev2.example.com")
```

## パスパラメタにプレフィックス・サフィックスをもたせる

ページ数などをパスパラメタに持たせたとき、ページ数とわかるように、「page-」などをつけたいとき

```php
@Route("/sample/page-{number}")

@Route("/sample/{number}-page")
```

## 種類などを表すパラメタで、値の数が限られているとき

例えば、（グルメ、旅行、買い物）のようなサイト内で決め打ちのカテゴリがあって、それらのトップページを共通のアクションで実現したいとき

```php
@Route("/top/{category}", requirements={"category"="(gourmet|travel|shopping)"})
```

パターンが少ないときは正規表現を使うと手軽でわかりやすい

# コンソールコマンド

## 標準エラーに出力

```sample.php
 protected function execute(InputInterface $input, OutputInterface $output)
    {
        $std = $output instanceof ConsoleOutputInterface ? $output->getErrorOutput() : $output;
        $std->writeln('<error>error message</error>');
    }
```

## メッセージの装飾

`<info></info>` などでくくると、標準出力での表示がちょっとキレイになります。
詳細は本家のページをどうぞ。
https://symfony.com/doc/current/console/coloring.html


# その他

## キャッシュクリアは「--no-warmup」をつける

[こちらの記事](http://d.hatena.ne.jp/jiskay/20110606/1307332702)　を参考にしました。

ロード時に、リクエストのドメインや、実行環境のホスト名などをみて、定数を決める場合、キャッシュを自動生成させると、思わぬ事故が発生します。時間短縮の観点からもウォームアップなしの方が安全のようです。

`php app/console cache:clear --no-warmup`


## SingletonをDI機能で導入

インスタンスをひとつだけ作ってそれを使い続けたい場合。実はデフォルトで、symfonyのサービスコンテナーは、同じインスタンスを提供しつづける仕様になっています。クラスの方でSingletonとして設計するのもありですが、面倒ならサービスコンテナから提供させるという選択肢もあるということです。

http://symfony.com/doc/current/service_container.html

```
As a bonus, the Mailer service is only created once and the same instance is 
returned each time you ask for the service. This is almost always the behavior 
you'll need (it's more flexible and powerful), but you'll learn later how you can
configure a service that has multiple instances in the How to Define Non Shared 
Services article.
```

例えば、DBに接続するためのインスタンスは、コネクション数を抑制したいなどから、ひとつのインスタンスにしたくなります。そういう場合は下記のようなクラスを作成して、これをDIから提供させればOKです。DIを使っていると、テスト時はテスト用のクラスにするなどの使い分けできるので、Singletonとしてclass作成するより柔軟に使えます。[こちらの記事](http://qiita.com/trashtoy/items/eea5dcfb1eb003101cc9)でも、そういった示唆があり、参考にさせていただきました。

```sample.php

class DBSingleton {

private $DBConnection;

public function __construct() {...}

}

```


ちなみに、2.8から、逆に毎回新しいインスタンスを提供させることもできます。このページでもデフォルトでは一度作成したインスタンスを使い続けるとあります。
http://symfony.com/doc/2.8/service_container/shared.html

```
In the service container, all services are shared by default. 
This means that each time you retrieve the service, you'll get the same instance. 
```

## singletonについての参考サイト

* https://ja.wikipedia.org/wiki/Singleton_%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3
* [シングルトンのベターな実装方法](http://qiita.com/kikuuuty/items/fcf5f7df2f0493c437dc)
* http://hiroaki-kono.hatenablog.com/entry/2014/05/04/135130
* [Singleton何が悪いの？](http://qiita.com/mo12ino/items/abf2e31e34278ebea42c)
* [Singleton パターンの使いどころをまとめてみた](http://qiita.com/trashtoy/items/eea5dcfb1eb003101cc9)

# 参考サイト

* http://kseta.hatenablog.jp/entry/2014/12/02/004208
* http://shiro-goma.hatenablog.com/entry/2014/12/19/114144
* http://tdak.hateblo.jp/entry/20141213/1418462893
* http://okapon-pon.hatenablog.com/entry/symfony-debug
* http://qiita.com/tarokamikaze/items/b6cb73be0294fe6b14c1
* http://d.hatena.ne.jp/jiskay/20110606/1307332702
* http://tech.quartetcom.co.jp/2015/12/11/easyadminbundle/
* http://symnote.kwalk.jp/public/debug
* https://gist.github.com/MattKetmo/3665488
* https://symfony.com/blog/new-in-symfony-2-4-show-logs-in-console
* https://symfony.com/doc/current/console/coloring.html


