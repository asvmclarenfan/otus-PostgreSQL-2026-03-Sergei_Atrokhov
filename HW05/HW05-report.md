
###
Проверим, что кластер MAIN запущен, фоновые процессы работают:
###
```sh
postgres@asvpg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ 
postgres@asvpg:~$ ps -xf
    PID TTY      STAT   TIME COMMAND
   4609 pts/0    S      0:00 -bash
   4861 pts/0    R+     0:00  \_ ps -xf
   1336 ?        Ss     0:00 /usr/lib/postgresql/18/bin/postgres -D /var/lib/postgresql/18/main -c config
   1353 ?        Ss     0:00  \_ postgres: 18/main: io worker 0
   1354 ?        Ss     0:00  \_ postgres: 18/main: io worker 1
   1355 ?        Ss     0:00  \_ postgres: 18/main: io worker 2
   1356 ?        Ss     0:00  \_ postgres: 18/main: checkpointer 
   1357 ?        Ss     0:00  \_ postgres: 18/main: background writer 
   1664 ?        Ss     0:00  \_ postgres: 18/main: walwriter 
   1665 ?        Ss     0:00  \_ postgres: 18/main: autovacuum launcher 
   1666 ?        Ss     0:00  \_ postgres: 18/main: logical replication launcher 
postgres@asvpg:~$
```

###
Создадим новую БД для ДЗ с pgbench:
###
```sh
postgres@asvpg:~$ psql
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

postgres=# select datname from pg_database;
  datname  
-----------
 postgres
 template1
 template0
 otus_dba1
(4 rows)

postgres=# create database pgbenchtest;
CREATE DATABASE

postgres=# select datname from pg_database;
  datname   
------------
 postgres
 template1
 template0
 otus_dba1
 pgbenchtest
(5 rows)

postgres=# 
```

###
Подготовьте pgbench: инициализируйте тестовую базу (scale выбрать 10);
Получаем ошибку создания таблиц..
###
```sh
postgres@asvpg:~$ pgbench -h localhost -p 5433 -U postgres -i -s 10 pgbenchtest
Password: 
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
pgbench: error: query failed: ERROR:  no schema has been selected to create in
LINE 1: create table pgbench_history(tid int,bid int,aid    int,delt...
                     ^
pgbench: detail: Query was: create table pgbench_history(tid int,bid int,aid    int,delta int,mtime timestamp,filler char(22))
postgres@asvpg:~$
```

###
Проверяем настройки схем в search_path:
###
```sh
postgres@asvpg:~$ psql
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

postgres=# \conninfo
           Connection Information
      Parameter       |        Value        
----------------------+---------------------
 Database             | postgres
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 22116
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

postgres=# \c pgbenchtest
You are now connected to database "pgbechtest" as user "postgres".

pgbechtest=# \conninfo
           Connection Information
      Parameter       |        Value        
----------------------+---------------------
 Database             | pgbechtest
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 22394
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

pgbechtest=# show search_path;
 search_path 
-------------
 testnm
(1 row)

pgbechtest=#
```
###
Видим, что в search_path явно указана схема для прошлого ДЗ, проверим ее:
###
```sh
postgres@asvpg:~$ psql -d pgbenchtest
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

pgbechtest=# show search_path;
 search_path 
-------------
 testnm
(1 row)

pgbechtest=# \dn+ testnm
                List of schemas
 Name | Owner | Access privileges | Description 
------+-------+-------------------+-------------
(0 rows)

pgbechtest=# select nspname from pg_namespace order by 1;
      nspname       
--------------------
 information_schema
 pg_catalog
 pg_toast
 public
(4 rows)

pgbechtest=# 
pgbechtest=# alter system reset search_path;
ALTER SYSTEM
pgbechtest=# select context from pg_settings where name = 'search_path';
 context 
---------
 user
(1 row)

pgbechtest=# show search_path;
 search_path 
-------------
 testnm
(1 row)

pgbechtest=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

pgbechtest=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

pgbechtest=#
```

###
У схемы public есть нужные права (USAGE), можно делать в ней:
###
```
pgbechtest=# \dn+ public
                                       List of schemas
  Name  |       Owner       |           Access privileges            |      Description       
--------+-------------------+----------------------------------------+------------------------
 public | pg_database_owner | pg_database_owner=UC/pg_database_owner+| standard public schema
        |                   | =U/pg_database_owner                   | 
(1 row)

pgbechtest=#

postgres@asvpg:~$ pgbench -h localhost -p 5433 -U postgres -i -s 10 pgbenchtest
Password: 
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
vacuuming...                                                                                
creating primary keys...
done in 2.22 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 1.28 s, vacuum 0.06 s, primary keys 0.86 s).
postgres@asvpg:~$
```

