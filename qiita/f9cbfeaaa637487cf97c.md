---
title: slackに当日の自分の作業分のgit logを流す
tags: Slack Git 日報
author: n_slender
slide: false
---
# どんな記事か？
その日に、どのくらいコミットして、変更を加えたのかを知りたくなったので、slackに流すようにしてみました。

# どうやったか？

色んなところで使えるように、incoming webhookにリクエストするシェルスクリプトをつくって、それをcronで実行する形をとりました。


```daily-git-report.sh
#!/bin/sh

HOOK_URL=""
slack_channel=""

REPO=""
AUTHOR=""

cd ${REPO}
COMMIT_LOG=$(git shortlog --all --no-merges --author="${AUTHOR}"  --since="midnight")

LINES=$(git log --all --numstat --pretty="%H" --author="${AUTHOR}" --since="midnight" | awk 'NF==3 {plus+=$1; minus+=$2} END {printf("+%d, -%d\n", plus, minus)}')

MESSAGE="${COMMIT_LOG} \\r\\n [insertions, deletions]: \\r\\n ${LINES}"

PAYLOAD="payload={\"channel\":\"${slack_channel}\" , \"text\":\"${MESSAGE}\"}"

curl -X POST --data-urlencode "${PAYLOAD}" ${HOOK_URL}
```

これで、次のようなのが、slackに投稿されます。`git shortlog`でコミット数とコミットメッセージの1行目を、[こちらの記事](http://koyamay.hatenablog.com/entry/2014/10/06/022654)をもとに、追加・削除した行数を算出したのを出しています。

```
author(10)
  commit message 1
  commit message 2
  ...
  commit message 10
[insertions, deletions]
+XXX -YYY

```

あとはcronで、

`30 17 * * 1-5 /path/to/daily-git-report.sh > /dev/null`　

などと、平日の定時のタイミングで実行させれば完了です。

コミットメッセージの書式は最低限のルールは守らないと、綺麗に表示されないので気をつけましょう。[こちら](http://qiita.com/itosho/items/9565c6ad2ffc24c09364)や[Gitのコミットメッセージの書き方](http://postd.cc/how-to-write-a-git-commit-message/)　を参考にしました。

# 何が期待できるか

開発チームのチャンネルに投稿することで、

- 自分の進捗や作業内容を報告
- 人目にさらすことで、コミットメッセージをわかりやすくしようとする
- コミット粒度にも意識するようになる

と、なるのではないかと。とりあえず、自分だけでまずやってみます。

# 参考サイト

https://www.generation.ne.jp/topics/gitessagetoslack/

http://stackoverflow.com/questions/5113425/how-to-make-git-log-show-all-of-todays-commits

http://qiita.com/take4s5i/items/15d8648405f4e7ea3039

http://qiita.com/takashibagura/items/6d03cdd9ab2f88df828d

http://koyamay.hatenablog.com/entry/2014/10/06/022654

https://gist.github.com/eyecatchup/3fb7ef0c0cbdb72412fc

https://gist.github.com/lexqt/2720263

http://qiita.com/rubytomato@github/items/6558bfdb37d982891c09

http://dqn.sakusakutto.jp/cron-maker.html

http://qiita.com/itosho/items/9565c6ad2ffc24c09364

