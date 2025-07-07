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
  - восстановление grub
  - восстановить grub uefi
  - восстановить grub-efi
  - восстановить grub
  - grub uefi установка
  - установить grub uefi
  - linux uefi grub
  - установка grub
---

При попытке сделать загрузочную флэшку, сделал процедуру установки загрузчка **Grub** на флэшку. В результате поменялась информация и в загрузочном меню **UEFI**. Видно что то не учел. Система перестал грузиться с основного жесткого диск, только с флэшки. Для исправления загрузки не подходил способ для восстановления загрузчика **BIOS** систем.



<!-- > [Ubuntu 14.04 UEFI boot partition and GRUB reinstall problem](http://ubuntuforums.org/showthread.php?t=2223856&page=3) -->
<!-- > -->
> Данный способ изначально для **Ubuntu Linux**, но должен подойти для любого дистрибутива.
>
> [Оригинальный пост](http://ubuntuforums.org/showthread.php?t=2223856&page=3), последняя страница, первый описанный вариант.

<!--more-->

Загружаемся с **Live CD** и определяем, используется ли **UEFI** при загрузке с **Live CD**:

```bash
[ -d /sys/firmware/efi ] && echo "EFI boot on HDD" || echo "Legacy boot on HDD"
```

В нашем случае должно быть `EFI boot on HDD`.

Устанавливаем `efibootmgr`:

```bash
sudo apt-get install efibootmgr
```

Проверяем, то что есть **EFI** раздел и диск с **GPT**:

```bash
sudo gdisk -l /dev/sda
```

Конечно же вместо `/dev/sda` подставьте ваш диск. У вас он может быть другим.

Монтируем файловую систему. В моём случае, `root` в `/dev/sda2`, `efi` раздел в `/dev/sda1`:

```bash
sudo mkdir -p /mnt
sudo mount /dev/sda2 /mnt
sudo mount /dev/sda1 /mnt/efi
```

Прописываем пункт в меню **UEFI**:

```bash
sudo efibootmgr -c -d /dev/sda -p 1 -w -L Ubuntu
```

При необходимости удаляем пункты из меню:

```bash
sudo efibootmgr -v
sudo efibootmgr -B -b [number item]
```

Устанавливаем `grub-efi`:

```bash
sudo apt-get install grub-efi-amd64
```

Устанавливаем загрузчик на диск:

```bash
sudo grub-install --root-directory=/mnt --boot-directory=/mnt/boot \
--efi-directory=/mnt/efi --target=x86_64-efi --bootloader-id=Ubuntu \
--recheck --debug /dev/sda
```

Далее прописано обновить меню **Grub**. В оригинале как то странно. Привожу рабочий набор команд. Надо попасть в `chroot`, и там уже обновить конфигурацию **Grub**:

```bash
sudo mount --bind /dev /mnt/dev
sudo mount --bind /sys /mnt/sys
sudo mount --bind /proc /mnt/proc
sudo chroot /mnt

grub-mkconfig -o /boot/grub/grub.cfg

# или строка выше или строка ниже

update-grub
```
