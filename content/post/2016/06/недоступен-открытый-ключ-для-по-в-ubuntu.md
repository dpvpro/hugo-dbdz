---
title: "Недоступен открытый ключ для ПО в Ubuntu"
date: 2016-06-26
lastmod: 2026-04-18
categories:
  - "linux"
---
<!--more-->

В консоли, при обновлении системы, писало множество ошибок:

```bash
W: Ошибка GPG: http://apt.nylas.com trusty InRelease:
Следующие подписи не могут быть проверены, так как недоступен открытый ключ:
NO_PUBKEY 38FC6E967D0ACF4A
```

Устанавливаем необходимые ключи:

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 38FC6E967D0ACF4A
```

Источник - [Cannot solve GPG error](http://askubuntu.com/questions/511736/cannot-solve-gpg-error)
