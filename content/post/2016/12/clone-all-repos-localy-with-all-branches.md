---
title: "Склонировать список репозиториев gitlab со всеми ветками"
date: "2016-12-22"
categories: 
  - "bash"
  - "linux"
tags: 
  - "backup"
  - "bash"
  - "gitlab"
  - "резервная копия"
---

Задача: сделать резервную копию на внешний накопитель репозиториев Gitlab.

<!--more-->

```bash
#!/bin/bash

# clone all repos from "apsh" project localy with all brnaches for backup needs.
# use key for authentification

DIRECTORY=~/ApshGit

cd $DIRECTORY

# in repos.txt all repos without ".git" in name
for i in $(cat repos.txt);do
	git clone ssh://git@XX.XX.XX.XX:2001/apsh/$i
	cd $i
	# checkout all branches localy. default it's not. only master branch clone
	for branch in $(git branch -a | grep remotes | grep -v HEAD | grep -v master ); do
    	    git branch --track ${branch#remotes/origin/} $branch
	done
	cd ..
done

```
