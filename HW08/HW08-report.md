###
Настройте выполнение контрольной точки раз в 30 секунд (параметр времени) и включите логирование контрольных точек;
###
####
Для выполнения прошлых ДЗ pgbench бфл настроен:
####
```sh
postgres@asvpg:~$ psql -d otus_dba1 -p 5433
Password for user postgres: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

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
```

###
Проверяем текущие настройки контрольной точки (КТ):
###
```sh
otus_dba1=# show checkpoint_timeout;
 checkpoint_timeout 
--------------------
 5min
(1 row)

otus_dba1=# show checkpoint_completion_target;
 checkpoint_completion_target 
------------------------------
 0.9
(1 row)

otus_dba1=#
```
####
Период запуска КТ с условием не достижения значения параметра max_wal_size установлен в 5 минут.
По моему опыту и лучшим практикам это очень мало, на высоконагруженном OLTP можно ставить в 30 минут.
####

####
Проверяем текущие настройки и меняем на требуемые (логирование КТ включено):
####
```sh
otus_dba1=# select name, setting, context from pg_settings where name like '%checkpoint%';
             name             | setting | context 
------------------------------+---------+---------
 checkpoint_completion_target | 0.9     | sighup
 checkpoint_flush_after       | 32      | sighup
 checkpoint_timeout           | 300     | sighup
 checkpoint_warning           | 30      | sighup
 log_checkpoints              | on      | sighup
(5 rows)

otus_dba1=# alter system set checkpoint_timeout = '30s';
ALTER SYSTEM
otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# select name, setting, context from pg_settings where name like '%checkpoint%';
             name             | setting | context 
------------------------------+---------+---------
 checkpoint_completion_target | 0.9     | sighup
 checkpoint_flush_after       | 32      | sighup
 checkpoint_timeout           | 30      | sighup
 checkpoint_warning           | 30      | sighup
 log_checkpoints              | on      | sighup
(5 rows)

otus_dba1=#
```

###
Зафиксируйте начальные значения статистики контрольных точек и WAL (снимок «до»);
###
```sh
otus_dba1=# select * from pg_stat_bgwriter \gx
-[ RECORD 1 ]----+------------------------------
buffers_clean    | 0
maxwritten_clean | 0
buffers_alloc    | 166
stats_reset      | 2026-05-02 11:23:14.945495+03

otus_dba1=#

otus_dba1=# 
-[ RECORD 1 ]-------+------------------------------
num_timed           | 13
num_requested       | 1
num_done            | 1
restartpoints_timed | 0
restartpoints_req   | 0
restartpoints_done  | 0
write_time          | 12
sync_time           | 3
buffers_written     | 0
slru_written        | 3
stats_reset         | 2026-05-02 11:23:14.945495+03

otus_dba1=#

--сохраняем текущий lsn записи в WAL буфер:
otus_dba1=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn 
---------------------------
 1/24110E80
(1 row)

otus_dba1=#
```

###
Запустите нагрузку pgbench на 10 минут и зафиксируйте TPS;
###
```sh
postgres@asvpg:~$ pgbench -c 8 -P 6 -T 600 -U postgres pgbenchtest
Password: 
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 6.0 s, 1392.1 tps, lat 5.699 ms stddev 2.511, 0 failed
progress: 12.0 s, 1782.8 tps, lat 4.477 ms stddev 2.494, 0 failed
...
progress: 594.0 s, 2228.3 tps, lat 3.579 ms stddev 1.845, 0 failed
progress: 600.0 s, 2237.8 tps, lat 3.565 ms stddev 1.994, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 1277974
number of failed transactions: 0 (0.000%)
latency average = 3.746 ms
latency stddev = 2.044 ms
initial connection time = 39.164 ms
tps = 2130.034537 (without initial connection time)
postgres@asvpg:~$
```
###
TPS - 2130
###
###
Зафиксируйте конечные значения статистики контрольных точек и WAL (снимок «после»);
###
```sh
otus_dba1=# select * from pg_stat_bgwriter \gx
-[ RECORD 1 ]----+------------------------------
buffers_clean    | 0
maxwritten_clean | 0
buffers_alloc    | 28575
stats_reset      | 2026-05-02 11:23:14.945495+03

otus_dba1=# select * from pg_stat_checkpointer \gx
-[ RECORD 1 ]-------+------------------------------
num_timed           | 47
num_requested       | 1
num_done            | 22
restartpoints_timed | 0
restartpoints_req   | 0
restartpoints_done  | 0
write_time          | 564115
sync_time           | 704
buffers_written     | 317482
slru_written        | 146
stats_reset         | 2026-05-02 11:23:14.945495+03

otus_dba1=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn 
---------------------------
 1/EF848290
(1 row)

otus_dba1=#
```

