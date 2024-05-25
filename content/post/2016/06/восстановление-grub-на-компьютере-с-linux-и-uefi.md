---
title: Восстановление Grub на компьютере с Linux, UEFI, GPT
date: 2016-06-11
description: Восстановление Grub на компьютере с Linux, UEFI, GPT
categories: 
  - linux
tags: 
  - grub uefi
  - grub efi
  - восстановление grub uefi
  - восстановление grub efi
  - восстановить grub uefi
  - grub uefi установка
  - установить grub uefi
  - linux uefi grub
  - установка grub
  - восстановить grub
  - восстановление grub
---

При попытке сделать загрузочную флэшку, сделал процедуру установки загрузчка *Grub* на флэшку.
В результате поменялась информация и в загрузочном меню *UEFI*. Видно что то не учел.

Система перестал грузиться с основного жесткого диск, только с флэшки. Для исправления загрузки не подходил обычный способ восстановления загрузчика.

Для решения проблемы используем ссылку [Ubuntu 14.04 UEFI boot partition and GRUB reinstall problem](http://ubuntuforums.org/showthread.php?t=2223856&page=3).
Смотреть сразу последний пост, первый описанный вариант.

*Приведу его здесь в пошаговом варианте*:

<!--more-->

Загружаемся с Live CD и определяем, используется ли UEFI при загрузке с Live CD:

```bash
[ -d /sys/firmware/efi ] && echo "EFI boot on HDD" || echo "Legacy boot on HDD"
```

В нашем случае должно быть "EFI boot on HDD".

Устанавливаем `efibootmgr`:

```bash
sudo apt-get install efibootmgr
```

Проверяем, то что есть EFI раздел и диск с GPT:

```bash
sudo gdisk -l /dev/sda
```

Монтируем файловую систему. В данному случае, `root` у нас в /dev/sda2, `efi` раздел в /dev/sda1:

```bash
sudo mkdir -p /mnt/system
sudo mount /dev/sda2 /mnt/system
sudo mount /dev/sda1 /mnt/system/efi
```

Прописываем пункт в меню UEFI:

`efibootmgr -c -d /dev/sda -p 1 -w -L Ubuntu`

При необходимости удаляем пункты из меню:

```bash
efibootmgr -v
efibootmgr -B -b [number item]
```

Устанавливаем `grub-efi`:

```bash
sudo apt-get install grub-efi-amd64
```

Устанавливаем загрузчик на диск:

```bash
sudo  grub-install --root-directory=/mnt/system --boot-directory=/mnt/system/boot --efi-directory=/mnt/system/efi --bootloader-id=Ubuntu --target=x86_64-efi --recheck  --debug /dev/sda
```

Далее прописано обновить меню grub. В оригинале написано не правильно. Привожу правильную команду:

```bash
sudo grub-mkconfig -o /mnt/system/boot/grub/grub.cfg
```

