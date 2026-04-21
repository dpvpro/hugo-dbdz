---
title: "Миграция репозиториев Gitlab на другой сервер"
date: 2016-12-22
lastmod: 2026-07-21
categories:
  - "devops"
tags:
  - "gitlab перенос репозиториев"
---

<!--more-->

Необходимо перенести репозитории *Gitlab* на другой сервер.

```bash
#!/bin/bash

# migrate repos to another gitlab.

# directory with code
DIRECTORY=~/Code
# name of repository
REPO_NEW=enserver
# name of old repository
REPO_OLD=${REPO_NEW}.git

cd $DIRECTORY

echo "Clone from old server"
git clone --mirror git@gitlab:apsh/$REPO_OLD

cd $REPO_OLD
# create new repo on new gitlab server
# repo will be private by default
git push --set-upstream ssh://git@XX.XX.XX.XX:2001/apsh/$REPO_NEW newserver
git remote add newserver ssh://git@XX.XX.XX.XX:2001/apsh/$REPO_NEW

echo "Push in new server"

# push everything back to the new repo.
git push --mirror newserver
```
