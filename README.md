Clickhouse
=========

Роль предназначена для установки и создания баз Clickhouse


Role Variables
--------------

Для настроки репликации используются переменные clickhouse_shards и zookeeper_hosts

zookeeper_hosts - заимствована из роли ansible-zookeeper чтобы не конфликтовать.

Пример конфигурации шарда с именем first в котором настроена репликация на 2 хоста host01 и host02: 
```
clickhouse_shards:
  first:
  - { name: "01", replicas: ["host01","host02"]}
```

Пример конфигурации шардирования с именем second в котором настроено разделение на 2 шарда,
которые в свою очередь реплицируются по хостам
```
clickhouse_shards:
  second:
  - { name: "01", replicas: ["host01","host02"]}
  - { name: "02", replicas: ["host03","host04"]}
```

После настройки репликации на уровне конфигурации сервера так же необходимо определить какие таблицы необходимо реплицировать.
Делается это в аргументах команды CREATE TABLE

```
CREATE TABLE default.replicated_visit ( target String,  ga_id String,  campaign_id Int32,  ip String,  referrer String,  datetime DateTime,  type String,  request_id UUID,  partner_id Int32,  manager_id Int32,  date Date) 
ENGINE = ReplicatedMergeTree('/clickhouse/tables/first/replicated_visit', 'test31', date, (request_id, date), 8192);
```
Основное отличие это первые 2 параметра к движку таблицы
1. Ветка в zookeeper где данная таблица отслеживается
   Рекомендуется использовать формат /clickhouse/tables/{shard_id}/{table_id}
2. Имя ноды

