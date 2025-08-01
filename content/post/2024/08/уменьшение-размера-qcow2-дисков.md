---
title: Уменьшение размера QCOW2 дисков
date: 2024-08-06T05:39:25+03:00
draft: false
description: При использовании виртуализации KVM и образов виртуальных дисков QCOW2 существует проблема когда диски начинают расходовать излишнее пространство. Оптимизируем это.
categories:
  - linux
tags:
  - qcow2 shrink
  - уменьшение qcow2 дисков
---

При использовании виртуализации KVM и образов виртуальных дисков QCOW2 существует проблема когда диски начинают расходовать излишнее пространство. Особенно это начинает проявляться когда используется функционал снимков (snapshots).

Недостаточно просто освободить место на самой виртуальной машине или удалить снимок виртуальной машины (ВМ). Как только QCOW2 увеличивается в размерах, он никогда не уменьшается сам по себе.

Ниже привожу два независимых способа исправить ситуацию. Данные мероприятия позволяют оптимизировать десятки и сотни гигабайт дискового пространства. Все действия производиться на хостовой ситсеме.

Утилита `virt-sparsify` находится в пакете `guestfs-tools` в Arch Linux и Debian Linux. Утилита `qemu-img` находится в пакете `qemu-img` в Arch Linux и в пакете `qemu-utils` в Debian Linux.

Перечисленные пакеты должен быть установлены в хостовой системе.

<!--more-->

### Virt-sparsify

Чтобы сделать это быстро и безопасно, рекомендуются следующие шаги:

1. При необходимости, освободите все место на диске внутри ВМ .

2. Выключите ВМ.

3. Если ВМ особо ценная, то можно создать резервную копию образа диска, который вы хотите уменьшить:
```bash
cp --sparse=auto debian12_debian12.img debian12_debian12.img.old
```

4. Смотрим информацию о нашем диске:
```bash
[lan-lucky images]# qemu-img info debian12_debian12.img
image: debian12_debian12.img
file format: qcow2
virtual size: 100 GiB (107374182400 bytes)
disk size: 13.1 GiB
cluster_size: 65536
backing file: /var/lib/libvirt/images/debian-VAGRANTSLASH-bookworm64_vagrant_box_image_12.20240503.1_box.img
backing file format: qcow2
Format specific information:
	compat: 1.1
	compression type: zlib
	lazy refcounts: false
	refcount bits: 16
	corrupt: false
	extended l2: false
Child node '/file':
	filename: debian12_debian12.img
	protocol type: file
	file length: 13.1 GiB (14087487488 bytes)
	disk size: 13.1 GiB

```

5. Основное действие. Из каталога, где хранится ваш образ, вызовите команду:

```bash
[lan-lucky images]# virt-sparsify --in-place debian12_debian12.img
[   2.4] Trimming /dev/sda1
[   5.6] Sparsify in-place operation completed with no errors
```

6. Cнова смотрим информацию о нашем диске. Видно что параметр `disk size` уменьшился:

```bash
[lan-lucky images]# qemu-img info debian12_debian12.img
image: debian12_debian12.img
file format: qcow2
virtual size: 100 GiB (107374182400 bytes)
disk size: 7.41 GiB
cluster_size: 65536
backing file: /var/lib/libvirt/images/debian-VAGRANTSLASH-bookworm64_vagrant_box_image_12.20240503.1_box.img
backing file format: qcow2
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
Child node '/file':
    filename: debian12_debian12.img
    protocol type: file
    file length: 13.1 GiB (14087487488 bytes)
    disk size: 7.41 GiB
```

7. После успешного выполнения команды перезапустите виртуальную машину.

8. Можно проверить, что использование дискового пространства снизилось с помощью команды `df -Th`.

### Конвертация

При использовании способа выше видимый размер виртуального диска остаётся прежним. Это видно по полю `file length` в выводе выше.
В системе это можно увидеть с помощью команды:

```bash
[lan-lucky images]# du -hs --apparent-size debian12_debian12.img
14G	debian12_debian12.img
```

Что бы привести видимый размер и реальный к общему знаменателю диск можно сконвертировать:

```bash
qemu-img convert -O qcow2 debian12_debian12.img debian12_debian12.img.new
```

Смотрим информацию о диске после конвертации:

```bash
[lan-lucky images]# qemu-img info debian12_debian12.img.new
image: debian12_debian12.img.new
file format: qcow2
virtual size: 100 GiB (107374182400 bytes)
disk size: 7.74 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
Child node '/file':
    filename: debian12_debian12.img.new
    protocol type: file
    file length: 7.74 GiB (8315994112 bytes)
    disk size: 7.74 GiB
```

```bash
[lan-lucky images]# du -hs --apparent-size debian12_debian12.img.new
7.8G	debian12_debian12.img.new
```

Заменяем старый диск на новый:
```bash
mv -f debian12_debian12.img.new debian12_debian12.img
```

### Саммари

Первый способ относительно быстрый, второй более качественный, но может быть медленней первого и потребовать больше дискового пространства.
