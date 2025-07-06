---
title: Переход на Systemd-boot на Manjaro
date: 2025-06-24
description: Переход на Systemd-boot на Manjaro.
draft:  false
categories:
  -  linux
tags:
  - systemd-boot
---

### Подготовка к переходу

#### Zotac (Тестовый стенд)

Используется UEFI, так как после перепрошивки UEFI нормально заработал. Разметка на диске GPT. На Zotac используется Systemd-boot. Директория `/boot` является одновременно и EFI разделом и boot разделом.

Каталог `/boot` имеет атрибут EFI раздела. Примонтирован в отдельный раздел. Каталог `/efi` отсутствует. Размер каталога в 1Гб достаточен что бы разместить там  множество ядер. Это является условием возможности  использования Systemd-boot.

```bash
user@lan-zotac /boot> ls -la /boot/ /efi/
ls: cannot access '/efi/': No such file or directory
/boot/:
total 29508
drwxr-xr-x 5 root root     4096 Jan  1  1970 ./
drwxr-xr-x 1 root root      142 Jun 23 17:35 ../
drwxr-xr-x 6 root root     4096 Jun 23 14:30 EFI/
drwxr-xr-x 6 root root     4096 Jun 23 14:49 grub/
-rwxr-xr-x 1 root root 13286400 May 12 20:56 intel-ucode.img*
drwxr-xr-x 4 root root     4096 Jun 24 00:30 loader/
-rwxr-xr-x 1 root root 16908800 Jun 23 06:34 vmlinuz-linux-zen*
```

Команда установки Systemd-boot:

```bash
sudo bootctl list
sudo bootctl install
```

Команда установки Grub:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo grub-install --force --recheck --efi-directory=/boot/ --boot-directory=/boot/ --debug /dev/sda
```

На Zotac каталог нужно переименовать в `/efi`, настроить монтирования и переустановить загрузчики.

#### Lan-lucky (Основной ПК)

На Lan-lucky имеются и каталог `/boot` и `/efi`. Каталог `/efi` примонтирован в отдельный раздел.

```bash
user@lan-lucky ~> ls -la /boot/ /efi/
/boot/:
total 391244
drwxr-xr-x 1 root root       522 июн 23 17:09 ./
drwxr-xr-x 1 root root       204 июн 23 17:50 ../
drwxr-xr-x 1 root root       152 июн 23 13:04 grub/
-rw------- 1 root root 155054155 июн 23 12:56 initramfs-6.12-x86_64-fallback.img
-rw------- 1 root root  16960599 июн 23 12:56 initramfs-6.12-x86_64.img
-rw------- 1 root root 154173909 июн 20 22:12 initramfs-6.15-x86_64-fallback.img
-rw------- 1 root root  15962573 июн 20 22:11 initramfs-6.15-x86_64.img
-rw------- 1 root root  15479653 мая 12  2024 initramfs-linux.img
-rw-r--r-- 1 root root  13286400 мая 12 20:56 intel-ucode.img
-rw-r--r-- 1 root root        22 июн 19 18:49 linux612-x86_64.kver
-rw-r--r-- 1 root root        21 июн 19 18:49 linux615-x86_64.kver
drwxr-xr-x 1 root root        22 янв  5 01:04 memtest86+/
-rw-r--r-- 1 root root       276 июн 23 17:09 refind_linux.conf
-rw-r--r-- 1 root root  13890048 июн 23 12:56 vmlinuz-6.12-x86_64
-rw-r--r-- 1 root root  15790592 июн 20 12:17 vmlinuz-6.15-x86_64

/efi/:
total 2
drwxr-xr-x 4 root root 512 янв  1  1970 ./
drwxr-xr-x 1 root root 204 июн 23 17:50 ../
drwxr-xr-x 8 root root 512 июн 23 17:18 EFI/
drwxr-xr-x 4 root root 512 июн 23 12:53 loader/
```

Команда установки  Grub:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo grub-install --efi-directory=/boot --recheck --debug --bootloader-id=Manjaro --target=x86_64-efi /dev/nvme0n1
```

