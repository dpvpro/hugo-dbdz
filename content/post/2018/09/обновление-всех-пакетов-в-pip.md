---
title: "Обновление всех пакетов в pip"
date: "2018-09-24"
categories: 
  - "python"
tags: 
  - "pip update"
---
<!--more-->
`pip | pip3` означает что запускаем либо pip, либо pip3.

Просмотр устаревших пакетов.

`pip | pip3 list --outdated --format=columns`

Для обновления следует поставить вспомогательный пакет **pipdate**:

`sudo -H pip | pip3 install pipdate`

Далее обновляем пакеты командой:

`sudo -H pipdate | pipdate3`
