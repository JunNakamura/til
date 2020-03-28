---
title: SubGitでGitリポジトリをSubversionリポジトリへ移行
tags: subgit Git Subversion GitHub
author: n_slender
slide: false
---
# どんな人向けか？
諸事情でGitからSubversionに移行するケースを想定しています。
例えば、GitHubの有料プランをやめて、社内にもとからあるSubversionのリポジトリサーバに全て統一しようとか。

# そもそもSubGitはSubversionからGitに移行するためのものでは？

[SubGit](http://www.subgit.com/) は、gitとsvnのマイグレーションツールのようです。もちろん、gitへの移行が主なユースケースだと思います。今回はgitとsvnのリポジトリの同期をとれるのを利用して、gitからsubversionへの移行を実現します。フリーの機能だけでいけます。

# git-svnでもできるのでは？

もちろんできます。下記も参考しました。

http://qiita.com/digdagdag/items/043ebba864fa289c9a89 

http://www.coltware.com/2010/02/18/git_to_subversion/

ただ、すこし試してみて時間がかかりそうだったので、別のを探した結果、SubGitにいきつきました。

#　手順

1. SubGitのインストール
2. 移行先のリSubversionリポトリ作成
3. SubGitで移行用Gitベアリポジトリ作成
4. 移行元の内容をコピー
5. Subversionへコミット
6. checkoutして確認

わかりやすくするために、GitHubのリポジトリを、Subversionリポジトリがあるサーバに移行する形で記述します。

##  1. SubGitのインストール

公式サイトを参照。java7以上が必要なので、事前にインストールしておきます。
http://www.subgit.com/downloadingzip.html

展開したもの中にある、bin/subgitへのパスを通しておきます。

## 2. 移行先のSubversionリポジトリ作成

通常の手順通りにつくって、trunk,branches,tagsディレクトリを作成したのをコミットしておきます。
便宜上、リポジトリ名は移行元のと同じものとします。

```
# cd /var/www/svn
# svnadmin create [repo_name]
# svn mkdir file:///var/www/svn/[repo_name]/{trunk,branches,tags} -m "create trunk,branches and tags."
```

## 3. SubGitで移行用Gitベアリポジトリ作成

SubGitを使って、先につくったSubversionリポジトリと同期する、Gitのベアリポジトリをつくります。

```
# cd /var/www/git
# subgit configure --layout auto file:///var/www/svn/[repo_name]
# subgit install [repo_name].git
```

ディレクトリ作成のリビジョン1が、最初のコミットとして登録されます。
（Subversionから移行する場合、これでほぼ完了になります。移行するだけなら、subgit importをつかえばよいようですが。）

## 4. 移行元の内容をコピー

移行用のベアリポジトリからgit cloneして、作業用のリポジトリを用意します。

```
# git clone /var/www/git/[repo_name].git 
```

一旦、リモートのURLを移行元のGitHubリポジトリに書き換えます。

```
# cd /var/www/git/[repo_name]
# vi .git/config
```

下記のようにすればOKです。sshでとる場合、GitHubに公開鍵を登録しておきます。
（gitのバージョンが1.7.10未満だと、httpsのbasic認証での取得ができないバグある模様。CentOSだと、epelなどの別のリポジトリから新しいバージョンを取得するか、ソースからビルドするかのどちらか）

```
[remote "origin"]
        fetch = +refs/heads/*:refs/remotes/origin/*
        #url = /var/www/git/[repo_name].git
         url = git@github.com:[user_name]/[repo_name].git
```

これで、とにかくpullしてきます。subversionでディレクトリをきったのに相当するコミットが、先頭にきてしまいますが、subversionに移したときのリビジョンはうまくいっているので、このままいきます。

```
# git pull
```

つぎはリモートブランチの取得です。移行したいものだけチェックアウトするか、面倒なら全部やります。

```
# for branch_name in `git branch -r | grep -v master`;do
> git checkout -t $branch_name
> done
```

ただし、ブランチ名にスラッシュが含む（ex. feature/XXX）場合は、SubGitの設定ファイルで調整が必要です。

/var/www/git/[repo_name].git/subgit/config に `branches = branches/feature/*:refs/heads/feature/*` などと追記します。詳しくは[この記事](https://issues.tmatesoft.com/issue/SGT-769)をどうぞ。


## 5. Subversionへコミット

必要ものを取得できたら、リモートのURLをもとに戻します。

```
[remote "origin"]
        fetch = +refs/heads/*:refs/remotes/origin/*
        url = /var/www/git/[repo_name].git
        #url = git@github.com:[user_name]/[repo_name].git
```

あとはpushするだけです。タグもオプションつけてpushします。

```
# git push
# git push --tags
```

SubGitで、最初につくったSubversionリポジトリと連動しているので、リビジョンがコミット数に応じて進んでいきます。コミット数が1000くらいあっても、数分で終わりました。

## 6. checkoutして確認

適当なところに、Subversionリポジトリをチェックアウトして、移行できたか確認します。

```
# cd /tmp
# svn checkout file:///var/www/svn/[repo_name]
# cd [repo_name]
# svn log -l 5
```

チェックアウトのときに、想定しているファイルやブランチなどが標準出力されていれば、まず大丈夫です。
念のため、適当にsvn logで、直近のリビジョンのコミットメッセージが、gitのと対応しているかを確認しておきます。

# まとめ

SubGitがあれば、gitとsvnの行き来は結構できますね。これでgitにもsvnにも移行できるノウハウが得られました。

## 蛇足

実は、GitHubはsvnクライアントからでもアクセスできます。
https://help.github.com/articles/support-for-subversion-clients/

tortoisesvnで普通にチェックアウトできましたよw 
最初はこのワーキングコピーをつかって、subversionのリポジトリに移行できないかと思いましたが、subversionは中央集権型だから、それにつながるコマンドはないですよね... リポジトリが死んだら、過去のリビジョンを復元するのは絶望的なんでしょうか。

GitHub側も、svnから移行するための機能を用意していますね。
https://help.github.com/articles/importing-from-subversion/





