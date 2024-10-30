# Домашнее задание 3

1. Создаем новую таблицу и вставляем 1 млн записей:
```
thai=# create table movies (title text not null);
CREATE TABLE

thai=# INSERT INTO movies SELECT 'Movie '||s.title FROM generate_series(1,1000000) AS s(title);
INSERT 0 1000000
```

2. Размер файла с таблицей:
```
thai=# \d+
                                  List of relations
 Schema |  Name  | Type  | Owner | Persistence | Access method | Size  | Description
--------+--------+-------+-------+-------------+---------------+-------+-------------
 public | movies | table | dds   | permanent   | heap          | 42 MB |
(1 row)
```

3. 5 раз обновляем все строчки:
```
thai=# update movies set title = title||'a';
UPDATE 1000000
thai=# update movies set title = title||'b';
UPDATE 1000000
thai=# update movies set title = title||'c';
UPDATE 1000000
thai=# update movies set title = title||'d';
UPDATE 1000000
thai=# update movies set title = title||'e';
UPDATE 1000000
```

4. Мертвые строки и автовакуум:
```
thai=# SELECT relname, n_live_tup, n_dead_tup, last_autovacuum FROM pg_stat_user_tables WHERE relname = 'movies';
 relname | n_live_tup | n_dead_tup |        last_autovacuum
---------+------------+------------+-------------------------------
 movies  |    1000000 |    4999894 | 2024-10-30 20:34:06.449885+00
(1 row)
```

5. Когда пришел автовакуум
```
thai=# SELECT relname, n_live_tup, n_dead_tup, last_autovacuum FROM pg_stat_user_tables WHERE relname = 'movies';
 relname | n_live_tup | n_dead_tup |        last_autovacuum
---------+------------+------------+-------------------------------
 movies  |     998174 |          0 | 2024-10-30 20:47:09.914213+00
(1 row)
```

6. Еще 5 раз обновляем все строчки:
```
thai=# update movies set title = title||'A';
UPDATE 1000000
thai=# update movies set title = title||'B';
UPDATE 1000000
thai=# update movies set title = title||'C';
UPDATE 1000000
thai=# update movies set title = title||'D';
UPDATE 1000000
thai=# update movies set title = title||'E';
UPDATE 1000000
```
7. Размер файла с таблицей
```
thai=# \d+
                                  List of relations
 Schema |  Name  | Type  | Owner | Persistence | Access method |  Size  | Description
--------+--------+-------+-------+-------------+---------------+--------+-------------
 public | movies | table | dds   | permanent   | heap          | 299 MB |
(1 row)
```

8. Отключаем автовакуум для таблицы movies
```
thai=# ALTER TABLE movies SET (autovacuum_enabled = false);
ALTER TABLE
```

9. 10 раз обновляем все строчки
```
thai=# update movies set title = title||'0';
UPDATE 1000000
thai=# update movies set title = title||'1';
UPDATE 1000000
thai=# update movies set title = title||'2';
UPDATE 1000000
thai=# update movies set title = title||'3';
UPDATE 1000000
thai=# update movies set title = title||'4';
UPDATE 1000000
thai=# update movies set title = title||'5';
UPDATE 1000000
thai=# update movies set title = title||'6';
UPDATE 1000000
thai=# update movies set title = title||'7';
UPDATE 1000000
thai=# update movies set title = title||'8';
UPDATE 1000000
thai=# update movies set title = title||'9';
UPDATE 1000000
```

10. Размер файла с таблицей
```
thai=# \d+
                                  List of relations
 Schema |  Name  | Type  | Owner | Persistence | Access method |  Size  | Description
--------+--------+-------+-------+-------------+---------------+--------+-------------
 public | movies | table | dds   | permanent   | heap          | 623 MB |
(1 row)
```

После создания таблицы и ее заполнения - мы храним 1млн живых строк.
Когда 5 раз обновили - получили 5млн мертвых строк, которые чуть позже автовакуум разметил как пустые.
При втором обновлении новые строки писались как раз в это пустое место.

С выключенным автовакуумом после обновления получили 10млн мертвых строк, которые будут занимать место + размер файла вырос в 2 раза.