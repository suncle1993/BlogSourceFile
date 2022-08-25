---
categories:
  - 效率
date: '2020-09-25T13:25:35'
description: ''
tags:
  - hook
  - git
title: git禁止在master分支push和commit
---




作为管理者，在远端将master分支设为保护分支，可以从根源上杜绝直接推送到master的问题。dev分支同理。

作为开发者，在本地的git hook中加配置可以做到在commit和push操作时做对应的检查

<!--more-->


### 禁止在master分支上Commit

```bash
#!/bin/sh

protected_branch='master'
current_branch=$(git rev-parse --symbolic --abbrev-ref HEAD)

if [ "$protected_branch" == "$current_branch" ]; then
  echo ".git/hooks: Do not commit to $current_branch branch"
  exit 1
fi

```



### 在master分支上Commit时提示

```bash
#!/bin/sh

protected_branch='master'
current_branch=$(git rev-parse --symbolic --abbrev-ref HEAD)

if [ "$protected_branch" == "$current_branch" ]; then
  read -p "You're about to commit to master, is that what you intended? [y|n] " -n 1 -r </dev/tty
  echo
  if echo "$REPLY" | grep -E '^[Yy]$' >/dev/null; then
    exit 0 # commit will execute
  fi
  exit 1 # commit will not execute
fi

```



### 禁止推送到master分支

```bash
#!/bin/sh

protected_branch='master'
remote_branch_prefix="refs/heads/"
protected_remote_branch=$remote_branch_prefix$protected_branch

while read local_ref local_sha remote_ref remote_sha
do
	if [ "$protected_remote_branch" == "$remote_ref" ]; then
		echo ".git/hooks: Do not commit to $protected_branch branch"
	  exit 1
	fi
done

exit 0

```



### 推送到master分支时提示

```bash
#!/bin/sh

protected_branch='master'
remote_branch_prefix="refs/heads/"
protected_remote_branch=$remote_branch_prefix$protected_branch

while read local_ref local_sha remote_ref remote_sha
do
	if [ "$protected_remote_branch" == "$remote_ref" ]; then
		read -p "You're about to push master, is that what you intended? [y|n] " -n 1 -r < /dev/tty
    echo
    if echo $REPLY | grep -E '^[Yy]$' > /dev/null
    then
        exit 0 # push will execute
    fi
    exit 1 # push will not execute
	fi
done

exit 0

```

> 为什么需要循环读取？因为git一次可以push多个分支



### 推送时如果commit消息包含WIP则禁止推送

```bash
#!/bin/sh

z40=0000000000000000000000000000000000000000

while read local_ref local_sha remote_ref remote_sha; do
  if [ "$local_sha" = $z40 ]; then
    # Handle delete
    :
  else
    if [ "$remote_sha" = $z40 ]; then
      # New branch, examine all commits
      range="$local_sha"
    else
      # Update to existing branch, examine new commits
      range="$remote_sha..$local_sha"
    fi

    # Check for WIP commit
    commit=$(git rev-list -n 1 --grep '^feat: WIP' "$range")
    if [ -n "$commit" ]; then
      echo >&2 "Found WIP commit in $local_ref, not pushing"
      exit 1
    fi
  fi
done

exit 0

```



这时候，你可能会发现，你每一次clone项目之后都需要手动把commit和push的hook文件丢在`.git/hooks`目录下，是不是觉得不方便？别着急，有办法，我们可以让所有项目的hook操作统一到一个自定义目录中。

```bash
mkdir ~/.git-hooks	# 创建一个存放hook的自定义目录
git config --global core.hookspath ~/.git-hooks	# 更改git配置指定hook目录到自定义，先别着急执行，往后看
```

这样就可以实现统一管理所有项目的hooks操作了

> core.hookspath配置需要git版本在v2.9以上才行


然后，你会觉得全局统一管理也太霸道了吧，比如说，公司的项目可以统一一套hooks操作，但是我不想把这一套hooks应用于个人github的项目啊。也就是说你需要在不同的目录下面执行不同的hooks操作，那么该怎么办呢？还是有办法：git配置是可以根据不同目录使用不同配置的

比如我只想统一管理`~/yy`目录下的所有项目，那就修改`~/.gitconfig`文件加入以下内容

```ini
[includeIf "gitdir:~/yy/"]
    path = .gitconfig-yy
```

然后增加一个`~/.gitconfig-yy`文件，在这个文件中加入yy目录下面的独有配置

```ini
[core]
    hookspath = ~/.git-hooks
```



---

参考：

1. https://stackoverflow.com/questions/42455506/in-pre-push-hook-get-git-push-command-full-content

2. https://www.geek-share.com/detail/2776108340.html

我的博客即将同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=38qhpnqeksg0g
