---
title: Памятка по использование утилиты Rsync
description: Памятка по использование утилиты Rsync. Резервное копирование в Linux.
date: 2024-04-16
lastmod: 2025-05-16
categories:
  - linux
tags:
  - rsync linux
  - rsync примеры
  - rsync linux примеры
  - rsync linux копирование файлов
  - rsync файлов
  - копирование rsync
  - клонирование dd
  - клонирование диска linux
  - клонирование диска dd linux
  - dd clone
---

### Резервное копирование (зеркалирование)

`sudo rsync --delete -ravn --stats /media/veracrypt1/ /media/veracrypt2/`

<!--more-->

### Синхронизация 2-х директорий

Используется для синхронизации музыкальной коллекции, когда в двух местах используются одинаковая структура, но в одном месте были изменены теги, без изменения даты файлов.

`rsync -rvan --delete --stats --progress -I /run/user/dp/music/ /media/dp/bzzz/music/`

`-n` - пробный запуск. без реального копирования/удаления.

`-a` - архивная копия. сохраняет время, размер, права оригинального файла

`--delete` - удаляет файлы которых нет на источнике

`--stats` - дополнительная статистика

`--progress` - выводит прогресс действия

`-I` - самое важная опция. отключает быструю проверку rsync, которая используется по умолчанию: при совпадении размера и времени файл пропускается.

Слэш в конце пути источника говорит о том, что копируется содержимое каталога источника, то есть, что бы было легче запомнить или понять, то что располагается за последним слешем.

Если слэша нет в конце пути источника, то копируется сам каталог источник в каталог назначения.

### Синхронизация набора файлов

Есть папка с небольшим набором файлов. Есть другая папка с большим набором файлов. Первый набор входит во второй. Во втором наборе есть более новые версии файлов с теми же самыми именами что и в первом наборе. Задача синхронизировать версии файлов в первом наборе, взяв их из второго, и не трогая остальные.

`rsync -tv --existing /share/stand/vis/* /media/denpro/depot1/sertification2016/repos2016/disks/skbv00033-02-visual/tnt-scylla/`

### Резервное копирование всей системы

Чтобы создать бэкап всей системы, хватит команды:

*tar.bz2:*

`sudo tar -cvjpf /mnt/matrix2/system.tar.bz2 --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/media --exclude=/nfs --exclude=/run --exclude=/tmp --exclude=.cache --exclude=/home --exclude=/var/tmp /`

*tar:*

`sudo tar -cvpf /mnt/matrix2/system.tar --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/media --exclude=/nfs --exclude=/run --exclude=/tmp --exclude=.cache --exclude=/home --exclude=/var/tmp /`

Что, собственно, в ней заключено?

С правами суперпользователя (`sudo`) создаём тарбол (`tar` с ключём `-c`) и архивируем его архиватором `bz2` (ключ `-j`).

Ключ `-p` для сохранения атрибутов и прав файлов.

При этом с помощью ключа `--exclude` исключаем из архива системные директории и файлы устройств и, конечно же, сам архив (чтобы он рекурсивно не начал паковаться сам в себя).

В итоге, получаем в корне наш полный архив системы в файле `system.tar`.

#### Как его потом развернуть?

Ну, во-первых, нужна будет всё-таки работающая система. Например, можно провести «чистую» установку (или же загрузиться с *LiveCD*).

Будем считать, что у нас есть работающая система, в которой мы хотим развернуть наш архив. Хватит тоже одной команды:

`tar xvpfz /mnt/matrix2/system.tar.bz2 -C /`

### DD клонирование

#### Классический способ, со сжатием

Архивация:

`sudo dd if=/dev/sda1 bs=8096 conv=noerror | gzip -9cf > sda1.dd-image.gz`

Восстановление:

`sudo gunzip -cd sda1.dd-image.gz | dd of=/dev/sda1 bs=8096`

Далее команды будут идти в такой же последовательности - архивация, восстановление.

#### Классический способ, со сжатием, и утилитой "pigz"

Для задействования всех ядер процессора можно использовать утилиту `pigz`. Это может существенно ускорить процесс создания архивов. Но и жарить процессор будет по полной. Можно использовать для стресс тестов)

