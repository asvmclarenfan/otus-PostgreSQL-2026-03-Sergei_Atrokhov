
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