Необходимо объединить `/boot` и `/efi`, увеличить полученный раздел до 1Тб, настроить монтирования и установить Systemd-boot .


#### Summary

Надо учесть что на Zotac установлена система через `archinstall`. То есть считаем что установка канонически правильная. Его схема взята за образец и дальше будем ориентироваться на неё.


### Проблемы при переходе

#### Первая попытка

При первой попытке перезагрузке я столкнулся с тем что не смог загрузиться.
Начал восстанавливать загрузку через LiveCD Arch Linux.

При генерации в chroot через mkinitcpio `sudo mkinitcpio -P` мне выдавалось предупреждение:

```bash
==> WARNING: Note: no cmdline files were found and --cmdline is not set!
==> WARNING: Reusing current kernel cmdline from /proc/cmdline
```

Предупреждение это не строгая ошибка, поэтому я не предавал ей значения. Как оказалось очень зря!

Это то почему я не могу загрузиться после восстановления из LiveCD. В параметры ядра записывались данные из параметров ядра LiveCD. В процессе запуска реального хоста применялись не правильные параметры. Загрузка заканчивалась не удачей.

Структуру параметров ядра я смотрел на своём тестовом стенде который уже работал на Arch Linux:

```bash
user@lan-zotac /e/mkinitcpio.d> cat /proc/cmdline
root=PARTUUID=2aa6782c-15f0-4d33-9f73-b0eda539aca0 rootflags=subvol=@ rw rootfstype=btrfs
```

Добавил параметры ядра в файл `/etc/kernel/cmdline`. На моём хосте его почему то не было(!).

```bash
user@lan-lucky ~> cat /etc/kernel/cmdline
root=UUID=93bda743-9226-4764-9eaa-9478d379f9e8 rw rootflags=subvol=@ quiet
```

Здесь укажите свои данные. UUID раздела можно посмотреть командой `lsblk -f`

В итоге при генерации в chroot через mkinitcpio нужно все таки использовать `--cmdline` что бы пробросить правильные параметры для ядра.

#### Проблема с Grub

В Manjaro установка ядер завязано на Grub. Поэтому от Grub полностью избавиться не получится. Удалить его не получится. Нужно переходить на Arch.

#### UKI и обычные ядра

UKI гененерируется автоматически через mkinitcpio, и они потом сразу видны Systemd-boot. Для обычных ядер нужно вручную  прописывать конфигурацию.

Пример записи загрузчика в Systemd-boot:

```bash
root@lan-lucky /b/l/entries# cat manjaro.conf
title Manjaro Linux 6.12
linux /vmlinuz-6.12-x86_64
initrd /initramfs-6.12-x86_64.img
options root=UUID=93bda743-9226-4764-9eaa-9478d379f9e8 rw rootflags=subvol=@ quiet
```

```bash
sudo bootctl list
sudo bootctl install
sudo bootctl status
```


### Конфигурация UKI ядер

Я остановился на UKI ядрах. При установке обычных ядер из репозиториев в Arch Linux приезжает файл конфигурации расположенный в `/etc/mkinitcpio.d/`. Его надо отредактировать что бы отключить генерацию initrd обычных ядер и включить генерацию UKI ядер.

Произведем это на примере linux-lts ядра:

```bash
user@lan-lucky /e/mkinitcpio.d> sudo cat /etc/mkinitcpio.d/linux-lts.preset
# mkinitcpio preset file for the 'linux-lts' package

#ALL_config="/etc/mkinitcpio.conf"
ALL_kver="/boot/vmlinuz-linux-lts"

PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-linux-lts.img"
default_uki="/boot/EFI/Linux/arch-linux-lts.efi"
default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-linux-lts-fallback.img"
fallback_uki="/boot/EFI/Linux/arch-linux-lts-fallback.efi"
fallback_options="-S autodetect"
```

```bash
sudo mkinitcpio -P
```

#### Systemd-boot-manager

Для управления записями загрузчика нужен пакет systemd-boot-manager. Неприятный эффект, он затирает вручную созданные записи. С ним надо разбираться отдельно.
