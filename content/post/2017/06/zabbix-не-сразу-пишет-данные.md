---
title: "Zabbix не сразу пишет данные которые ему посылает zabbix_sender"
date: "2017-06-26"
categories: 
  - "linux"
tags: 
  - "import"
  - "zabbix_sender"
---

При проверке ПО на тестовом стенде были проведены исследования производительности.
Результаты хранились в csv файлах, для последующего импорта в Zabbix. При попытке импортировать данные в Zabbix возникала ошибка.

```bash
zabbix_sender -vv -z zabbix.server.host -p 10051 -s "test.node.001" -k "trap.item.001" -o "0.0"
zabbix_sender [24284]: DEBUG: answer [{"response":"success","info":"processed: 0; failed: 1; total: 1; seconds spent: 0.000031"}]
info from server: "processed: 0; failed: 1; total: 1; seconds spent: 0.000031"
```

Тем не менее, после определенного времени, данные импортировались нормально.
После того как начал изучать эту тему более подробно, обратил внимание что Zabbix обновляет данные раз в 10 минут.

После исследования выяснилось что за период обновление информации в БД от процесса `zabbix_server` отвечает
параметр [ConfigFrequency](https://www.zabbix.com/documentation/3.0/manual/appendix/config/zabbix_server) в конфигурационном файле Zabbix.
После выставления значение в 60 секунд и рестарта сервиса, обновление стало таким как нужно.