`sudo dd if=/dev/sda bs=8M conv=noerror status=progress | pigz -0 > /mnt/apriel/lan-zotac.archlinux.sda.dd.gz`

`sudo pigz -dc /mnt/apriel/lan-zotac.archlinux.sda.dd.gz | dd of=/dev/sda bs=8M status=progress`

#### Классический способ, без сжатия:

Иногда выгодней выбрать способ без сжатия. Время создания будет ограничено пропускной способностью жесткого диска.

`dd if=/dev/sda of=/mnt/apriel/lan-zotac.windows11.sda.dd bs=8M conv=noerror status=progress`

`dd if=/mnt/apriel/lan-zotac.windows11.sda.dd of=/dev/sda bs=8M conv=noerror status=progress`

#### C использованием опции "sparse" утилиты "dd":

Это самый интересный режим, потому как позволят значительно уменьшить время клонирования больших разделов или дисков. При использовании опции `sparse` будут использованны только сектора с данными с диск источника, пустое пространство будет исключено.

Данный режим можно использовать при переносе системы на другой диск, при условии что новый диск равен или больше старого диска.

Другой сценарий который я использую на тестовом стенде. По умолчанию на тестовом стенде установлен *Arch Linux*. Изредка требуется использование *Windows 11*. Я сделал полный образ диска с *Arch Linux*. После установил на этот же диск *Windows 11*, затерев *Linux*. Выполнил все настройки, и произвел необходимые операции под *Windows*. Сделал еще раз полный образ диска с *Windows*.

Теперь переключение, при необходимости, между системами занимает 20 минут.

`dd if=/dev/sda of=/mnt/apriel/lan-zotac.archlinux.sda.sparse.dd bs=8M conv=sparse,noerror status=progress`

`dd if=/mnt/apriel/lan-zotac.archlinux.sda.sparse.dd of=/dev/sda bs=8M status=progress`

Этот метод актуален если файловая система где создается результирующий файл поддерживает работу со `sparse` файлами.

<!---
Info was taken from https://www.baeldung.com/linux/clone-space-in-use-from-disk
-->

### Резервная копия HDD to HHD для ExFat

#### Прежний метод копирования tracey (deprecated):

`time rsync -vrltDn --whole-file --info=progress2 /mnt/reserved4/tracey /mnt/matrix1/tracey`

#### Новый метод копирования tracey (deprecated):

Исходим из того что `matrix1` основной диск, `matrix2` запасной:

`time rsync -vrltDn --no-whole-file --inplace --ignore-times --info=progress2 /mnt/reserved4/tracey /mnt/matrix1/tracey`

`time rsync -vrltDn --no-whole-file --inplace --ignore-times --info=progress2 /mnt/reserved4/tracey /mnt/matrix2/tracey`

Копирование всего остального

`time rsync -vrltDn --delete --info=progress2 /mnt/matrix1/ /mnt/matrix2/`

### Резервная копия HDD to HHD для Ext4/Btrfs

`time sudo ionice -c idle rsync -axHAXSvn --delete --info=progress2 /mnt/bublick/ /run/media/dp/matrix2/`

Ключ `-n` означает тестовый запуск, без выполнения реальных действий. Необходимо убрать его при реальном использовании.

> Не забыть у VeraCrypt снять опцию "Оставлять время изменения контейнера неизменным" что бы у зашифрованного контейнера менялась дата изменения и он входил в набор синхронизируемых файлов.

> Так как я перешел на шифрование контейнера с использованием LUKS (dm-crypt), то проблема с датами как у VeraCrypt не актуальна. Контейнер корректно помечается временем при записи в него.

### Архивация скрытых файлов и директорий домашнего каталога

`tar cvf lan-lucky.home.tar --exclude=.cache ~/.[^.]*`

### Аналог scp для rsync c ограничением скорости загрузки, поддержкой докачки, и соединения по не стандартному порту

`rsync -v --bwlimit=5M --partial --progress -e 'ssh -p 10531' debian-work.qcow2 dp@185.51.XX.XX:/mnt/bublick/soft/vm/`

`rsync -avn --delete --progress -e 'ssh -p 10531' ~/Code/ dp@83.243.xx.xxx:~/Code/`
