---
title: git 重置 commit 中的 name 和 email
date: 2017-09-06 18:28:14
tags:
---
# git 重置 commit 中的 name 和 email

这两天换了自己的电脑开发公司的项目，项目 clone 下来之后二话不说直接开始写代(bu)码(g)，直到今天下午 `git log` 时才发现 user name 和 user email 还没有更改默认的值，这就很尴尬了。

之前也遇到过重置 user name 和 user email 的情况，但是一般都是在首次提交后就想起来需要更改，那么首先使用 `git config set user.name "my_name" && git config set user.email "email@foo.com"` 设置当前 repo 的 name 和 email，然后使用 `git commit --amend --reset-author` 就可以重置完成了， 而像这次后知后觉，开发了半个星期才发现倒是第一次遇到，期间已经提交了若干次，无法简单的通过重置上一次的 commit 信息就能完成了。

搜了一圈发现 `git filter-branch` 命令就是为了解决这样的[“核弹级”命令](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)，尝试了一发果然爽翻。直接上代码：

```shell
git filter-branch --commit-filter '
	if [ "$GIT_AUTHOR_NAME" = 'origin_name' ];
	then
		GIT_AUTHOR_NAME="new_name"; 		
		GIT_AUTHOR_EMAIL="new_email@foo.com"; 
		git commit-tree "$@"; 
	else
		git commit-tree "$@"; 
	fi' HEAD
```

切记过滤条件不能省略，不然跟你一起开发的同事非打死你不可。