---
title: GitHub をHTTPSで使う
date: 2020-08-05T12:06:06.507Z
image: img/github-mark-120px-plus.png
draft: true
---
GitHubではsshよりhttpsでの通信を推奨している

sshを使ってgitを使ってる人も多いと思うので、httpsを使ってgitを使う方法を書く

linux の cui 環境での想定のため、gnome-keyring などはメインで扱わない

## credential helper を設定する

helper を設定することで、毎回 account, password を入力する手間を省く

なお、 github で 2FA を利用している場合は、access tokenを発行して password の代わりに入力する必要がある。

ref: https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E8%AA%8D%E8%A8%BC%E6%83%85%E5%A0%B1%E3%81%AE%E4%BF%9D%E5%AD%98

### cache を使い一定時間記憶させる

helper に cache を指定することで一定時間 account, password を記憶させておくことが出来る

`--timeout` option を利用して、キャッシュさせておく時間を指定出来る (default は 900 秒)

```
# 1時間 cache させておく
git config --global credential.helper "cache --timeout 3600"
```

```
# 一回目は username, password を聞かれる
% git push -n
Username for 'https://github.com': 
Password for 'https://xxx@github.com':
# 二回目は聞かれない
% git push -n
```

### store を使いディスクに保存する

helper に store を指定することで、 account, password をディスクに保存することが出来る。
これをすることで、 password を変更したり、tokenを無効化するまで account, password の入力をシなくても良くなります。
ただし、 file に password が暗号化されずに保存されているため、リスクがあります。

`--file` を指定することで、保存先を設定できます。 (default は `~/.git-credentials`)

```
# ~/.git-credentials に account, password を保存
git config --global credential.helper store
# ~/.my-credentials に account, password を保存
git config --global credential.helper "store --file ~/.my-credentials"
```

```
# 一回目は username, password を聞かれる
% git push -n
Username for 'https://github.com': 
Password for 'https://xxx@github.com':
# 二回目は聞かれない
% git push -n
```


### keycahin を使う

未確認。 平文でパスワードを保存する必要がなくなるのでより安全。

#### Mac

```
git credential-osxkeychain
git config --global credential.helper osxkeychain
```

#### windows

```
git config --global credential.helper wincred
```

#### Linux (Ubuntu)

```
sudo apt install libssl-dev libsodium-dev libsecret-1-dev libgnome-keyring-dev
cd /usr/share/doc/git/contrib/credential/gnome-keyring
sudo make
git config --global credential.helper /usr/share/doc/git/contrib/credential/gnome-keyring/git-credential-gnome-keyring
```