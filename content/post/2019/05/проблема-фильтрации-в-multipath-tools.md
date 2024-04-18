---
title: "Проблема фильтрации в multipath-tools"
date: "2019-05-22"
categories: 
  - "linux"
tags: 
  - "iscsi"
  - "multipath-tools"
---
Много времени потрачено на выяснение, почему multipath-tools НЕ фильтрует iscsi подключение. Расследование длилось два дня.
В итоге выяснилось, что при подключении по iscsi, блочное устройство создается в контексте /dev/sd{a,b,c} и т.д.
Как правило, это загрузочные устройства. Это ключевой момент. По умолчанию multipath-tools фильтрует устройства с такими именами.

Для решения нужно добавить в конфиг строчку:

```bash
find_multipaths "no"

```

Полный конфиг multipath.conf:

```bash
defaults {
verbosity 2
user_friendly_names "no"
find_multipaths "no"
}

blacklist {
protocol ".*"
}

blacklist_exceptions {
protocol "scsi:iscsi"
protocol "scsi:fc"
}

```

commit id - 4b062334631084b3622a6f38061329c9cf675e8d
