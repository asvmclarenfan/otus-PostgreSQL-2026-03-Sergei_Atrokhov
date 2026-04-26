
###
Конфигурация моей ВМ - 8 ядер, 8ГБ RAM, 250 ГБ SSD
###
```sh
postgres@asvpg:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       1.3Gi       1.6Gi        50Mi       5.2Gi       6.4Gi
Swap:          4.0Gi          0B       4.0Gi
postgres@asvpg:~$

postgres@asvpg:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           794M  1.7M  793M   1% /run
/dev/sda2       251G   14G  225G   6% / --> postgresql
tmpfs           3.9G  1.1M  3.9G   1% /dev/shm
tmpfs           5.0M  8.0K  5.0M   1% /run/lock
/dev/sdb1       9.8G   63M  9.2G   1% /media/hdd
tmpfs           794M  136K  794M   1% /run/user/1001
postgres@asvpg:~$
```

###
Установка параметров инициализации для выполнения ДЗ:
###
```sh
otus_dba1=# show max_connections;
 max_connections 
-----------------
 100
(1 row)

otus_dba1=# alter system set max_connections = 40;
ALTER SYSTEM
otus_dba1=# 
otus_dba1=# show shared_buffers;
 shared_buffers 
----------------
 128MB
(1 row)

otus_dba1=# alter system set shared_buffers = '1GB';
ALTER SYSTEM
otus_dba1=# alter system set effective_cache_size = '3GB';
ALTER SYSTEM
otus_dba1=# show effective_cache_size;
 effective_cache_size 
----------------------
 4GB
(1 row)

otus_dba1=# show maintenance_work_mem;
 maintenance_work_mem 
----------------------
 64MB
(1 row)

otus_dba1=# alter system set maintenance_work_mem = '512MB';
ALTER SYSTEM
otus_dba1=# show checkpoint_completion_target;
 checkpoint_completion_target 
------------------------------
 0.9
(1 row)

otus_dba1=# show wal_buffers;
 wal_buffers 
-------------
 4MB
(1 row)

otus_dba1=# alter system set wal_buffers = '16MB';
ALTER SYSTEM
otus_dba1=# show default_statistics_target;
 default_statistics_target 
---------------------------
 100
(1 row)

otus_dba1=# alter system set default_statistics_target = 500;
ALTER SYSTEM
otus_dba1=# show random_page_cost;
 random_page_cost 
------------------
 4
(1 row)

otus_dba1=# show effective_io_concurrency;
 effective_io_concurrency 
--------------------------
 16
(1 row)

otus_dba1=# alter system set effective_io_concurrency = 2;
ALTER SYSTEM
otus_dba1=# show work_mem;
 work_mem 
----------
 4MB
(1 row)

otus_dba1=# alter system set work_mem = '6553kB';
ALTER SYSTEM
otus_dba1=# show min_wal_size;
 min_wal_size 
--------------
 80MB
(1 row)

otus_dba1=# alter system set min_wal_size = '4GB';
ALTER SYSTEM
otus_dba1=# show max_wal_size;
 max_wal_size 
--------------
 1GB
(1 row)

otus_dba1=# alter system set max_wal_size = '16GB';
ALTER SYSTEM
otus_dba1=#
```

###
Выполняем рестарт кластера БД и проверяем значения параметров:
###
```sh
postgres@asvpg:~$ sudo pg_ctlcluster 18 main stop
[sudo] password for postgres: 
Warning: The unit file, source configuration file or drop-ins of postgresql@18-main.service changed on disk. Run 'systemctl daemon-reload' to reload units.
postgres@asvpg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 down   postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ sudo pg_ctlcluster 18 main start
Warning: The unit file, source configuration file or drop-ins of postgresql@18-main.service changed on disk. Run 'systemctl daemon-reload' to reload units.
postgres@asvpg:~$ systemctl daemon-reload
postgres@asvpg:~$ 
postgres@asvpg:~$ 
postgres@asvpg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ sudo pg_ctlcluster 18 main stop
postgres@asvpg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 down   postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ ps -xf
    PID TTY      STAT   TIME COMMAND
  12337 pts/3    S      0:00 -bash
 333457 pts/3    R+     0:00  \_ ps -xf
postgres@asvpg:~$ sudo pg_ctlcluster 18 main start
postgres@asvpg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
postgres@asvpg:~$ ps -xf
    PID TTY      STAT   TIME COMMAND
  12337 pts/3    S      0:00 -bash
 333788 pts/3    R+     0:00  \_ ps -xf
 333546 ?        Ss     0:00 /usr/lib/postgresql/18/bin/postgres -D /var/lib/pos
 333547 ?        Ss     0:00  \_ postgres: 18/main: io worker 0
 333548 ?        Ss     0:00  \_ postgres: 18/main: io worker 1
 333549 ?        Ss     0:00  \_ postgres: 18/main: io worker 2
 333550 ?        Ss     0:00  \_ postgres: 18/main: checkpointer 
 333551 ?        Ss     0:00  \_ postgres: 18/main: background writer 
 333553 ?        Ss     0:00  \_ postgres: 18/main: walwriter 
 333554 ?        Ss     0:00  \_ postgres: 18/main: autovacuum launcher 
 333555 ?        Ss     0:00  \_ postgres: 18/main: logical replication launcher
postgres@asvpg:~$

otus_dba1=# select name, setting, unit from pg_settings where name in
(
'max_connections', 'shared_buffers', 'effective_cache_size', 'maintenance_work_mem',
'checkpoint_completion_target', 'wal_buffers', 'default_statistics_target', 'random_page_cost', 'effective_io_concurrency', 'work_mem', 'min_wal_size', 'max_wal_size');
             name             | setting | unit 
------------------------------+---------+------
 checkpoint_completion_target | 0.9     | 
 default_statistics_target    | 500     | 
 effective_cache_size         | 393216  | 8kB
 effective_io_concurrency     | 2       | 
 maintenance_work_mem         | 524288  | kB
 max_connections              | 40      | 
 max_wal_size                 | 16384   | MB
 min_wal_size                 | 4096    | MB
 random_page_cost             | 4       | 
 shared_buffers               | 131072  | 8kB
 wal_buffers                  | 2048    | 8kB
 work_mem                     | 6553    | kB
(12 rows)

otus_dba1=#
```

