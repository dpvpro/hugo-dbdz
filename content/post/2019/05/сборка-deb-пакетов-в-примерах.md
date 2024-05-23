---
title: "Сборка deb пакетов в Debian на примере Openvswitch"
date: "2019-05-22"
categories:
  - "linux"
  - "opensource"
tags:
  - "debian package"
  - "openvswitch"
---

Возникла задача собрать deb пакеты Openvswitch на базе Debian 9.8. Сейчас я часто делаю подобные сборки ПО под определенную кодовую базу, поэтому данный пост я делаю в качестве заметки на будущее.

<!--more-->

### Вариант #1 (Original)

Качаем исходники:

`git clone https://github.com/openvswitch/ovs.git`

Переходим в каталог:

`cd ovs`

Выбираем ветку:

`git checkout branch-2.10`

Проверяем зависимости:

`dpkg-checkbuilddeps`

Если выдает что чего то не хватает, то устанавливаем их командой:

`apt install ... `

Сама сборка:

`fakeroot debian/rules binary`

Чистка сборочного окружения ( опционально ):

`fakeroot debian/rules clean`

Если почему то не идет сборка пакетов в результате ваших экспериментов сборкой, а чистка не помогает, то попробуйте сброс ветки:

`git reset --hard`

Если все пройдет хорошо, собранные пакеты будут в папке на уровень выше.

### Вариант #2 (Debian Mainters)

Качаем исходники:

`https://salsa.debian.org/openstack-team/third-party/openvswitch`

Переходим в каталог:

`cd openvswitch`

Просматриваем тэги:

`git tag`

Создаем ветку из тэга и переходим в неё:

`git checkout -b v2.10.0`

Проверяем зависимости:

`dpkg-checkbuilddeps`

Если выдает что чего то не хватает, то устанавливаем их командой:

`apt install ... `

Сама сборка:

`dpkg-buildpackage -B`

Чистка сборочного окружения ( опционально ):

`fakeroot debian/rules clean`

Если почему то не идет сборка пакетов в результате ваших экспериментов сборкой, а чистка не помогает, то попробуйте сброс ветки:

`git reset --hard`

Если все пройдет хорошо, собранные пакеты будут в папке на уровень выше.

##### Использованная литература:

[https://wiki.debian.org/ru/DebianBuildPackages](https://wiki.debian.org/ru/DebianBuildPackages)

[https://www.debian.org/doc/manuals/maint-guide/build.ru.html](https://www.debian.org/doc/manuals/maint-guide/build.ru.html)
