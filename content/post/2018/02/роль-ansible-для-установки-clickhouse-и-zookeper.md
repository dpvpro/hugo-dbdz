---
title: "Роль Ansible для установки Clickhouse и Zookeper"
date: "2018-02-05"
categories: 
  - "linux"
tags: 
  - "ansible role"
  - "clickhouse"
  - "zookeper"
---

<!--more-->

Копируем роль:

`git clone https://github.com/dpvpro/ansible-clickhouse-dp`

Настраиваем хосты в `ansible_hosts`:

```bash
[clickhouse]
host0.example.org ansible_host=host0.example.org ansible_user=root zookeeper_id=1
host1.example.org ansible_host=host1.example.org ansible_user=root zookeeper_id=2
host2.example.org ansible_host=host2.example.org ansible_user=root zookeeper_id=3
host3.example.org ansible_host=host3.example.org ansible_user=root zookeeper_id=4
```

Запускаем роль командой `ansible-playbook -i ansible_hosts click.yaml`.

Далее заходим на любой хост с Clickhouse и запускаем `clickhouse-client`.

Запускаем команды для заведения БД, таблицы и тестовых данных:

```sql
CREATE DATABASE test;

CREATE TABLE test.Migrations ( date Date DEFAULT toDate(now()), id UInt64, time UInt64) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/test/Migrations', '{replica}', date, (id, time), 8192);

INSERT INTO test.Migrations (date) VALUES (689);
INSERT INTO test.Migrations (date) VALUES (6896);
INSERT INTO test.Migrations (date) VALUES (6898);
INSERT INTO test.Migrations (date) VALUES (68999);
INSERT INTO test.Migrations (date) VALUES (689345);

SELECT * from test.Migrations
```

На всех остальных нодах входящих в кластер, создаем только БД и таблицу:

```sql
CREATE DATABASE test;
CREATE TABLE test.Migrations ( date Date DEFAULT toDate(now()), id UInt64, time UInt64) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/test/Migrations', '{replica}', date, (id, time), 8192);
SELECT * from test.Migrations

```

Должны быть выведены реплицированные данные, т.е. задача выполнена.

Плейбук был использован в докладе https://habr.com/ru/articles/484640/