###
Рассчитайте объём WAL за 10 минут и средний объём WAL на одну контрольную точку;
###
```sh
otus_dba1=# SELECT '1/EF848290'::pg_lsn - '1/24110E80'::pg_lsn;
  ?column?  
------------
 3413341200 --3255МБ
(1 row)

otus_dba1=#
```
###
За время подачи нагрузки было выполнено 34 КТ (47 - 13), не было ни одной форсированной КТ (все по таймауту).
Смотрим значение параметра max_wal_size:
###
```sh
otus_dba1=# show max_wal_size;
 max_wal_size 
--------------
 16GB
(1 row)

otus_dba1=
```
###
С таким значением и с генерацией 3.2 ГБ за 10 минут - действительно все КТ будут по таймауту.
###
###
Средний объем WALов на каждую КТ - 98МБ - небольшая нагрузка.
###
###
Проверьте по статистике/логам, выполнялись ли контрольные точки строго по расписанию, и объясните отклонения;
###
```sh
--ищем, где назходится лог СУБД (пока не выучил)
root@asvpg:/home/asvpg# find / -type f -name "postgresql*.log"
/var/log/postgresql/postgresql-18-main.log
root@asvpg:/home/asvpg#

root@asvpg:/home/asvpg# su - postgres
postgres@asvpg:~$ cd /var/log/postgresql
postgres@asvpg:/var/log/postgresql$ ls -altr
total 52
-rw-r-----  1 postgres postgres  5700 Apr 18 13:04 postgresql-18-main.log.3.gz
-rw-r-----  1 postgres postgres  2502 Apr 25 02:02 postgresql-18-main.log.2.gz
drwxrwxr-t  2 root     postgres  4096 May  2 11:21 .
-rw-r-----  1 postgres postgres 13642 May  2 11:21 postgresql-18-main.log.1
drwxrwxr-x 17 root     syslog    4096 May  2 11:23 ..
-rw-r-----  1 postgres postgres 13595 May  2 12:05 postgresql-18-main.log
postgres@asvpg:/var/log/postgresql$

--смотри содержимое лога:
2026-05-02 11:45:33.067 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:46:00.086 MSK [1370] LOG:  checkpoint complete: wrote 2215 buffers (1.7%), wrote 2 SLRU buffers; 0 WAL file(s) added, 0 removed, 1 recycled; write=26.934 s, sync=0.038 s, total=27.019 s; sync files=17, longest=0.012 s, average=0.003 s; distance=17522 kB, estimate=17522 kB; lsn=1/2E86D660, redo lsn=1/2522D7F8
2026-05-02 11:46:03.089 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:46:30.191 MSK [1370] LOG:  checkpoint complete: wrote 17156 buffers (13.1%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 9 recycled; write=26.973 s, sync=0.032 s, total=27.103 s; sync files=13, longest=0.012 s, average=0.003 s; distance=158934 kB, estimate=158934 kB; lsn=1/387387A8, redo lsn=1/2ED63330
2026-05-02 11:46:33.194 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:47:00.157 MSK [1370] LOG:  checkpoint complete: wrote 15560 buffers (11.9%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.828 s, sync=0.027 s, total=26.964 s; sync files=6, longest=0.012 s, average=0.005 s; distance=162621 kB, estimate=162621 kB; lsn=1/42D359F8, redo lsn=1/38C32B20
2026-05-02 11:47:03.160 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:47:30.141 MSK [1370] LOG:  checkpoint complete: wrote 16185 buffers (12.3%), wrote 8 SLRU buffers; 0 WAL file(s) added, 0 removed, 11 recycled; write=26.819 s, sync=0.044 s, total=26.981 s; sync files=20, longest=0.012 s, average=0.003 s; distance=169479 kB, estimate=169479 kB; lsn=1/4CAFF640, redo lsn=1/431B4A88
2026-05-02 11:47:33.144 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:48:00.183 MSK [1370] LOG:  checkpoint complete: wrote 15708 buffers (12.0%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 9 recycled; write=26.914 s, sync=0.024 s, total=27.039 s; sync files=6, longest=0.012 s, average=0.004 s; distance=161848 kB, estimate=168716 kB; lsn=1/570D3B58, redo lsn=1/4CFC2DB0
2026-05-02 11:48:03.186 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:48:30.179 MSK [1370] LOG:  checkpoint complete: wrote 15976 buffers (12.2%), wrote 8 SLRU buffers; 0 WAL file(s) added, 0 removed, 11 recycled; write=26.836 s, sync=0.045 s, total=26.994 s; sync files=20, longest=0.012 s, average=0.003 s; distance=169171 kB, estimate=169171 kB; lsn=1/60DD9780, redo lsn=1/574F7BF0
2026-05-02 11:48:33.182 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:49:00.149 MSK [1370] LOG:  checkpoint complete: wrote 15667 buffers (12.0%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.831 s, sync=0.025 s, total=26.968 s; sync files=6, longest=0.012 s, average=0.005 s; distance=161502 kB, estimate=168404 kB; lsn=1/6B30ECE8, redo lsn=1/612AF418
2026-05-02 11:49:03.152 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:49:30.218 MSK [1370] LOG:  checkpoint complete: wrote 16019 buffers (12.2%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.914 s, sync=0.044 s, total=27.066 s; sync files=20, longest=0.011 s, average=0.003 s; distance=168741 kB, estimate=168741 kB; lsn=1/751346C8, redo lsn=1/6B7789E8
2026-05-02 11:49:33.221 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:50:00.166 MSK [1370] LOG:  checkpoint complete: wrote 15713 buffers (12.0%), wrote 8 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.803 s, sync=0.023 s, total=26.946 s; sync files=6, longest=0.012 s, average=0.004 s; distance=162273 kB, estimate=168094 kB; lsn=1/7F63FB00, redo lsn=1/755F11C8
2026-05-02 11:50:03.170 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:50:30.171 MSK [1370] LOG:  checkpoint complete: wrote 15997 buffers (12.2%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.844 s, sync=0.049 s, total=27.002 s; sync files=21, longest=0.011 s, average=0.003 s; distance=168759 kB, estimate=168759 kB; lsn=1/89409A48, redo lsn=1/7FABEF10
2026-05-02 11:50:33.174 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:51:00.121 MSK [1370] LOG:  checkpoint complete: wrote 15610 buffers (11.9%), wrote 8 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.813 s, sync=0.026 s, total=26.948 s; sync files=7, longest=0.011 s, average=0.004 s; distance=161802 kB, estimate=168063 kB; lsn=1/939215A0, redo lsn=1/898C1770
2026-05-02 11:51:03.124 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:51:30.229 MSK [1370] LOG:  checkpoint complete: wrote 15919 buffers (12.1%), wrote 6 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.946 s, sync=0.060 s, total=27.105 s; sync files=20, longest=0.019 s, average=0.003 s; distance=168565 kB, estimate=168565 kB; lsn=1/9D69D818, redo lsn=1/93D5ED80
2026-05-02 11:51:33.232 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:52:00.182 MSK [1370] LOG:  checkpoint complete: wrote 15645 buffers (11.9%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.827 s, sync=0.022 s, total=26.951 s; sync files=6, longest=0.011 s, average=0.004 s; distance=161640 kB, estimate=167872 kB; lsn=1/A7BF6C30, redo lsn=1/9DB38E10
2026-05-02 11:52:03.185 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:52:30.175 MSK [1370] LOG:  checkpoint complete: wrote 15950 buffers (12.2%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 11 recycled; write=26.834 s, sync=0.046 s, total=26.991 s; sync files=20, longest=0.011 s, average=0.003 s; distance=168979 kB, estimate=168979 kB; lsn=1/B19AD1C0, redo lsn=1/A803DAB0
2026-05-02 11:52:33.179 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:53:00.109 MSK [1370] LOG:  checkpoint complete: wrote 15479 buffers (11.8%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 9 recycled; write=26.803 s, sync=0.022 s, total=26.931 s; sync files=6, longest=0.010 s, average=0.004 s; distance=161660 kB, estimate=168247 kB; lsn=1/BBD0EB98, redo lsn=1/B1E1CC10
2026-05-02 11:53:03.113 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:53:30.216 MSK [1370] LOG:  checkpoint complete: wrote 15793 buffers (12.0%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 11 recycled; write=26.937 s, sync=0.043 s, total=27.104 s; sync files=20, longest=0.010 s, average=0.003 s; distance=167230 kB, estimate=168145 kB; lsn=1/C589DDE8, redo lsn=1/BC16C480
2026-05-02 11:53:33.219 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:54:00.169 MSK [1370] LOG:  checkpoint complete: wrote 15338 buffers (11.7%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 9 recycled; write=26.826 s, sync=0.016 s, total=26.950 s; sync files=6, longest=0.006 s, average=0.003 s; distance=159536 kB, estimate=167284 kB; lsn=1/CFB126D8, redo lsn=1/C5D384E0
2026-05-02 11:54:03.174 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:54:30.184 MSK [1370] LOG:  checkpoint complete: wrote 15593 buffers (11.9%), wrote 6 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.864 s, sync=0.041 s, total=27.010 s; sync files=19, longest=0.008 s, average=0.003 s; distance=165813 kB, estimate=167137 kB; lsn=1/D95FA560, redo lsn=1/CFF25CC8
2026-05-02 11:54:33.186 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:55:00.110 MSK [1370] LOG:  checkpoint complete: wrote 15316 buffers (11.7%), wrote 8 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.800 s, sync=0.020 s, total=26.925 s; sync files=6, longest=0.010 s, average=0.004 s; distance=159330 kB, estimate=166356 kB; lsn=1/E3A6CB68, redo lsn=1/D9ABE628
2026-05-02 11:55:03.113 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:55:30.204 MSK [1370] LOG:  checkpoint complete: wrote 15896 buffers (12.1%), wrote 7 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.930 s, sync=0.042 s, total=27.091 s; sync files=20, longest=0.009 s, average=0.003 s; distance=168051 kB, estimate=168051 kB; lsn=1/ED817EC8, redo lsn=1/E3EDB410
2026-05-02 11:55:33.209 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 11:56:00.093 MSK [1370] LOG:  checkpoint complete: wrote 14747 buffers (11.3%), wrote 5 SLRU buffers; 0 WAL file(s) added, 0 removed, 10 recycled; write=26.827 s, sync=0.012 s, total=26.884 s; sync files=7, longest=0.007 s, average=0.002 s; distance=158458 kB, estimate=167092 kB; lsn=1/EF8481E0, redo lsn=1/ED999F08
```
###
Подтверждаем, что интервал КТ составляет 30 секунд (значение параметра checkpoint_timeout), форсированных КТ нет.
###

