
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

```
