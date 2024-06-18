---
title: "Миграция репозиториев gitlab на другой сервер"
date: "2016-12-22"
categories:
  - "DevOps"
tags:
  - "gitlab перенос репозиториев"
---

<!--more-->

Задача: перенести репозитории *Gitlab* на другой сервер. Данная задача решена в полуавтоматическом режиме.
Требуется вручную создавать репозиторий на новом gitlab.

**Update 04.01.2017**

Сейчас я знаю как это можно решить в автоматическом режиме, но не буду дописывать, потому что это можно доработать самостоятельно.
Требуется всего лишь дописать строку создающую репозиторий на новом сервере.

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

echo -e "Clone from old server\n"
git clone --mirror git@gitlab:apsh/$REPO_OLD

# Create a blank repo on the new gitlab manualy with name $REPO_NEW
# Add the new repo as a remote on the dev machine.
cd $REPO_OLD
git remote add master ssh://git@XX.XX.XX.XX:2001/apsh/$REPO_NEW

echo -e "\nPush in new server\n"

# Push everything back to the new repo.
git push --mirror master

```