###
Попробуем подать такую нагрузку, при которой будут появляться форсированные КТ
###
```sh
--уменьшим значение параметра max_wal_size:
otus_dba1=# show max_wal_size;
 max_wal_size 
--------------
 16GB
(1 row)

otus_dba1=# alter system set max_wal_size = '128MB';
ALTER SYSTEM
otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# show max_wal_size;
 max_wal_size 
--------------
 128MB
(1 row)

otus_dba1=#

--Сохраним информацию по КТ и фоновой записи и текущему lsn в WAL буфере:
otus_dba1=# select * from pg_stat_bgwriter \gx
-[ RECORD 1 ]----+------------------------------
buffers_clean    | 0
maxwritten_clean | 0
buffers_alloc    | 28686
stats_reset      | 2026-05-02 11:23:14.945495+03

otus_dba1=# select * from pg_stat_checkpointer \gx
-[ RECORD 1 ]-------+------------------------------
num_timed           | 96
num_requested       | 1
num_done            | 22
restartpoints_timed | 0
restartpoints_req   | 0
restartpoints_done  | 0
write_time          | 564115
sync_time           | 704
buffers_written     | 317482
slru_written        | 146
stats_reset         | 2026-05-02 11:23:14.945495+03

otus_dba1=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn 
---------------------------
 1/EF848290
(1 row)

otus_dba1=#

--запустим pgbench с более агрессивными настройками (параллельность и число клиентов):
postgres@asvpg:~$ pgbench -c 32 -j 4 -P 60 -T 600 -U postgres pgbenchtest
Password: 
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 60.0 s, 2753.5 tps, lat 11.594 ms stddev 11.802, 0 failed
progress: 120.0 s, 2723.9 tps, lat 11.745 ms stddev 11.910, 0 failed
progress: 180.0 s, 2737.1 tps, lat 11.685 ms stddev 11.923, 0 failed
progress: 240.0 s, 2733.6 tps, lat 11.702 ms stddev 11.830, 0 failed
progress: 300.0 s, 2758.6 tps, lat 11.596 ms stddev 11.808, 0 failed
progress: 360.0 s, 2659.0 tps, lat 12.031 ms stddev 12.262, 0 failed
progress: 420.0 s, 2612.5 tps, lat 12.242 ms stddev 12.140, 0 failed
progress: 480.0 s, 2672.4 tps, lat 11.970 ms stddev 12.283, 0 failed
progress: 540.0 s, 2749.1 tps, lat 11.637 ms stddev 11.795, 0 failed
progress: 600.0 s, 2733.1 tps, lat 11.703 ms stddev 11.700, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 32
number of threads: 4
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 1627999
number of failed transactions: 0 (0.000%)
latency average = 11.787 ms
latency stddev = 11.946 ms
initial connection time = 105.129 ms
tps = 2713.660951 (without initial connection time)
postgres@asvpg:~$
```
###
Проверяем статистику:
###
```sh

--теперь видим большое число форсированных КТ (за 30 секунд объем генерируемых WAL-записей превышает установленные 128 МБ)
otus_dba1=# select * from pg_stat_bgwriter \gx
-[ RECORD 1 ]----+------------------------------
buffers_clean    | 0
maxwritten_clean | 0
buffers_alloc    | 39466
stats_reset      | 2026-05-02 11:23:14.945495+03

otus_dba1=# select * from pg_stat_checkpointer \gx
-[ RECORD 1 ]-------+------------------------------
num_timed           | 104
num_requested       | 184
num_done            | 206
restartpoints_timed | 0
restartpoints_req   | 0
restartpoints_done  | 0
write_time          | 1119141
sync_time           | 5202
buffers_written     | 1598013
slru_written        | 719
stats_reset         | 2026-05-02 11:23:14.945495+03

otus_dba1=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn 
---------------------------
 4/CCF4F3C8
(1 row)

otus_dba1=#

--смотрим лог СУБД:
2026-05-02 12:25:43.480 MSK [1370] LOG:  checkpoints are occurring too frequently (9 seconds apart)
2026-05-02 12:25:43.480 MSK [1370] HINT:  Consider increasing the configuration parameter "max_wal_size".
2026-05-02 12:25:43.480 MSK [1370] LOG:  checkpoint starting: wal
2026-05-02 12:25:46.756 MSK [1370] LOG:  checkpoint complete: wrote 6137 buffers (4.7%), wrote 3 SLRU buffers; 0 WAL file(s) added, 4 removed, 0 recycled; write=3.179 s, sync=0.043 s, total=3.276 s; sync files=21, longest=0.006 s, average=0.003 s; distance=55759 kB, estimate=155958 kB; lsn=1/F4AB90C0, redo lsn=1/F100DE68
2026-05-02 12:25:47.064 MSK [1370] LOG:  checkpoints are occurring too frequently (4 seconds apart)
2026-05-02 12:25:47.064 MSK [1370] HINT:  Consider increasing the configuration parameter "max_wal_size".
2026-05-02 12:25:47.064 MSK [1370] LOG:  checkpoint starting: wal
2026-05-02 12:25:50.286 MSK [1370] LOG:  checkpoint complete: wrote 6980 buffers (5.3%), wrote 3 SLRU buffers; 0 WAL file(s) added, 4 removed, 0 recycled; write=3.150 s, sync=0.024 s, total=3.222 s; sync files=10, longest=0.006 s, average=0.003 s; distance=65535 kB, estimate=146916 kB; lsn=1/F8B5A080, redo lsn=1/F500DDF0
2026-05-02 12:25:50.564 MSK [1370] LOG:  checkpoints are occurring too frequently (3 seconds apart)
2026-05-02 12:25:50.564 MSK [1370] HINT:  Consider increasing the configuration parameter "max_wal_size".
...
026-05-02 12:35:35.457 MSK [1370] LOG:  checkpoints are occurring too frequently (4 seconds apart)
2026-05-02 12:35:35.457 MSK [1370] HINT:  Consider increasing the configuration parameter "max_wal_size".
2026-05-02 12:35:35.457 MSK [1370] LOG:  checkpoint starting: wal
2026-05-02 12:35:38.194 MSK [1370] LOG:  checkpoint complete: wrote 6841 buffers (5.2%), wrote 6 SLRU buffers; 0 WAL file(s) added, 0 removed, 4 recycled; write=2.651 s, sync=0.023 s, total=2.738 s; sync files=9, longest=0.006 s, average=0.003 s; distance=65519 kB, estimate=65616 kB; lsn=4/C8A12FD8, redo lsn=4/C5004828
2026-05-02 12:35:38.541 MSK [1370] LOG:  checkpoints are occurring too frequently (3 seconds apart)
2026-05-02 12:35:38.541 MSK [1370] HINT:  Consider increasing the configuration parameter "max_wal_size".
2026-05-02 12:35:38.541 MSK [1370] LOG:  checkpoint starting: wal
2026-05-02 12:35:41.852 MSK [1370] LOG:  checkpoint complete: wrote 7055 buffers (5.4%), wrote 3 SLRU buffers; 0 WAL file(s) added, 0 removed, 4 recycled; write=3.209 s, sync=0.039 s, total=3.312 s; sync files=18, longest=0.006 s, average=0.003 s; distance=65582 kB, estimate=65612 kB; lsn=4/CCB01178, redo lsn=4/C9010068
2026-05-02 12:36:08.880 MSK [1370] LOG:  checkpoint starting: time
2026-05-02 12:36:35.091 MSK [1370] LOG:  checkpoint complete: wrote 6983 buffers (5.3%), wrote 4 SLRU buffers; 0 WAL file(s) added, 0 removed, 3 recycled; write=26.165 s, sync=0.021 s, total=26.211 s; sync files=9, longest=0.005 s, average=0.003 s; distance=64670 kB, estimate=65518 kB; lsn=4/CCF4F318, redo lsn=4/CCF37A48
postgres@asvpg:/var/log/postgresql$

--считаем генерацию WALов:
otus_dba1=# SELECT '4/CCF4F3C8'::pg_lsn - '1/EF848290'::pg_lsn;
  ?column?   
-------------
 12305068344 --11735ГБ
(1 row)

otus_dba1=#
```
###
Значение max_wal_size для такой нагрузки является слишком небольшим!
###


