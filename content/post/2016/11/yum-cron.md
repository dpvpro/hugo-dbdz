---
title: "Автоматизация обновления ОС и ПО на CentOS 7 с помощью Yum-Cron"
date: "2016-11-16"
categories: 
  - "centos"
  - "linux"
tags: 
  - "centos"
  - "yum"
  - "cron"
---

CentOS славится своей стабильностью и удобными системными инструментами. Сегодня мы рассмотрим программу `yum-cron` для CentOS 7.
Она служит для автоматизации установки обновлений. Устанавливаем:

```bash
yum -y install yum-cron
chkconfig yum-cron on

```

Поведение программы контролируют 3-и параметра в файле `/etc/yum/yum-cron.conf` или `/etc/yum/yum-cron-hourly.conf`. 

Эти параметры по умолчанию выставлены в `no`. Необходимо изменить это. Я не буду использовать `/etc/yum/yum-cron-hourly.conf`, который отвечает за ежечасные обновления. Будем работать с `/etc/yum/yum-cron.conf`, который отвечает за обновления раз в сутки.

```bash
# Whether a message should emitted when updates are available.
update_messages = no
# Whether updates should be downloaded when they are available. Note
# that updates_messages must also be yes for updates to be downloaded.
download_updates = no
# Whether updates should be applied when they are available. Note
# that both update_messages and download_updates must also be yes for
# the update to be applied
apply_updates = no

```

Используем замену с использованием `sed` для того что бы изменить настройки в конфигурационном файле.

```bash
sed -i 's|^update_messages = no|update_messages = yes|' /etc/yum/yum-cron.conf
sed -i 's|^download_updates = no|download_updates = yes|' /etc/yum/yum-cron.conf
sed -i 's|^apply_updates = no|apply_updates = yes|' /etc/yum/yum-cron.conf
egrep '^update_messages|^download_updates|^apply_updates' /etc/yum/yum-cron.conf
service yum-cron restart
```

Далее нужно сделать самый главный выбор. В каком режим будет обновляться хост. Я думаю все понятно из описания. Богатый выбор для любителей стабильных систем.

```bash
[commands]
# What kind of update to use:
# default                            = yum upgrade
# security                           = yum --security upgrade
# security-severity:Critical         = yum --sec-severity=Critical upgrade
# minimal                            = yum --bugfix update-minimal
# minimal-security                   = yum --security update-minimal
# minimal-security-severity:Critical =  --sec-severity=Critical update-minimal
update_cmd = security

```