###
Подготовьте тестовую базу pgbench (pgbench -i postgres) и выполните прогон нагрузки (pgbench -c8 -P 6 -T 60 -U postgres postgres);
###
####
Тестовая БД называется pgbenchtest:
####
```sh
otus_dba1=# select datname from pg_database;
   datname   
-------------
 postgres
 template1
 template0
 otus_dba1
 pgbenchtest
(5 rows)

otus_dba1=#

otus_dba1=# \c pgbenchtest
You are now connected to database "pgbenchtest" as user "postgres".
pgbenchtest=# \dt
                List of tables
 Schema |       Name       | Type  |  Owner   
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
(4 rows)

pgbenchtest=#

###Старт нагрузки:
postgres@asvpg:~$ pgbench -c 8 -P 6 -T 60 -U postgres pgbenchtest
Password: 
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 6.0 s, 1078.7 tps, lat 7.353 ms stddev 5.819, 0 failed
progress: 12.0 s, 1223.4 tps, lat 6.529 ms stddev 7.704, 0 failed
progress: 18.0 s, 1711.8 tps, lat 4.658 ms stddev 5.463, 0 failed
progress: 24.0 s, 1929.2 tps, lat 4.128 ms stddev 3.715, 0 failed
progress: 30.0 s, 2093.7 tps, lat 3.799 ms stddev 2.596, 0 failed
progress: 36.0 s, 2105.8 tps, lat 3.792 ms stddev 2.621, 0 failed
progress: 42.0 s, 2196.5 tps, lat 3.630 ms stddev 2.270, 0 failed
progress: 48.0 s, 2176.3 tps, lat 3.662 ms stddev 2.172, 0 failed
progress: 54.0 s, 2157.3 tps, lat 3.696 ms stddev 2.488, 0 failed
progress: 60.0 s, 1630.2 tps, lat 4.888 ms stddev 7.127, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 109825
number of failed transactions: 0 (0.000%)
latency average = 4.354 ms
latency stddev = 4.397 ms
initial connection time = 39.858 ms
tps = 1831.323569 (without initial connection time)
postgres@asvpg:~$
```

###
Зафиксируйте состояние обслуживания после нагрузки: для пользовательских таблиц сохраните n_dead_tup, last_autovacuum, last_vacuum из pg_stat_user_tables;
###
```sh
pgbenchtest=# select relname, n_dead_tup, last_autovacuum, last_vacuum from pg_stat_user_tables
pgbenchtest-# where relname in ('pgbench_accounts', 'pgbench_branches', 'pgbench_history', 'pgbench_tellers');
     relname      | n_dead_tup |        last_autovacuum        |          last_vacuum          
------------------+------------+-------------------------------+-------------------------------
 pgbench_history  |          0 | 2026-04-26 14:51:55.762067+03 | 
 pgbench_tellers  |          0 | 2026-04-26 14:51:55.746932+03 | 2026-04-26 14:50:25.078488+03
 pgbench_accounts |      41876 |                               | 
 pgbench_branches |          0 | 2026-04-26 14:51:55.744628+03 | 2026-04-26 14:50:25.072413+03
(4 rows)

pgbenchtest=#
```

###
Создайте таблицу с текстовым полем и заполните её 1 000 000 строк (допускается генерация через generate_series);
###
```sh
pgbenchtest=# create table av_table (v1 varchar(100), t2 timestamp);
CREATE TABLE
pgbenchtest=# \dt+ av_table
                                       List of tables
 Schema |   Name   | Type  |  Owner   | Persistence | Access method |  Size   | Description 
--------+----------+-------+----------+-------------+---------------+---------+-------------
 public | av_table | table | postgres | permanent   | heap          | 0 bytes | 
(1 row)

pgbenchtest=#

do $$
declare
   n_of_recs bigint := 1000000;
   random_varchar varchar(100);
   random_timestamp timestamp;
   query text;
   rec record;
begin

   for idx_rec in 1..n_of_recs loop

   -- some random varchar length between 1 and 100
      random_varchar_length := floor(random()*(100-1+1))+1;

      -- some random varchar
      random_varchar := array_to_string(array(select chr((ascii('a') + round(random() * 25)) :: integer) from generate_series(1,random_varchar_length)), ''); 

      -- some random timestamp between '2026-04-01 00:00:00' and '2026-04-26 00:00:00'
      random_timestamp := timestamp '2026-04-01 00:00:00' + random() * (timestamp '2026-04-26 00:00:00' - timestamp '2026-04-01 00:00:00');

      query := 'insert into my_table values($1, $2, $3)';

      execute query using random_varchar, random_timestamp, random_int;

      if idx_rec % 100000 = 0 then
         raise notice 'Num of recs inserted into the table my_table: %', idx_rec;
      end if;

   end loop;

end$$;
```
