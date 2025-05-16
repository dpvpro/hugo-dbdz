---
title: "Dtrx - умная распаковка архивов (tar, zip, cpio, rpm, deb, rar)"
date: "2015-09-25"
lastmod: 2025-05-16
categories: 
  - "linux"
tags:
  - "dtrx"
---

Если много работать с различными архивами в Linux, то приходится использовать множество различных утилит и ключей для распаковки этих архивов.

Для комфортной распаковки будем использовать утилиту *Dtrx*. *Dtrx* расшифровывается как *Do The Right Extraction*, т.е. "делай правильную распаковку".

Команда `dtrx` является заменой `tar -zxvf` или `tar -xjf`, и представляет собой единую команду для извлечения архивов различных форматов, включая `tar, zip, rpm, deb, gem, 7z, cpio, rar` и множества других.

По умолчанию, программа извлекает содержимое в отдельную директорию, с таким же именем как и у архива, и обеспечивает права на чтение и запись для пользователя. В числе прочего, поддерживается рекурсивная распаковка - можно распаковать архивы найденные в распаковываемом архиве. Так же утилита будет очень полезна, если надо распаковать сразу много архивов.

<!--more-->

### Установка на Debian/Ubuntu/Linux Mint

`sudo apt-get install dtrx`

### Установка на RHEL/CentOS/Fedora

На RedHat подобных системах программа `dtrx` не доступна в репозиториях по умолчанию. Нужно скачать и установить скрипт в систему, используя следующие команды, от учетной записи *root*.

```bash
wget http://brettcsmith.org/2007/dtrx/dtrx-7.1.tar.gz
tar -xvf dtrx-7.1.tar.gz 
cd dtrx-7.1
python setup.py install --prefix=/usr/local
```

### Синтаксис

`dtrx [ОПЦИЯ] ARCHIVE [АРХИВ ...]`

### Опции

`-r, -recursive` - с этой опцией `dtrx` будет искать архивы внутри архива и извлекать их. 

`--one, --one-entry` - если архив содержит только один файл или только одну папку с именем не совпадающим с именем архива, то `dtrx` у вас спросит, что с ним делать.

При помощи этой опции вы можете заранее задать действия архиватора. Возможные параметры опции:

- `inside` - распаковка файла или папки в каталог с другим именем. Значение по умолчанию.
- `rename` - распаковка в текущую папку, а затем переименовывание распаковывание файла/каталога.
- `here` - распаковка файла/папки в текущую директорию.

`-o, --overwrite` - обычно, `dtrx` не извлекает файлы в папку, которая уже была создана, а подбирает альтернативное имя для директории. 
Если эта опция указана, то `dtrx` не будет использовать имя каталога по умолчанию не смотря ни на что. 

`-f, --flat` - извлечение всего содержимого в одну папку. Эта опция будет полезна, если у вас есть несколько архивов, которые необходимо распаковать в одну папку.
При включение этой опции созданные файлы могут быть перезаписаны.

`-l , -t , --list , --table` - не распаковывать архив, а только вывести его содержимое.

`-m , --metadata` - извлекает мета данные из .deb и .gem архивов вместо их содержимого. 

`-n, --noninteractive` - обычно `dtrx` выдает запросы, при возникновении каких то ситуаций. например, как обрабатывать архив, только с одним файлом или каталогом.
При использовании этой опции, `dtrx` использует настройки по умолчанию.

### Пример использования

```bash
test@sev-lucky:/tmp$ dtrx -l blog-150926.tar.gz var/www/html/ var/www/html/wp-admin/
test@sev-lucky:/tmp$ dtrx blog-150926.tar.gz
blog-150926.tar.gz contains one directory but its name doesn`t match.
Expected: blog-150926
Actual: var/
You can:
* extract the directory _I_nside a new directory named blog-150926
* extract the directory and _R_ename it blog-150926
* extract the directory _H_ere
What do you want to do? (I/r/h) i
blog-150926.tar.gz contains 2 other archive file(s), out of 1482 file(s) total.
You can:
* _A_lways extract included archives during this session
* extract included archives this _O_nce
* choose _N_ot to extract included archives this once
* ne_V_er extract included archives during this session
* _L_ist included archives
What do you want to do? (a/o/N/v/l) n
test@sev-lucky:/tmp$
```
