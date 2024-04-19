---
title: "Cisco AnyConnect Compatible VPN"
date: "2016-11-16"
categories: 
  - "linux"
tags: 
  - "cisco"
  - "vpn"
---

На работе дали возможность пользоваться VPN. В качестве VPN используется Cisco VPN Client. Веб интрефейс Cisco VPN Client ужасен. Так же как и клиент AnyConnect. Под Ubuntu Linux для использования VPN требуется поставить несколько пакетов:

```bash
sudo apt-get install network-manager-openconnect network-manager-openconnect-gnome
```

После этого в NetworkManager появляется возможность настроить соединении AnyConnect VPN.
