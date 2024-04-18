---
title: "Bind9 - isc_stdio_open '/var/log/bind/misc.log' failed: permission denied"
date: "2018-02-12"
categories: 
  - "linux"
tags: 
  - "apparmor"
  - "bind"
  - "ubuntu"
---
При настройке сервиса _bind_ на Ubuntu 16.04 столкнулся с ошибкой:

`isc_stdio_open '/var/log/bind/misc.log' failed: permission denied`

Проблема кроется в усиленной защите AppArmor.

Для исправления разрешаем запись в файл лога в файле конфигурации AppArmor `/etc/apparmor.d/usr.sbin.named`:

```bash
/var/log/dns/** rw,
```

Другой вариант отключаем AppArmor:

```bash
sudo systemctl stop apparmor
sudo systemctl disable apparmor
```
