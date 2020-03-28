---
title: RedmineのアカウントでSubversionリポジトリにアクセスする
tags: Redmine Subversion Apache
author: n_slender
slide: false
---
新規のプロジェクトでSubversionが選択されるケースも少なっていそうだし、よくあるネタですがすこし躓いたので整理してメモっておきます。

ほとんどは[ここ](http://www.redmine.org/projects/redmine/wiki/Repositories_access_control_with_apache_mod_dav_svn_and_mod_perl)に書いてあります。

## 前提

* RedmineとSubversionが同一サーバ
* apacheのperl_module、dav_svn_module、authz_svn_moduleのモジュールが使えるようになっていること[^1]
* /var/www/svn以下がSubversionのリポジトリの置き場所

## ポイント

* プロジェクト識別子とリポジトリ名を一致させる
* プロジェクトメンバーのアカウントでのみアクセス可能

## subversion側の設定

### ディレクトリのアクセス権限

apacheユーザが読み書きできるようにします。

`chown -R apache:apache /var/www/svn`

SELinuxが有効な状態では下記が必要みたいです。

`chcon -R system_u:object_r:httpd_sys_content_t /var/www/svn`

### apacheの設定ファイル

apacheのconf.dにsubversion用の設定ファイルを追加します。
ここで/svnのアクセスに、Basic認証して、RedmineのDBを見に行くように設定します。

```subversion.conf

LoadModule perl_module        modules/mod_perl.so
PerlLoadModule Apache::Authn::Redmine

LoadModule dav_svn_module     modules/mod_dav_svn.so
LoadModule authz_svn_module   modules/mod_authz_svn.so

RequestHeader edit Destination ^https http early
<Location /svn>
   DAV svn
   SVNParentPath /var/www/svn　

   AuthType Basic
   AuthName "subversion repository"
   PerlAuthenHandler Apache::Authn::Redmine::authen_handler

   # RedmineのDBの接続情報
   RedmineDSN "DBI:mysql:database=redmine;host=localhost"
   RedmineDbUser "redmine"
   RedmineDbPass "redmine"

   Require valid-user
</Location>

```

## Redmineとの連携手順

あとは、Redmineのプロジェクトと連携できるように、プロジェクトとリポジトリを作成します。

1. プロジェクトを作成
2. プロジェクト識別子と同じ名前でリポジトリを作成
3. アクセスさせたいユーザをプロジェクトのメンバーに追加
4. http://[redmine_server]/svn/[project_identifier]でリポジトリにアクセスできることを確認

### 1. プロジェクトを作成

Redmineの画面から作ります。識別子をおぼえておきます。

### 2. プロジェクト識別子と同じ名前でリポジトリを作成

これがポイントです。ここを間違えてハマリました...

```
# cd /var/www/svn
# svnadmin create [project_identifier]
# chown -R apache:apache [project_identifier]
```

SELinuxが有効な状態では下記が必要なです。

`chcon -R system_u:object_r:httpd_sys_content_t [project_identifier]`

### 3. アクセスさせたいユーザをプロジェクトのメンバーに追加

プロジェクト識別子で認証対象を制限しているので、アクセスできるのはプロジェクトのメンバーに限定されます。プロジェクトの設定タブのメンバーから、追加します。

（これに気付くのに、わざわざSQLのログを出して確認してしまいましたよ...）

### 4. http://[redmine_server]/svn/[project_identifier]でリポジトリにアクセスできることを確認

ブラウザからアクセスしてもよいですし、

`svn ls "http://[redmine_server]/svn/[project_identifier]"` でも確認できます。

認証が要求されるので、プロジェクトメンバーのアカウント情報を入力して、アクセスできればOKです。

## 参考サイト

http://www.redmine.org/projects/redmine/wiki/Repositories_access_control_with_apache_mod_dav_svn_and_mod_perl

http://idlysphere.blog66.fc2.com/blog-entry-239.html

http://qiita.com/kota_na_/items/46faf25268953da5fd37

http://r1990297.hatenablog.com/entry/2013/11/20/093114

https://www.redmine.org/boards/2/topics/6387

https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Working_with_SELinux-SELinux_Contexts_Labeling_Files.html

http://www.sera.desuyo.net/WebDAV/selinux.html

### CentOSへのapache+subversionの入れ方

* [7.x](https://www.howtoforge.com/how-to-install-svn-with-apache-dav_svn-on-centos-7)
* [6.x以前](https://wiki.centos.org/HowTos/Subversion)

[^1]: CentOSなら `yum install mod_perl mod_dav_svn subversion`