###
Сравните TPS при синхронном и асинхронном подтверждении коммита (2 прогона pgbench в сопоставимых условиях) и объясните разницу;
###
```sh
--По умолчанию синхронное подтверждение транзакции включено (тест проводили с ним):
otus_dba1=# show synchronous_commit;
 synchronous_commit 
--------------------
 on
(1 row)

otus_dba1=#
--TSP - 2713

--выключим и выполним тот же тест:
otus_dba1=# select name, setting, context from pg_settings where name = 'synchronous_commit';
        name        | setting | context 
--------------------+---------+---------
 synchronous_commit | on      | user
(1 row)

otus_dba1=# alter system set synchronous_commit = 'off';
ALTER SYSTEM
otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# show synchronous_commit;
 synchronous_commit 
--------------------
 off
(1 row)

otus_dba1=#

--подаем тот же уровень нагрузки:
postgres@asvpg:~$ pgbench -c 32 -j 4 -P 60 -T 600 -U postgres pgbenchtest
Password: 
pgbench (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
starting vacuum...end.
progress: 60.0 s, 12280.8 tps, lat 2.597 ms stddev 1.933, 0 failed
progress: 120.0 s, 12514.5 tps, lat 2.554 ms stddev 1.852, 0 failed
progress: 180.0 s, 12841.3 tps, lat 2.488 ms stddev 1.863, 0 failed
progress: 240.0 s, 13198.2 tps, lat 2.421 ms stddev 1.590, 0 failed
progress: 300.0 s, 13119.4 tps, lat 2.436 ms stddev 1.719, 0 failed
progress: 360.0 s, 13290.1 tps, lat 2.404 ms stddev 1.705, 0 failed
progress: 420.0 s, 13365.1 tps, lat 2.391 ms stddev 1.587, 0 failed
progress: 480.0 s, 13164.0 tps, lat 2.428 ms stddev 1.799, 0 failed
progress: 540.0 s, 12493.3 tps, lat 2.558 ms stddev 10.592, 0 failed
progress: 600.0 s, 12766.3 tps, lat 2.504 ms stddev 8.379, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 32
number of threads: 4
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 7742007
number of failed transactions: 0 (0.000%)
latency average = 2.476 ms
latency stddev = 4.505 ms
initial connection time = 105.239 ms
tps = 12905.122423 (without initial connection time)
postgres@asvpg:~$
```

