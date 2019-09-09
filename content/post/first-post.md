---
title: Hugo+Netlify+Netlify CMS
date: 2019-09-07T06:48:27.000Z
draft: true
---
まだ整ってない部分はあるものの、とりあえずスタートさせるくらいまでには整ったので、備忘録として書いておく

## Hugo

![hugo](/img/hugo-logo-wide.svg)

[quick start](https://gohugo.io/getting-started/quick-start/) に従って作ったやり方を記載

1. hugo のインストール

```sh
# brew install hugo
sudo apt-get install hugo
```

2. 新しいサイト (プロジェクトの作成)

```sh
hugo new site blog
```

3. テーマを追加

今回は casper を使ったので、その手順

```
cd blog
git init
git submodule add https://github.com/vjeantet/hugo-theme-casper themes/casper

echo 'theme = "casper"' >> config.toml
```

4. コンテンツの追加

```sh
# 公式のquick start の場合: hugo new posts/my-first-post.md
hugo new post/my-first-post.md
```

ここのディレクトリはthemeによって違うのかな

```sh
cat content/post/my-first-post.md
```

でみると

```
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
```

のようなファイルが作られる

これが一つの記事の内容になる

5. サーバの起動
ローカルで検証をする

```
hugo server -D
```

http://localhost:1313/ にアクセスすると作ったページが見られるはず。

`-D` をつけると draft になっている記事も生成してくれる

6. github に push しておく

これで、hugo側の簡単な用意は完了。色々設定や用意するのは必要だが、追々やっていけばいいのでは

# Netlify

Netlify CMS は多分 Netlify でホスティングさせなくても使えそう (未検証) だが、一定無料で使えるので今回は Netlify でHosting

1. 
