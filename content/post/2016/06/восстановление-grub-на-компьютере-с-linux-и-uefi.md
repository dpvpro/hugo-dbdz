---
title: "Восстановление Grub на компьютере с Linux, UEFI, GPT"
date: "2016-06-11"
categories: 
  - "linux"
tags: 
  - "grub"
  - "linux"
  - "recovery"
  - "uefi"
---

При попытке сделать загрузочную флэшку, сделал процедуру установки загрузчки grub на флэшку.
В результате поменялась информация и в загрузочном меню UEFI. Видно что то не учел.

Система перестал грузиться с основного жесткого диск, только с флэшки. Для исправления загрузки не подходил обычный способ восстановления загрузчика.

Для решения проблемы используем ссылку - [Ubuntu 14.04 UEFI boot partition and GRUB reinstall problem](http://ubuntuforums.org/showthread.php?t=2223856&page=3).
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

Монтируем файловую систему. В данному случае, `root` у нас в /dev/sda2, `boot` в /dev/sda1:

```bash
sudo mkdir -p /mnt/system
sudo mount /dev/sda2 /mnt/system
sudo mount /dev/sda1 /mnt/system/boot/efi
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
sudo  grub-install --boot-directory=/mnt/system/boot --bootloader-id=Ubuntu  --target=x86_64-efi --efi-directory=/mnt/system/boot/efi --recheck  --debug /dev/sda
```

Далее прописано обновить меню grub, но у меня оно заканчивается с ошибкой.

```bash
sudo grub-mkconfig -o /mnt/system/boot/efi/EFI/GRUB/grub.cfg
```

Для решения, использовать команду:

```bash
sudo grub-install --root-directory=/mnt/system /dev/sda
```
