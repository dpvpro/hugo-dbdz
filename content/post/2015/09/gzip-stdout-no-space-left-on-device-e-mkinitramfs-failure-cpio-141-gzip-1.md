---
title: "Ошибка при обновлении Ubuntu Server 14.04"
date: "2015-09-28"
categories:
  - "linux"
tags:
  - "gzip stdout no space left on device"
---

Столкнулся с тем, что при обновлении Ubuntu Server 14.04 возникает ошибка:

`gzip: stdout: No space left on device E: mkinitramfs failure cpio 141 gzip 1`

<!--more-->

При ближайшем рассмотрении, выяснилось что она появляется при обновлении ядра системы. Оказалось, что закончилось место в каталоге `/boot`.
Данный каталог был создан как отдельный раздел в системе. Другие разделы были созданы через `lvm`.

Как известно, для такой схемы нужен отдельный каталог `/boot`. Далее выясняется, что при обновлении ядер, в каталог `/boot` записывается служебная информация.
Так вот, конфигураций ядра было слишком много что бы поместиться в этом разделе.

Решением было удалить лишние ядра:

`sudo apt-get autoremove`

Или же в ручном режиме:

`dpkg -l 'linux-image-*' | grep '^ii'`

и далее удаляем какое либо не нужное ядро:

`sudo dpkg --remove linux-image-extra-3.19.0-28-generic`

Листинг процесса ошибки в консоли выглядел так:

```bash
Log started: 2015-09-28 14:44:44
Настраивается пакет initramfs-tools (0.103ubuntu4.2) …
update-initramfs: deferring update (trigger activated)
Настраивается пакет linux-image-extra-3.13.0-65-generic (3.13.0-65.105) …
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 3.13.0-65-generic /boot/vmlinuz-3.13.0-65-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 3.13.0-65-generic /boot/vmlinuz-3.13.0-65-generic
update-initramfs: Generating /boot/initrd.img-3.13.0-65-generic
gzip: stdout: No space left on device
E: mkinitramfs failure cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-3.13.0-65-generic with 1.
run-parts: /etc/kernel/postinst.d/initramfs-tools exited with return code 1
dpkg: error processing package linux-image-extra-3.13.0-65-generic (--configure):
подпроцесс установлен сценарий post-installation возвратил код ошибки 1
dpkg: зависимости пакетов не позволяют настроить пакет linux-image-generic:
linux-image-generic зависит от linux-image-extra-3.13.0-65-generic, однако:
Пакет linux-image-extra-3.13.0-65-generic пока не настроен.
dpkg: error processing package linux-image-generic (--configure):
проблемы зависимостей — оставляем не настроенным
dpkg: зависимости пакетов не позволяют настроить пакет linux-generic:
linux-generic зависит от linux-image-generic (= 3.13.0.65.71), однако:
Пакет linux-image-generic пока не настроен.
dpkg: error processing package linux-generic (--configure):
проблемы зависимостей — оставляем не настроенным
dpkg: зависимости пакетов не позволяют настроить пакет linux-generic-lts-raring:
linux-generic-lts-raring зависит от linux-generic, однако:
Пакет linux-generic пока не настроен.
dpkg: error processing package linux-generic-lts-raring (--configure):
проблемы зависимостей — оставляем не настроенным
dpkg: зависимости пакетов не позволяют настроить пакет linux-image-generic-lts-raring:
linux-image-generic-lts-raring зависит от linux-image-generic, однако:
Пакет linux-image-generic пока не настроен.
dpkg: error processing package linux-image-generic-lts-raring (--configure):
проблемы зависимостей — оставляем не настроенным
Processing triggers for initramfs-tools (0.103ubuntu4.2) ...
update-initramfs: Generating /boot/initrd.img-3.13.0-65-generic
gzip: stdout: No space left on device
E: mkinitramfs failure cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-3.13.0-65-generic with 1.
dpkg: error processing package initramfs-tools (--configure):
подпроцесс установлен сценарий post-installation возвратил код ошибки 1
При обработке следующих пакетов произошли ошибки:
linux-image-extra-3.13.0-65-generic
linux-image-generic
linux-generic
linux-generic-lts-raring
linux-image-generic-lts-raring
initramfs-tools
Log ended: 2015-09-28 14:45:09
```
