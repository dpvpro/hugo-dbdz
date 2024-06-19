---
title: "Исправление сложности пароля на VmWare 6.0+"
date: "2017-03-04"
categories:
  - "devops"
tags:
  - "vmware сложность пароля"
---

С выходом VmWare 6.0 и последующих версий, изменились требования сложности пароля для пользователей.
Но сложные пароли не всегда актуальны. Для возможности установить простые пароли типа `zimaleto345` нужно внести изменения на хосте *Vmware Esxi*.

<!--more-->

Сложность пароля управляется в файле `/etc/pam.d/passwd`. Комментируем все строки, оставляем только одну в виде:

```bash
password sufficient /lib/security/$ISA/pam_unix.so nullok shadow sha512

```

Сохраняем файл. После этого можно поставить простой пароль.

[https://www.perfectcloud.org/fix-it/vmware/disable-esxi-password-complexity/](https://www.perfectcloud.org/fix-it/vmware/disable-esxi-password-complexity/)
