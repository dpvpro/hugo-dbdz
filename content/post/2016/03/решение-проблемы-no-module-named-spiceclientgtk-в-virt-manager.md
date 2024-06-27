---
title: "Решение проблемы «No module named SpiceClientGtk» в virt-manager"
date: "2016-03-21"
categories:
  - "linux"
tags:
  - "kvm libvirt spice"
---
<!--more-->

При попытке подключиться к консоли виртуальной машины через `spice` `virt-manager` отказался подключаться к виртуальной машине и выдает ошибку:

Невозможно отобразить графическую консоль типа `spice`:

`No module named SpiceClientGtk`

Для Ubuntu 16.04 решение сводится к установке необходимых пакетов:

`sudo apt-get install libspice-client-gtk-3.0-4 python-spice-client-gtk`

После установки пакетов все будет работать.

Иногда `virt-manager` выдает ошибку типа:

`Error opening spice console. SpiceClientGTK missing`

Это лечится установкой двух пакетов:

`sudo apt-get install gir1.2-spice-client-gtk-3.0 gir1.2-spice-client-gtk-2.0`