###
Выполните baseline-прогон pgbench с фиксированными настройками (например, 60 секунд, 10 клиентов, 4 потока);
###
```sh
postgres@asvpg:/etc/postgresql/18/main$ pgbench -c 10 -j 4 -P 10 -T 60 pgbenchtest
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 10.0 s, 2214.2 tps, lat 4.507 ms stddev 7.048, 0 failed
progress: 20.0 s, 2391.8 tps, lat 4.178 ms stddev 3.864, 0 failed
progress: 30.0 s, 1978.0 tps, lat 5.051 ms stddev 3.201, 0 failed
progress: 40.0 s, 1580.7 tps, lat 6.311 ms stddev 6.309, 0 failed
progress: 50.0 s, 1623.2 tps, lat 6.164 ms stddev 3.311, 0 failed
progress: 60.0 s, 1591.1 tps, lat 6.246 ms stddev 3.362, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 10
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 113810
number of failed transactions: 0 (0.000%)
latency average = 5.278 ms
latency stddev = 5.166 ms
initial connection time = 8.336 ms
tps = 1892.751772 (without initial connection time)
postgres@asvpg:/etc/postgresql/18/main$
```

###
Зафиксируйте tps baseline и условия теста (scale, время, клиенты/потоки);
###
####
TPS - 1892
Scale - 10
Threads - 4
Clients - 10
Duration - 60 sec
####


###
Измените параметры PostgreSQL для получения максимальной производительности
###
####
Запросов, которые могут запрашивать большой объем локальной памяти на сортировки, join и bitmap - нет. Параметр work_mem не тьюним.
shared_buffers равен 120 МБ - это немного, но в рамках теста оставляем как есть.
Считаем, что на данном этапе основной вариант тьюнинга - настройка IO (ввода-вывода).
####
```sh
pgbenchtest=# select name, setting, context from pg_settings
pgbenchtest-# where name in ('wal_sync_method', 'fsync', 'synchronous_commit', 'checkpoint_completion_target',
pgbenchtest(# 'checkpoint_timeout');
             name             |  setting  | context 
------------------------------+-----------+---------
 checkpoint_completion_target | 0.9       | sighup
 checkpoint_timeout           | 300       | sighup
 fsync                        | on        | sighup
 synchronous_commit           | on        | user
 wal_sync_method              | fdatasync | sighup
(5 rows)

pgbenchtest=#
```
###
Параметры checkpoint_completion_target и wal_sync_method уже настроены правильно.
Параметр checkpoint_timeout отвечает за интервал времени, который будет использоваться СУБД для выполнения КТ на протяжении checkpoint_completion_target от этого времени с учетом не превышения значения параметра max_wal_size (при его превышении активируется так называемая форсированная КТ).
###
```sh
pgbenchtest=# show max_wal_size;
 max_wal_size 
--------------
 1GB
(1 row)
```

###
Параметр checkpoint_timeout обычно делают минимум 30 минут, чтобы сильнее размазать IO от выполнения КТ. В рамках нашего теста оставляем как есть.
###

###
Останавливаемся на 2 оставшихся параметрах:
fsync
synchronous_commit
Выключаем их, перезагружаем конфигурацию и запускаем тест еще раз для сравнения:
###
```sh
pgbenchtest=# alter system set fsync = off;
ALTER SYSTEM
pgbenchtest=# alter system set synchronous_commit = off;
ALTER SYSTEM
pgbenchtest=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

pgbenchtest=# select name, setting, context from pg_settings
where name in ('wal_sync_method', 'fsync', 'synchronous_commit', 'checkpoint_completion_target',
'checkpoint_timeout');
             name             |  setting  | context 
------------------------------+-----------+---------
 checkpoint_completion_target | 0.9       | sighup
 checkpoint_timeout           | 300       | sighup
 fsync                        | off       | sighup
 synchronous_commit           | off       | user
 wal_sync_method              | fdatasync | sighup
(5 rows)

pgbenchtest=#

postgres@asvpg:/etc/postgresql/18/main$ pgbench -c 10 -j 4 -P 10 -T 60 pgbenchtest
Password: 
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 10.0 s, 14857.4 tps, lat 0.669 ms stddev 0.307, 0 failed
progress: 20.0 s, 12267.3 tps, lat 0.813 ms stddev 1.211, 0 failed
progress: 30.0 s, 10416.8 tps, lat 0.957 ms stddev 1.428, 0 failed
progress: 40.0 s, 10481.9 tps, lat 0.951 ms stddev 1.541, 0 failed
progress: 50.0 s, 11177.4 tps, lat 0.892 ms stddev 1.300, 0 failed
progress: 60.0 s, 11862.6 tps, lat 0.840 ms stddev 1.369, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 10
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 710644
number of failed transactions: 0 (0.000%)
latency average = 0.841 ms
latency stddev = 1.229 ms
initial connection time = 36.412 ms
tps = 11850.393394 (without initial connection time)
postgres@asvpg:/etc/postgresql/18/main$
```

###
Сравните tps до/после и сделайте вывод, какие изменения дали эффект (scale, время, клиенты/потоки);
###
####
TPS - 11850
Scale - 10
Threads - 4
Clients - 10
Duration - 60 sec
####

###
Видим, что пропускная способность выросла на порядок!
###

