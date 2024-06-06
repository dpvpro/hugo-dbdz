---
title: "OpenVPN на сервере со множеством сетевых интерфейсов"
date: "2021-01-30"
categories:
  - "network"
tags:
  - "openvpn multihome"
---
<!--more-->
При настройке OpenVPN сервера в контейнере существует два сетевых интерфейса:

```bash
eth0 == Внешний адрес : X.X.X.X
eth1 == Внутренний адрес: 192.168.0.100 255.255.240.0
tun0 == 10.8.0.1 255.255.255.255

```

Для корректной работы, нужно в конфиг прописать параметр `multihome`, иначе будет иметь значение порядок сетевых интерфейсов.

Без этого параметра, *OpenVPN* сервер выбирает какой либо сетевой интерфейс интерфейсом по умолчанию.
При получения пакета из другого сетевого интерфейса, отправляет ответ в интерфейс по умолчанию.

С параметром `multihome` с какого интерфейса пришел пакет туда и уйдет ответ.

Вроде бы, так должно быть, с точки зрения здравого смысла, но оказывается, для этого нужен отдельный параметр.

```bash
port 1194
proto udp
dev tun
multihome
tls-server
ca keys/ca.crt
cert keys/lup64.crt
key keys/lup64.key
dh keys/dh1024.pem
mode server
ifconfig 10.8.0.1 10.8.0.2
ifconfig-pool 10.8.0.4 10.8.0.255
route 10.8.0.0 255.255.255.0
keepalive 10 60
inactive 600
comp-lzo
user openvpn
group openvpn
persist-tun
persist-key
verb 3
log-append /var/log/openvpn.log
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 10.8.0.1"
```
