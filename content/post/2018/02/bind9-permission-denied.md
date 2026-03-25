---
title: 'Ошибка "Permission denied" в Bind9'
date: 2018-02-12
lastmod: 2026-03-25
categories:
  - "linux"
tags:
  - "ubuntu apparmor bind"
---

При настройке сервиса `Bind` на **Ubuntu 16.04** столкнулся с ошибкой:

```bash
isc_stdio_open '/var/log/bind/misc.log' failed: permission denied
```

<!--more-->

Проблема кроется в усиленной защите `AppArmor`.

Для исправления разрешаем запись в файл лога в файле конфигурации `AppArmor`. Добавляем в файле следующую строку:

```bash
sudo vim /etc/apparmor.d/usr.sbin.named
```
```bash
/var/log/dns/** rw,
```

Другой вариант отключаем `AppArmor`:

```bash
sudo systemctl stop apparmor
sudo systemctl disable apparmor
```