###
Наблюдаем огромную разницу в TPS (2713 в синхронном режиме против 12905 при асинхронном)!
Причина - при синхронном режиме при фиксации транзакции работа будет продолжена только после того, как все журнальные записи, которые относятся к этой транзакции, окажутся не диске, что служит для durability и высокой надежности. Если случится сбой, то СУБД выполнит так называемое Instance Recovery автоматически, путем наката записей в WAL-журналах с момента последней успешной КТ. При асинхронном режиме составляющая задержки предварительной записи WAL-записей на диск перед коммитом отсутствует (запись на диск происходит в фоне асинхронно), что с одной стороны сильно ускоряет работу, а с другой является ненадежным решением, которое может привести к потере данных. В промышленных системах такой режим не используется.
###

###
Возвращаем значения параметров в прежние значения:

otus_dba1=# alter system set synchronous_commit = 'on';
ALTER SYSTEM
otus_dba1=# alter system set max_wal_size = '1GB';
ALTER SYSTEM
otus_dba1=# alter system set checkpoint_timeout = '300s';
ALTER SYSTEM
otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# show synchronous_commit;
 synchronous_commit 
--------------------
 on
(1 row)

otus_dba1=# show max_wal_size;
 max_wal_size 
--------------
 1GB
(1 row)

otus_dba1=# show checkpoint_timeout;
 checkpoint_timeout 
--------------------
 5min
(1 row)

otus_dba1=#
###
