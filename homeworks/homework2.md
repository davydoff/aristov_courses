# Домашнее задание 2

```
thai=# show server_version;
17.0 (Debian 17.0-1.pgdg120+1)

\conninfo
You are connected to database "thai" as user "dds" on host "nas" (address "192.168.8.2") at port "5003".
```

Session 1: Создаем новую таблицу
```
thai=# create table movies (title text not null, is_watched bool default false);
CREATE TABLE
```

Session 1: Текущий уровень изоляции
```
thai=# show transaction isolation level;
read committed
```

Session 1 & 2: Начинаем транзакцию
```
thai=# begin;
BEGIN
```

Session 1: Добавляем запись
```
thai=*# insert into movies values ('Attack of the Killer Tomatoes', true);
INSERT 0 1
```

SESSION 2: Выбираем все записи
```
thai=*# select * from movies;
 title | is_watched
-------+------------
(0 rows)
```
⚠️ _Новую запись мы не видим, т.к. READ COMMITED позволяет видеть только закоммиченные изменения в параллельных транзакциях._

Session 1: Commit
```
thai=*# commit;
COMMIT
```

SESSION 2: Выбираем все записи
```
thai=*# select * from movies;
             title             | is_watched
-------------------------------+------------
 Attack of the Killer Tomatoes | t
(1 row)
```
⚠️ _Данные в параллельной транзакции закоммичены, поэтому теперь мы видим запись._

SESSION 2: Commit
```
thai=*# commit;
COMMIT
```
---

Session 1 & 2: Начинаем транзакции с уровнем изоляции REPEATABLE READ
```
thai=# begin; set transaction isolation level repeatable read;
BEGIN
SET
thai=*#
```

Session 1: Добавляем новую запись
```
thai=*# insert into movies values ('A Nightmare on Elm Street', false);
INSERT 0 1
```

SESSION 2: Выбираем все записи
```
thai=*# select * from movies;
title             | is_watched
-------------------------------+------------
Attack of the Killer Tomatoes | t
(1 row)
```
⚠️ _Новую запись не видим, т.к. она не закоммичена в параллельной транзакции._

Session 1: Commit
```
thai=*# commit;
COMMIT
```

SESSION 2: Выбираем все записи
```
thai=*# select * from movies;
title             | is_watched
-------------------------------+------------
Attack of the Killer Tomatoes | t
(1 row)
```
⚠️ _Новой записи всё ещё нет, т.к. на этом уровне изоляции мы видим состояние таблицы на начало транзакции._