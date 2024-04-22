---
title: 'Ошибка обновления Opera "Sub-process /usr/bin/dpkg returned an error code (1)"'
date: "2016-06-14"
categories: 
  - "linux"
tags: 
  - "linux"
  - "opera"
  - "ubuntu"
  - "ошибка обновления"
---

<!--more-->

После установки браузера Opera на Ubuntu, при последующем обновлении, возникает ошибка `Sub-process /usr/bin/dpkg returned an error code (1)`.

```bash
22:25:29 [user@host ~]$ sudo apt upgrade 
Чтение списков пакетов… Готово
Построение дерева зависимостей       
Чтение информации о состоянии… Готово
Расчёт обновлений…Готово
Пакеты, которые будут обновлены:
  opera-stable syncthing-gtk youtube-dl
обновлено 3, установлено 0 новых пакетов, для удаления отмечено 0 пакетов, и 0 пакетов не обновлено.
Необходимо скачать 50,4 MБ архивов.
После данной операции, объём занятого дискового пространства возрастёт на 21,5 kB.
Хотите продолжить? [Д/н] y
Получено:1 http://ppa.launchpad.net/nilarimogard/webupd8/ubuntu/ trusty/main syncthing-gtk all 0.9.0.3-1~webupd8~trusty0 [370 kB]
Получено:2 http://ppa.launchpad.net/nilarimogard/webupd8/ubuntu/ trusty/main youtube-dl all 2016.06.14-1~webupd8~trusty1 [733 kB]
Получено 50,4 MБ за 1мин 52с (447 kБ/c)                                        
Предварительная настройка пакетов ...
(Чтение базы данных … на данный момент установлено 978819 файлов и каталогов.)
Preparing to unpack …/opera-stable_38.0.2220.31_amd64.deb ...
Unpacking opera-stable (38.0.2220.31) over (38.0.2220.29) ...
Preparing to unpack …/syncthing-gtk_0.9.0.3-1~webupd8~trusty0_all.deb ...
Unpacking syncthing-gtk (0.9.0.3-1~webupd8~trusty0) over (0.9.0.2-1.01~eugenesan~trusty) ...
Preparing to unpack …/youtube-dl_2016.06.14-1~webupd8~trusty1_all.deb ...
Unpacking youtube-dl (2016.06.14-1~webupd8~trusty1) over (2016.06.12-1~webupd8~trusty1) ...
Processing triggers for shared-mime-info (1.2-0ubuntu3) ...
Processing triggers for hicolor-icon-theme (0.13-1) ...
Processing triggers for menu (2.1.46ubuntu1) ...
Processing triggers for mime-support (3.54ubuntu1.1) ...
Processing triggers for bamfdaemon (0.5.1+14.04.20140409-0ubuntu1) ...
Rebuilding /usr/share/applications/bamf-2.index...
Processing triggers for gnome-menus (3.10.1-0ubuntu2) ...
Processing triggers for desktop-file-utils (0.22-1ubuntu1) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Настраивается пакет opera-stable (38.0.2220.31) …
gpg: ресурс блока `/etc/apt/trusted.gpg.d/webupd8team-brackets.gpg': нехватка ресурсов
dpkg: error processing package opera-stable (--configure):
 подпроцесс установлен сценарий post-installation возвратил код ошибки 2
Настраивается пакет syncthing-gtk (0.9.0.3-1~webupd8~trusty0) …
Настраивается пакет youtube-dl (2016.06.14-1~webupd8~trusty1) …
При обработке следующих пакетов произошли ошибки:
 opera-stable
E: Sub-process /usr/bin/dpkg returned an error code (1)

```

Для исправления идем в директорию `/var/lib/dpkg/info` и удаляем все что относится к пакету oper'ы и обновляемся.

```bash
sudo rm /var/lib/dpkg/info/opera-stable.*
sudo apt-get -f install
```