###
Проверяем статистику по КТ:
###
```sh
pgbenchtest=# select * from pg_stat_checkpointer \gx
-[ RECORD 1 ]-------+------------------------------
num_timed           | 17
num_requested       | 2
num_done            | 5
restartpoints_timed | 0
restartpoints_req   | 0
restartpoints_done  | 0
write_time          | 572417
sync_time           | 655
buffers_written     | 21835
slru_written        | 96
stats_reset         | 2026-04-18 13:04:15.548598+03

pgbenchtest=#
```

###
Видим 2 форсированные КТ, скорее всего были в рамках рестарта кластера, но предлагается увеличить следующие параметры и проверить еще раз:
###
```sh
pgbenchtest=# alter system set checkpoint_timeout = 900;
ALTER SYSTEM

pgbenchtest=# alter system set max_wal_size = '5GB';
ALTER SYSTEM
pgbenchtest=# 
pgbenchtest=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

pgbenchtest=# show checkpoint_timeout;
 checkpoint_timeout 
--------------------
 15min
(1 row)

pgbenchtest=# show max_wal_size;
 max_wal_size 
--------------
 5GB
(1 row)

pgbenchtest=#

postgres@asvpg:/etc/postgresql/18/main$ pgbench -c 10 -j 4 -P 10 -T 60 pgbenchtest
Password: 
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 10.0 s, 9330.2 tps, lat 1.065 ms stddev 0.804, 0 failed
progress: 20.0 s, 12796.8 tps, lat 0.779 ms stddev 0.424, 0 failed
progress: 30.0 s, 11184.0 tps, lat 0.891 ms stddev 0.425, 0 failed
progress: 40.0 s, 4051.6 tps, lat 2.457 ms stddev 2.538, 0 failed
progress: 50.0 s, 11741.8 tps, lat 0.850 ms stddev 0.695, 0 failed
progress: 60.0 s, 12479.0 tps, lat 0.799 ms stddev 0.287, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 10
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 615806
number of failed transactions: 0 (0.000%)
latency average = 0.971 ms
latency stddev = 0.930 ms
initial connection time = 33.137 ms
tps = 10267.887029 (without initial connection time)
postgres@asvpg:/etc/postgresql/18/main$

pgbenchtest=# select * from pg_stat_checkpointer \gx
-[ RECORD 1 ]-------+------------------------------
num_timed           | 19
num_requested       | 2
num_done            | 6
restartpoints_timed | 0
restartpoints_req   | 0
restartpoints_done  | 0
write_time          | 842394
sync_time           | 656
buffers_written     | 30944
slru_written        | 147
stats_reset         | 2026-04-18 13:04:15.548598+03

pgbenchtest=#
```

###
Стало даже похуже - хотя не должно. Принимаем как следствие использования локальной ВМ.
Форсированных КТ не прибавилось.
###


###
Кратко обоснуйте изменение каких параметров больше всего влияет на повышение производительности;
###
Опытным путем выяснили, что основное влияние на производиетльность оказывают 2 параметра: fsync и syncronous_commit.
По умолчанию, при включенных параметрах, каждая транзакция ожидает завершения операции IO на диск ,что всегда является узким местом в работе СУБД,
т.к. скорость дисков на порядке хуже, чем скорость RAM.
fsync	- при выключенном параметре данные могут временно находиться только в кеше ОС, а при включенном параметре происходит принудительный сброс на диск (через системные вызовы ОС)
synchronous_commit - при выключенном параметре клиенту происходит подтверждение о выполнении транзакции сразу после записи в WAL-буфер в RAM,	а при включенном параметре подтверждение происходит после физической записи WAL-записи на диск, что вызывает задержку.

Минусы - слабая надежность. При проблеме с сервером БД, отключении питания - данные могут быть потеряны\повреждены.
Плюсы - повышение производительности на порядок.

В ПРОМ системах с высоким классом критичности оба параметра всегда включены.


###
Возвращаем измененные параметры в прежние значения:
###
```sh
pgbenchtest=# alter system set fsync = on;
ALTER SYSTEM
pgbenchtest=# alter system set synchronous_commit = on;
ALTER SYSTEM
pgbenchtest=# alter system reset max_wal_size;
ALTER SYSTEM
pgbenchtest=# alter system reset checkpoint_timeout;
ALTER SYSTEM
pgbenchtest=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

pgbenchtest=# select name, setting, context from pg_settings
where name in ('wal_sync_method', 'fsync', 'synchronous_commit', 'checkpoint_completion_target',
'checkpoint_timeout');
             name             |  setting  | context 
------------------------------+-----------+---------
 checkpoint_completion_target | 0.9       | sighup
 checkpoint_timeout           | 300       | sighup
 fsync                        | on        | sighup
 synchronous_commit           | on        | user
 wal_sync_method              | fdatasync | sighup
(5 rows)

pgbenchtest=# show max_wal_size;
 max_wal_size 
--------------
 1GB
(1 row)

pgbenchtest=#
```
