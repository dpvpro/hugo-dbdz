---
title: "Долгое монтирование NFS папки"
date: "2016-04-05"
categories: 
  - "linux"
tags: 
  - "nfs"
---
<!--more-->

Если монтирование сетевой папки NFS занимает длительное время, значит сервер не поддерживает NFS4. Нужно подключаться с использованием NFS3:

`sudo mount -v -t nfs server:/path/to/share  /mnt  -o nfsvers=3`

Параметр nfsvers=3 следует добавить в существующую строку монтирования.

Для файла fstab монтирование осуществляется строкой:

`10.10.152.40:/distrib    /distrib    nfs    auto,vers=3,mountvers=3,local_lock=none    0   0`
