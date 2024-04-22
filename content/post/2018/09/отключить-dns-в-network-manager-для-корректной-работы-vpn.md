---
title: "Отключить dnsmask в Network Manager для корректной работы VPN"
date: "2018-09-24"
categories: 
  - "linux"
  - "network"
tags: 
  - "dns"
  - "network manager"
  - "vpn"
---

После настройки openvpn клиента, блокировки Роскомнадзора все равно срабатывали.
Выяснилось что менеджер сетевых соединений Network Manager в Ubuntu использует свой встроенный dns.

<!--more-->

Из-за этого не работали dns сервера, которые я прописывал. Для отключение этой функции, воспользуемся советом со **Stackoverflow**:

`sudo vim /etc/NetworkManager/NetworkManager.conf`

В файле закоментировать строку:

`#dns=dnsmasq`

Сделать перезапуск Network Manager

`sudo systemctl restart NetworkManager`
