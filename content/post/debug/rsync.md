+++
title = 'Rsync'
description = "Резервное копирование в Linux"
date = 2024-04-16T21:12:00+03:00
+++
#### Резервное копирование

`sudo rsync --delete -ravn --stats /media/veracrypt1/ /media/veracrypt2/`

#### Синхронизация 2-х директорий

Используется для синхронизации музыкальной коллекции, когда в двух местах используются одинаковая структура, но в одном месте были изменены теги, без изменения даты файлов. 

`rsync -rvan --delete --stats --progress -I /run/user/1000/gvfs/smb-share\:server\=sev_cprodv\,share\=music/ /media/dp/BZZZ/music`

`-n` - пробный запуск. без реального копирования/удаления.

`-a` - архивная копия. сохраняет время, размер, права оригинального файла

`--delete` - удаляет файлы которых нет на источнике

`--stats` - дополнительная статистика

`--progress` - выводит прогресс действия

`-I` - самое важная опция. отключает быструю проверку rsync, которая используется по умолчанию: при совпадении размера и времени файл пропускается.

Слэш в конце пути источника говорит о том, что НЕ нужно создавать дополнительную каталог в каталоге назначения (т.е копируется содержимое).
Если слэша нет в конце пути источника, то нужно создавать копируемый каталог (т.е копируется каталог)

-------------------------------------------------------

Есть папка с небольшим набором файлов. Есть другая папка с большим набором файлов. Первый набор входит во второй. Во втором наборе есть более новые версии файлов с теми же самыми именами что и в первом наборе. Задача синхронизировать версии файлов в первом наборе, взяв их из второго, и не трогая остальные.

`rsync -tv --existing /share/stand/Vis/* /media/denpro/DEPOT1/Sertification2016/Repos2016/disks/skbv00033-02-visual/tnt-scylla/`

#### Резервное копирование всей системы

Чтобы создать бэкап всей системы, хватит команды:

tar.bz2:

`sudo tar -cvjpf /mnt/matrix2/system.tar.bz2 --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/media --exclude=/nfs --exclude=/run --exclude=/tmp --exclude=.cache --exclude=/home /`

tar:

`sudo tar -cvpf /mnt/matrix2/system.tar --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/media --exclude=/nfs --exclude=/run --exclude=/tmp --exclude=.cache --exclude=/home /`

Что, собственно, в ней заключено? 
С правами суперпользователя (sudo) создаём тарбол (tar с ключём c) и архивируем его архиватором bz2(ключ j).
Ключ p для сохранения атрибутов и прав файлов. 
При этом с помощью ключа --exclude исключаем из архива системные директории и файлы устройств и, конечно же, сам архив (чтобы он рекурсивно не начал паковаться сам в себя).
В итоге, получаем в корне наш полный архив системы в файле system.tar.

Как его потом развернуть?
Ну, во-первых, нужна будет всё-таки работающая система. Например, можно провести «чистую» установку (или же загрузиться с LiveCD).
Будем считать, что у нас есть работающая система, в которой мы хотим развернуть наш архив. Хватит тоже одной команды:

`tar xvpfz /mnt/matrix2/system.tar.bz2 -C /`


#### Squashfs

Устанавливаем `squashfs-tools`:

`sudo apt-get install squashfs-tools`
`sudo mksquashfs / /media/dp/BEGEMOT/ubuntu.sqfs -no-duplicates -ef /home/dp/exclude.txt`

exclude.txt – файл с исключениями. В отдельной строке файла содержится директория или файл исключения.

#### Пример копирование файла на подключенное хранилище

`cp /lib/firmware/iwlwifi-2030-6.ucode /run/user/dp/gvfs/smb-share\:server\=192.168.1.100\,share\=c/transmission/`

#### DD clone

Упаковка:

`sudo dd if=/dev/sda1 bs=8096 conv=noerror | gzip -9cf > sda1.dd-image.gz`

Распаковка:

`sudo gunzip -cd sda1.dd-image.gz | dd of=/dev/sda1 bs=8096` 

Для задействования всех ядер процессора нужно использовать утилиту `pigz`

`sudo dd if=/dev/sda bs=8096 conv=noerror | pigz -3 > /run/media/manjaro/RESERVED2/sda.dd-image.gz`

`sudo pigz -dc sda1.dd-image.gz | dd of=/dev/sda1 bs=8096`

#### Резервная копия HDD to HHD для ExFat

##### Прежний метод копирования tracey (deprecated):

`time rsync -vrltDn --whole-file --info=progress2 /mnt/RESERVED4/tracey /mnt/MATRIX1/tracey` 

##### Новый метод копирования tracey:

Исходим из того что MATRIX1 основной диск, MATRIX2 запасной:

`time rsync -vrltDn --no-whole-file --inplace --ignore-times --info=progress2 /mnt/RESERVED4/tracey /mnt/MATRIX1/tracey`

`time rsync -vrltDn --no-whole-file --inplace --ignore-times --info=progress2 /mnt/RESERVED4/tracey /mnt/MATRIX2/tracey`

Копирование всего остального

`time rsync -vrltDn --delete --info=progress2 /mnt/MATRIX1/ /mnt/MATRIX2/`


#### Резервная копия HDD to HHD для Ext4/Btrfs

`time sudo rsync -axHAXSvn --delete --info=progress2 /mnt/bublick/ /run/media/dp/test/`

У VeraCrypt снять опцию "Оставлять время изменения контейнера неизменным" что бы у контейнера tracey менялась дата изменения и он входил в набор синхронизируемых файлов. 

`time rsync -axHAXSvn --delete --info=progress2 --exclude={"/tmp","/proc","/dev","/sys","/run",".cache","/mnt"} / /mnt/vault/oldsys/`
