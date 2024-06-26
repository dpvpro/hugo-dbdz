---
title: "Проблемы при работе с файловой системой ExFat"
date: "2016-02-07"
categories:
  - "linux"
tags:
  - "exfat mount bug"
---
Переписал на диск с файловой системой `exfat` большой каталог со множеством файлов на ОС Windows 7. При попытке работы с этим каталогом из под Ubuntu диск постоянно отваливался с ошибкой:

`Функция stat завершилась с ошибкой: Конечная точка передачи не подсоединена" ("Transport endpoint is not connected").`

<!--more-->

После проверки внешнего диска командой:

```bash
sudo fsck.exfat /dev/sdc1

```

возникали ошибки:

`ERROR: name is too long.` и `BUG: failed to convert name to UTF-8.`

<!--more-->

После гугления решил вручную обновить драйвер `exfat` - [https://github.com/relan/exfat](https://github.com/relan/exfat).
Проблема ушла, но теперь пропало автоматическое монтирование диска. Монтировать можно только в ручном режиме.

Для решения проблемы автоматического монтирования, нужно установить скомпилированные бинарные программы в правильное место.

Для этого делаем:

```bash
# удаляем скомпилированные программы из /usr/local/sbin
sudo make uninstall
# меняем директорию для установки (после --prefix пусто(!))
./configure --prefix=
# устанавливаем в /sbin
sudo make install
```

Для решения проблемы файлов с длинным именем в пути, я нашел их, просмотрел что там нет ничего важного, и удалил командой:

```bash
find -regextype posix-extended -regex '.{257,}' -delete

```

Что бы просто просмотреть файлы с длинными путями, нужно выполнить команду без параметра `-delete`.
