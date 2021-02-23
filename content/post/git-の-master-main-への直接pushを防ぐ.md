---
title: Git の master/main への直接pushを防ぐ
date: 2021-02-23T05:59:03.887Z
image: img/github-mark-120px-plus.png
draft: true
---
誤って master/main へ直接pushしてしまわないようにクライアント側のhookで、特定のbranch名へのpushを禁止する

ref: https://git-scm.com/docs/githooks

## プロジェクトごとの pre-push hook

プロジェクトごとの hook path は `.git/hooks/` になっているので、ここにファイルを作成する。

```text:.git/hooks/pre-push
#!/bin/bash
protected_branches=('main' 'master')
while read local_ref local_sha1 remote_ref remote_sha1
do
  if [[ "${protected_branches[@]}" =~ "${remote_ref##refs/heads/}" ]]; then
    echo "Do not push to master branch!!!"
    exit 1
  fi
done
```

### (参考) hook の template を作る

init する際に、template をもとに、そのプロジェクトにhookが追加される

```
git config --global init.templatedir "~/.git_template/"
```

```text:.git_template/hooks/pre-push
#!/bin/bash
protected_branches=('main' 'master')
while read local_ref local_sha1 remote_ref remote_sha1
do
  if [[ "${protected_branches[@]}" =~ "${remote_ref##refs/heads/}" ]]; then
    echo "Do not push to master branch!!!"
    exit 1
  fi
done
```

## global の hookPath を設定する

全てのproject共通でhookさせたい場合は、globalで設定する

```
git config --global core.hooksPath "~.git_hooks"
```

```text:~/.git_hooks/pre-push
#!/bin/bash
protected_branches=('main' 'master')
while read local_ref local_sha1 remote_ref remote_sha1
do
  if [[ "${protected_branches[@]}" =~ "${remote_ref##refs/heads/}" ]]; then
    echo "Do not push to master branch!!!"
    exit 1
  fi
done
```