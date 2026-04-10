
###
В предыдущем ДЗ был создан кластер MAIN:
###

###
Проверяем, что работает:
###
```sh
postgres@asvpg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log

postgres@asvpg:~$ ps -xf
    PID TTY      STAT   TIME COMMAND
  32315 pts/2    S      0:00 -bash
  32344 pts/2    R+     0:00  \_ ps -xf
   1232 ?        Ss     0:01 /usr/lib/postgresql/18/bin/postgres -D /var/lib/postgresql/18
   1244 ?        Ss     0:00  \_ postgres: 18/main: io worker 0
   1245 ?        Ss     0:00  \_ postgres: 18/main: io worker 1
   1246 ?        Ss     0:00  \_ postgres: 18/main: io worker 2
   1247 ?        Ss     0:00  \_ postgres: 18/main: checkpointer 
   1248 ?        Ss     0:00  \_ postgres: 18/main: background writer 
   1325 ?        Ss     0:00  \_ postgres: 18/main: walwriter 
   1326 ?        Ss     0:00  \_ postgres: 18/main: autovacuum launcher 
   1327 ?        Ss     0:00  \_ postgres: 18/main: logical replication launcher 
postgres@asvpg:~$
```

###
Проверим состав БД в кластере, перейдем в пользовательскую БД (создана в прошлом ДЗ) под пользователем postgres, проверим, что есть пользовательская таблица с 3 строками:
###
```sh
postgres=# select oid, datname, dattablespace from pg_database order by oid;
  oid  |  datname  | dattablespace 
-------+-----------+---------------
     1 | template1 |          1663
     4 | template0 |          1663
     5 | postgres  |          1663
 16388 | otus_dba1 |          1663
(4 rows)

postgres=# \c otus_dba1
You are now connected to database "otus_dba1" as user "postgres".
otus_dba1=# 
otus_dba1=# \conninfo
           Connection Information
      Parameter       |        Value        
----------------------+---------------------
 Database             | otus_dba1
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 32398
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=#

otus_dba1=# select table_schema, table_name, column_name, column_default, is_nullable, data_type
otus_dba1-# from information_schema.columns where table_name = 'tb_test' order by ordinal_position;
 table_schema | table_name | column_name |  column_default   | is_nullable |          data_type          
--------------+------------+-------------+-------------------+-------------+-----------------------------
 public       | tb_test    | id          |                   | NO          | bigint
 public       | tb_test    | name        |                   | YES         | text
 public       | tb_test    | now_date    | CURRENT_TIMESTAMP | YES         | timestamp without time zone
(3 rows)

otus_dba1=# 
otus_dba1=# select * from tb_test;
 id | name  |          now_date          
----+-------+----------------------------
  1 | Test1 | 2026-04-07 22:21:12.575669
  2 | Test2 | 2026-04-07 22:21:16.054283
  3 | Test3 | 2026-04-07 22:21:19.026294
(3 rows)

otus_dba1=#
```

###
Останавливаем кластер БД:
###
```sh
postgres@asvpg:~$ sudo pg_ctlcluster 18 main stop
postgres@asvpg:~$ ps -xf
    PID TTY      STAT   TIME COMMAND
  32315 pts/2    S      0:00 -bash
  32534 pts/2    R+     0:00  \_ ps -xf
postgres@asvpg:~$
```

###
Создайте дополнительный диск 10 GB и подключите его к виртуальной машине + пункты 6-8
###
####
ниже указана последовательность в настройке, за основу взят следующий материал: https://dzen.ru/a/YutJU3cX-VGHyHZN
####
```sh
asvpg@asvpg:~$ su - postgres
Password: 
postgres@asvpg:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           794M  1.6M  793M   1% /run
/dev/sda2       251G   12G  227G   5% /
tmpfs           3.9G  1.1M  3.9G   1% /dev/shm
tmpfs           5.0M  8.0K  5.0M   1% /run/lock
tmpfs           794M  116K  794M   1% /run/user/1001
postgres@asvpg:~$ 
postgres@asvpg:~$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0  73.9M  1 loop /snap/core22/2045
loop2    7:2    0 245.1M  1 loop /snap/firefox/6565
loop3    7:3    0  11.1M  1 loop /snap/firmware-updater/167
loop4    7:4    0   516M  1 loop /snap/gnome-42-2204/202
loop5    7:5    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop6    7:6    0  10.8M  1 loop /snap/snap-store/1270
loop7    7:7    0  49.3M  1 loop /snap/snapd/24792
loop8    7:8    0   576K  1 loop /snap/snapd-desktop-integration/315
loop9    7:9    0  48.4M  1 loop /snap/snapd/26382
sda      8:0    0   256G  0 disk 
├─sda1   8:1    0     1M  0 part 
└─sda2   8:2    0   256G  0 part /
sdb      8:16   0    10G  0 disk 
sr0     11:0    1  1024M  0 rom  
postgres@asvpg:~$ 
postgres@asvpg:~$ sudo fdisk /dev/sdb
[sudo] password for postgres: 

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0xaebb92a0.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519): 

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

postgres@asvpg:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2621184 4k blocks and 655360 inodes
Filesystem UUID: a16cf906-2fab-496f-a843-6f148ec7837a
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

postgres@asvpg:~$ sudo mkdir /media/hdd
postgres@asvpg:~$ sudo mount /dev/sdb1 /media/hdd
postgres@asvpg:~$ 
postgres@asvpg:~$ sudo nano /etc/fstab
postgres@asvpg:~$

--рестарт ВМ

postgres@asvpg:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           794M  1.6M  793M   1% /run
/dev/sda2       251G   12G  227G   5% /
tmpfs           3.9G  1.1M  3.9G   1% /dev/shm
tmpfs           5.0M  8.0K  5.0M   1% /run/lock
/dev/sdb1       9.8G   24K  9.3G   1% /media/hdd
tmpfs           794M   96K  794M   1% /run/user/120
tmpfs           794M  120K  794M   1% /run/user/1001
postgres@asvpg:~$
```

###
Назначьте владельцем нового каталога /media/hdd пользователя postgres;
###
```sh
postgres@asvpg:~$ cd /media
postgres@asvpg:/media$ ls -altr
total 20
drwxr-xr-x  23 root root 4096 Jan 26 22:59 ..
drwxr-x---+  2 root root 4096 Jan 26 23:21 postgres
drwxr-x---+  2 root root 4096 Jan 26 23:54 asvpg
drwxr-xr-x   3 root root 4096 Apr 10 13:38 hdd
drwxr-xr-x   5 root root 4096 Apr 10 13:38 .
postgres@asvpg:/media$ 
postgres@asvpg:/media$ sudo chown -R postgres:postgres /media/hdd
postgres@asvpg:/media$ ls -altr
total 20
drwxr-xr-x  23 root     root     4096 Jan 26 22:59 ..
drwxr-x---+  2 root     root     4096 Jan 26 23:21 postgres
drwxr-x---+  2 root     root     4096 Jan 26 23:54 asvpg
drwxr-xr-x   3 postgres postgres 4096 Apr 10 13:38 hdd
drwxr-xr-x   5 root     root     4096 Apr 10 13:38 .
postgres@asvpg:/media$
```

###
Перенесите каталог данных PostgreSQL с текущего пути на /mnt/data (с сохранением прав);
###
```sh
otus_dba1=# show data_directory;
       data_directory        
-----------------------------
 /var/lib/postgresql/18/main
(1 row)

otus_dba1=#

postgres@asvpg:/var/lib/postgresql/18/main/base$ pwd
/var/lib/postgresql/18/main/base
postgres@asvpg:/var/lib/postgresql/18/main/base$ ls -altr
total 40
drwx------  2 postgres postgres  4096 Apr  7 18:22 4
drwx------  6 postgres postgres  4096 Apr  7 20:12 .
drwx------ 19 postgres postgres  4096 Apr 10 13:58 ..
drwx------  2 postgres postgres 12288 Apr 10 13:59 5
drwx------  2 postgres postgres  4096 Apr 10 13:59 1
drwx------  2 postgres postgres 12288 Apr 10 14:09 16388
postgres@asvpg:/var/lib/postgresql/18/main/base$
``

###
Останавливаем кластер БД:
###
```sh
postgres@asvpg:/media$ sudo pg_ctlcluster 18 main stop
postgres@asvpg:/media$ sudo pg_ctlcluster 18 main status
pg_ctl: no server running
postgres@asvpg:/media$ ps -xf
    PID TTY      STAT   TIME COMMAND
   4472 pts/1    S+     0:00 -bash
   4273 pts/0    S      0:00 -bash
   4549 pts/0    R+     0:00  \_ ps -xf
postgres@asvpg:/media$
```

###
Перемещаем включая подкаталоги 18/main:
###
```sh
postgres@asvpg:/media$ cd hdd
postgres@asvpg:/media/hdd$ ls -altr
total 24
drwx------ 2 postgres postgres 16384 Apr 10 13:38 lost+found
drwxr-xr-x 3 postgres postgres  4096 Apr 10 13:38 .
drwxr-xr-x 5 root     root      4096 Apr 10 13:38 ..
postgres@asvpg:/media/hdd$ 
postgres@asvpg:/media/hdd$ sudo rsync -av /var/lib/postgresql /media/hdd
sending incremental file list
postgresql/
postgresql/18/
postgresql/18/main/
postgresql/18/main/PG_VERSION
postgresql/18/main/postgresql.auto.conf
postgresql/18/main/postmaster.opts
postgresql/18/main/base/
postgresql/18/main/base/1/
postgresql/18/main/base/1/112
postgresql/18/main/base/1/113
postgresql/18/main/base/1/1247
postgresql/18/main/base/1/1247_fsm
postgresql/18/main/base/1/1247_vm
postgresql/18/main/base/1/1249
postgresql/18/main/base/1/1249_fsm
postgresql/18/main/base/1/1249_vm
postgresql/18/main/base/1/1255
postgresql/18/main/base/1/1255_fsm
postgresql/18/main/base/1/1255_vm
postgresql/18/main/base/1/1259
postgresql/18/main/base/1/1259_fsm
postgresql/18/main/base/1/1259_vm
postgresql/18/main/base/1/13492
postgresql/18/main/base/1/13492_fsm
postgresql/18/main/base/1/13492_vm
postgresql/18/main/base/1/13495
postgresql/18/main/base/1/13496
postgresql/18/main/base/1/13497
postgresql/18/main/base/1/13497_fsm
postgresql/18/main/base/1/13497_vm
postgresql/18/main/base/1/13500
postgresql/18/main/base/1/13501
postgresql/18/main/base/1/13502
postgresql/18/main/base/1/13502_fsm
postgresql/18/main/base/1/13502_vm
postgresql/18/main/base/1/13505
postgresql/18/main/base/1/13506
postgresql/18/main/base/1/13507
postgresql/18/main/base/1/13507_fsm
postgresql/18/main/base/1/13507_vm
postgresql/18/main/base/1/13510
postgresql/18/main/base/1/13511
postgresql/18/main/base/1/1417
postgresql/18/main/base/1/1418
postgresql/18/main/base/1/174
postgresql/18/main/base/1/175
postgresql/18/main/base/1/2187
postgresql/18/main/base/1/2224
postgresql/18/main/base/1/2228
postgresql/18/main/base/1/2328
postgresql/18/main/base/1/2336
postgresql/18/main/base/1/2337
postgresql/18/main/base/1/2579
postgresql/18/main/base/1/2600
postgresql/18/main/base/1/2600_fsm
postgresql/18/main/base/1/2600_vm
postgresql/18/main/base/1/2601
postgresql/18/main/base/1/2601_fsm
postgresql/18/main/base/1/2601_vm
postgresql/18/main/base/1/2602
postgresql/18/main/base/1/2602_fsm
postgresql/18/main/base/1/2602_vm
postgresql/18/main/base/1/2603
postgresql/18/main/base/1/2603_fsm
postgresql/18/main/base/1/2603_vm
postgresql/18/main/base/1/2604
postgresql/18/main/base/1/2605
postgresql/18/main/base/1/2605_fsm
postgresql/18/main/base/1/2605_vm
postgresql/18/main/base/1/2606
postgresql/18/main/base/1/2606_fsm
postgresql/18/main/base/1/2606_vm
postgresql/18/main/base/1/2607
postgresql/18/main/base/1/2607_fsm
postgresql/18/main/base/1/2607_vm
postgresql/18/main/base/1/2608
postgresql/18/main/base/1/2608_fsm
postgresql/18/main/base/1/2608_vm
postgresql/18/main/base/1/2609
postgresql/18/main/base/1/2609_fsm
postgresql/18/main/base/1/2609_vm
postgresql/18/main/base/1/2610
postgresql/18/main/base/1/2610_fsm
postgresql/18/main/base/1/2610_vm
postgresql/18/main/base/1/2611
postgresql/18/main/base/1/2612
postgresql/18/main/base/1/2612_fsm
postgresql/18/main/base/1/2612_vm
postgresql/18/main/base/1/2613
postgresql/18/main/base/1/2615
postgresql/18/main/base/1/2615_fsm
postgresql/18/main/base/1/2615_vm
postgresql/18/main/base/1/2616
postgresql/18/main/base/1/2616_fsm
postgresql/18/main/base/1/2616_vm
postgresql/18/main/base/1/2617
postgresql/18/main/base/1/2617_fsm
postgresql/18/main/base/1/2617_vm
postgresql/18/main/base/1/2618
postgresql/18/main/base/1/2618_fsm
postgresql/18/main/base/1/2618_vm
postgresql/18/main/base/1/2619
postgresql/18/main/base/1/2619_fsm
postgresql/18/main/base/1/2619_vm
postgresql/18/main/base/1/2620
postgresql/18/main/base/1/2650
postgresql/18/main/base/1/2651
postgresql/18/main/base/1/2652
postgresql/18/main/base/1/2653
postgresql/18/main/base/1/2654
postgresql/18/main/base/1/2655
postgresql/18/main/base/1/2656
postgresql/18/main/base/1/2657
postgresql/18/main/base/1/2658
postgresql/18/main/base/1/2659
postgresql/18/main/base/1/2660
postgresql/18/main/base/1/2661
postgresql/18/main/base/1/2662
postgresql/18/main/base/1/2663
postgresql/18/main/base/1/2664
postgresql/18/main/base/1/2665
postgresql/18/main/base/1/2666
postgresql/18/main/base/1/2667
postgresql/18/main/base/1/2668
postgresql/18/main/base/1/2669
postgresql/18/main/base/1/2670
postgresql/18/main/base/1/2673
postgresql/18/main/base/1/2674
postgresql/18/main/base/1/2675
postgresql/18/main/base/1/2678
postgresql/18/main/base/1/2679
postgresql/18/main/base/1/2680
postgresql/18/main/base/1/2681
postgresql/18/main/base/1/2682
postgresql/18/main/base/1/2683
postgresql/18/main/base/1/2684
postgresql/18/main/base/1/2685
postgresql/18/main/base/1/2686
postgresql/18/main/base/1/2687
postgresql/18/main/base/1/2688
postgresql/18/main/base/1/2689
postgresql/18/main/base/1/2690
postgresql/18/main/base/1/2691
postgresql/18/main/base/1/2692
postgresql/18/main/base/1/2693
postgresql/18/main/base/1/2696
postgresql/18/main/base/1/2699
postgresql/18/main/base/1/2701
postgresql/18/main/base/1/2702
postgresql/18/main/base/1/2703
postgresql/18/main/base/1/2704
postgresql/18/main/base/1/2753
postgresql/18/main/base/1/2753_fsm
postgresql/18/main/base/1/2753_vm
postgresql/18/main/base/1/2754
postgresql/18/main/base/1/2755
postgresql/18/main/base/1/2756
postgresql/18/main/base/1/2757
postgresql/18/main/base/1/2830
postgresql/18/main/base/1/2831
postgresql/18/main/base/1/2832
postgresql/18/main/base/1/2833
postgresql/18/main/base/1/2834
postgresql/18/main/base/1/2835
postgresql/18/main/base/1/2836
postgresql/18/main/base/1/2836_fsm
postgresql/18/main/base/1/2836_vm
postgresql/18/main/base/1/2837
postgresql/18/main/base/1/2838
postgresql/18/main/base/1/2838_fsm
postgresql/18/main/base/1/2838_vm
postgresql/18/main/base/1/2839
postgresql/18/main/base/1/2840
postgresql/18/main/base/1/2840_fsm
postgresql/18/main/base/1/2840_vm
postgresql/18/main/base/1/2841
postgresql/18/main/base/1/2995
postgresql/18/main/base/1/2996
postgresql/18/main/base/1/3079
postgresql/18/main/base/1/3079_fsm
postgresql/18/main/base/1/3079_vm
postgresql/18/main/base/1/3080
postgresql/18/main/base/1/3081
postgresql/18/main/base/1/3085
postgresql/18/main/base/1/3118
postgresql/18/main/base/1/3119
postgresql/18/main/base/1/3164
postgresql/18/main/base/1/3256
postgresql/18/main/base/1/3257
postgresql/18/main/base/1/3258
postgresql/18/main/base/1/3350
postgresql/18/main/base/1/3351
postgresql/18/main/base/1/3379
postgresql/18/main/base/1/3380
postgresql/18/main/base/1/3381
postgresql/18/main/base/1/3394
postgresql/18/main/base/1/3394_fsm
postgresql/18/main/base/1/3394_vm
postgresql/18/main/base/1/3395
postgresql/18/main/base/1/3429
postgresql/18/main/base/1/3430
postgresql/18/main/base/1/3431
postgresql/18/main/base/1/3433
postgresql/18/main/base/1/3439
postgresql/18/main/base/1/3440
postgresql/18/main/base/1/3455
postgresql/18/main/base/1/3456
postgresql/18/main/base/1/3456_fsm
postgresql/18/main/base/1/3456_vm
postgresql/18/main/base/1/3466
postgresql/18/main/base/1/3467
postgresql/18/main/base/1/3468
postgresql/18/main/base/1/3501
postgresql/18/main/base/1/3502
postgresql/18/main/base/1/3503
postgresql/18/main/base/1/3534
postgresql/18/main/base/1/3541
postgresql/18/main/base/1/3541_fsm
postgresql/18/main/base/1/3541_vm
postgresql/18/main/base/1/3542
postgresql/18/main/base/1/3574
postgresql/18/main/base/1/3575
postgresql/18/main/base/1/3576
postgresql/18/main/base/1/3596
postgresql/18/main/base/1/3597
postgresql/18/main/base/1/3598
postgresql/18/main/base/1/3599
postgresql/18/main/base/1/3600
postgresql/18/main/base/1/3600_fsm
postgresql/18/main/base/1/3600_vm
postgresql/18/main/base/1/3601
postgresql/18/main/base/1/3601_fsm
postgresql/18/main/base/1/3601_vm
postgresql/18/main/base/1/3602
postgresql/18/main/base/1/3602_fsm
postgresql/18/main/base/1/3602_vm
postgresql/18/main/base/1/3603
postgresql/18/main/base/1/3603_fsm
postgresql/18/main/base/1/3603_vm
postgresql/18/main/base/1/3604
postgresql/18/main/base/1/3605
postgresql/18/main/base/1/3606
postgresql/18/main/base/1/3607
postgresql/18/main/base/1/3608
postgresql/18/main/base/1/3609
postgresql/18/main/base/1/3712
postgresql/18/main/base/1/3764
postgresql/18/main/base/1/3764_fsm
postgresql/18/main/base/1/3764_vm
postgresql/18/main/base/1/3766
postgresql/18/main/base/1/3767
postgresql/18/main/base/1/3997
postgresql/18/main/base/1/4143
postgresql/18/main/base/1/4144
postgresql/18/main/base/1/4145
postgresql/18/main/base/1/4146
postgresql/18/main/base/1/4147
postgresql/18/main/base/1/4148
postgresql/18/main/base/1/4149
postgresql/18/main/base/1/4150
postgresql/18/main/base/1/4151
postgresql/18/main/base/1/4152
postgresql/18/main/base/1/4153
postgresql/18/main/base/1/4154
postgresql/18/main/base/1/4155
postgresql/18/main/base/1/4156
postgresql/18/main/base/1/4157
postgresql/18/main/base/1/4158
postgresql/18/main/base/1/4159
postgresql/18/main/base/1/4160
postgresql/18/main/base/1/4163
postgresql/18/main/base/1/4164
postgresql/18/main/base/1/4165
postgresql/18/main/base/1/4166
postgresql/18/main/base/1/4167
postgresql/18/main/base/1/4168
postgresql/18/main/base/1/4169
postgresql/18/main/base/1/4170
postgresql/18/main/base/1/4171
postgresql/18/main/base/1/4172
postgresql/18/main/base/1/4173
postgresql/18/main/base/1/4174
postgresql/18/main/base/1/5002
postgresql/18/main/base/1/548
postgresql/18/main/base/1/549
postgresql/18/main/base/1/6102
postgresql/18/main/base/1/6104
postgresql/18/main/base/1/6106
postgresql/18/main/base/1/6110
postgresql/18/main/base/1/6111
postgresql/18/main/base/1/6112
postgresql/18/main/base/1/6113
postgresql/18/main/base/1/6116
postgresql/18/main/base/1/6117
postgresql/18/main/base/1/6175
postgresql/18/main/base/1/6176
postgresql/18/main/base/1/6228
postgresql/18/main/base/1/6229
postgresql/18/main/base/1/6237
postgresql/18/main/base/1/6238
postgresql/18/main/base/1/6239
postgresql/18/main/base/1/6351
postgresql/18/main/base/1/6352
postgresql/18/main/base/1/826
postgresql/18/main/base/1/827
postgresql/18/main/base/1/828
postgresql/18/main/base/1/PG_VERSION
postgresql/18/main/base/1/pg_filenode.map
postgresql/18/main/base/1/pg_internal.init
postgresql/18/main/base/16388/
postgresql/18/main/base/16388/112
postgresql/18/main/base/16388/113
postgresql/18/main/base/16388/1247
postgresql/18/main/base/16388/1247_fsm
postgresql/18/main/base/16388/1247_vm
postgresql/18/main/base/16388/1249
postgresql/18/main/base/16388/1249_fsm
postgresql/18/main/base/16388/1249_vm
postgresql/18/main/base/16388/1255
postgresql/18/main/base/16388/1255_fsm
postgresql/18/main/base/16388/1255_vm
postgresql/18/main/base/16388/1259
postgresql/18/main/base/16388/1259_fsm
postgresql/18/main/base/16388/1259_vm
postgresql/18/main/base/16388/13492
postgresql/18/main/base/16388/13492_fsm
postgresql/18/main/base/16388/13492_vm
postgresql/18/main/base/16388/13495
postgresql/18/main/base/16388/13496
postgresql/18/main/base/16388/13497
postgresql/18/main/base/16388/13497_fsm
postgresql/18/main/base/16388/13497_vm
postgresql/18/main/base/16388/13500
postgresql/18/main/base/16388/13501
postgresql/18/main/base/16388/13502
postgresql/18/main/base/16388/13502_fsm
postgresql/18/main/base/16388/13502_vm
postgresql/18/main/base/16388/13505
postgresql/18/main/base/16388/13506
postgresql/18/main/base/16388/13507
postgresql/18/main/base/16388/13507_fsm
postgresql/18/main/base/16388/13507_vm
postgresql/18/main/base/16388/13510
postgresql/18/main/base/16388/13511
postgresql/18/main/base/16388/1417
postgresql/18/main/base/16388/1418
postgresql/18/main/base/16388/16399
postgresql/18/main/base/16388/16400
postgresql/18/main/base/16388/16405
postgresql/18/main/base/16388/16406
postgresql/18/main/base/16388/16407
postgresql/18/main/base/16388/174
postgresql/18/main/base/16388/175
postgresql/18/main/base/16388/2187
postgresql/18/main/base/16388/2224
postgresql/18/main/base/16388/2228
postgresql/18/main/base/16388/2328
postgresql/18/main/base/16388/2336
postgresql/18/main/base/16388/2337
postgresql/18/main/base/16388/2579
postgresql/18/main/base/16388/2600
postgresql/18/main/base/16388/2600_fsm
postgresql/18/main/base/16388/2600_vm
postgresql/18/main/base/16388/2601
postgresql/18/main/base/16388/2601_fsm
postgresql/18/main/base/16388/2601_vm
postgresql/18/main/base/16388/2602
postgresql/18/main/base/16388/2602_fsm
postgresql/18/main/base/16388/2602_vm
postgresql/18/main/base/16388/2603
postgresql/18/main/base/16388/2603_fsm
postgresql/18/main/base/16388/2603_vm
postgresql/18/main/base/16388/2604
postgresql/18/main/base/16388/2605
postgresql/18/main/base/16388/2605_fsm
postgresql/18/main/base/16388/2605_vm
postgresql/18/main/base/16388/2606
postgresql/18/main/base/16388/2606_fsm
postgresql/18/main/base/16388/2606_vm
postgresql/18/main/base/16388/2607
postgresql/18/main/base/16388/2607_fsm
postgresql/18/main/base/16388/2607_vm
postgresql/18/main/base/16388/2608
postgresql/18/main/base/16388/2608_fsm
postgresql/18/main/base/16388/2608_vm
postgresql/18/main/base/16388/2609
postgresql/18/main/base/16388/2609_fsm
postgresql/18/main/base/16388/2609_vm
postgresql/18/main/base/16388/2610
postgresql/18/main/base/16388/2610_fsm
postgresql/18/main/base/16388/2610_vm
postgresql/18/main/base/16388/2611
postgresql/18/main/base/16388/2612
postgresql/18/main/base/16388/2612_fsm
postgresql/18/main/base/16388/2612_vm
postgresql/18/main/base/16388/2613
postgresql/18/main/base/16388/2615
postgresql/18/main/base/16388/2615_fsm
postgresql/18/main/base/16388/2615_vm
postgresql/18/main/base/16388/2616
postgresql/18/main/base/16388/2616_fsm
postgresql/18/main/base/16388/2616_vm
postgresql/18/main/base/16388/2617
postgresql/18/main/base/16388/2617_fsm
postgresql/18/main/base/16388/2617_vm
postgresql/18/main/base/16388/2618
postgresql/18/main/base/16388/2618_fsm
postgresql/18/main/base/16388/2618_vm
postgresql/18/main/base/16388/2619
postgresql/18/main/base/16388/2619_fsm
postgresql/18/main/base/16388/2619_vm
postgresql/18/main/base/16388/2620
postgresql/18/main/base/16388/2650
postgresql/18/main/base/16388/2651
postgresql/18/main/base/16388/2652
postgresql/18/main/base/16388/2653
postgresql/18/main/base/16388/2654
postgresql/18/main/base/16388/2655
postgresql/18/main/base/16388/2656
postgresql/18/main/base/16388/2657
postgresql/18/main/base/16388/2658
postgresql/18/main/base/16388/2659
postgresql/18/main/base/16388/2660
postgresql/18/main/base/16388/2661
postgresql/18/main/base/16388/2662
postgresql/18/main/base/16388/2663
postgresql/18/main/base/16388/2664
postgresql/18/main/base/16388/2665
postgresql/18/main/base/16388/2666
postgresql/18/main/base/16388/2667
postgresql/18/main/base/16388/2668
postgresql/18/main/base/16388/2669
postgresql/18/main/base/16388/2670
postgresql/18/main/base/16388/2673
postgresql/18/main/base/16388/2674
postgresql/18/main/base/16388/2675
postgresql/18/main/base/16388/2678
postgresql/18/main/base/16388/2679
postgresql/18/main/base/16388/2680
postgresql/18/main/base/16388/2681
postgresql/18/main/base/16388/2682
postgresql/18/main/base/16388/2683
postgresql/18/main/base/16388/2684
postgresql/18/main/base/16388/2685
postgresql/18/main/base/16388/2686
postgresql/18/main/base/16388/2687
postgresql/18/main/base/16388/2688
postgresql/18/main/base/16388/2689
postgresql/18/main/base/16388/2690
postgresql/18/main/base/16388/2691
postgresql/18/main/base/16388/2692
postgresql/18/main/base/16388/2693
postgresql/18/main/base/16388/2696
postgresql/18/main/base/16388/2699
postgresql/18/main/base/16388/2701
postgresql/18/main/base/16388/2702
postgresql/18/main/base/16388/2703
postgresql/18/main/base/16388/2704
postgresql/18/main/base/16388/2753
postgresql/18/main/base/16388/2753_fsm
postgresql/18/main/base/16388/2753_vm
postgresql/18/main/base/16388/2754
postgresql/18/main/base/16388/2755
postgresql/18/main/base/16388/2756
postgresql/18/main/base/16388/2757
postgresql/18/main/base/16388/2830
postgresql/18/main/base/16388/2831
postgresql/18/main/base/16388/2832
postgresql/18/main/base/16388/2833
postgresql/18/main/base/16388/2834
postgresql/18/main/base/16388/2835
postgresql/18/main/base/16388/2836
postgresql/18/main/base/16388/2836_fsm
postgresql/18/main/base/16388/2836_vm
postgresql/18/main/base/16388/2837
postgresql/18/main/base/16388/2838
postgresql/18/main/base/16388/2838_fsm
postgresql/18/main/base/16388/2838_vm
postgresql/18/main/base/16388/2839
postgresql/18/main/base/16388/2840
postgresql/18/main/base/16388/2840_fsm
postgresql/18/main/base/16388/2840_vm
postgresql/18/main/base/16388/2841
postgresql/18/main/base/16388/2995
postgresql/18/main/base/16388/2996
postgresql/18/main/base/16388/3079
postgresql/18/main/base/16388/3079_fsm
postgresql/18/main/base/16388/3079_vm
postgresql/18/main/base/16388/3080
postgresql/18/main/base/16388/3081
postgresql/18/main/base/16388/3085
postgresql/18/main/base/16388/3118
postgresql/18/main/base/16388/3119
postgresql/18/main/base/16388/3164
postgresql/18/main/base/16388/3256
postgresql/18/main/base/16388/3257
postgresql/18/main/base/16388/3258
postgresql/18/main/base/16388/3350
postgresql/18/main/base/16388/3351
postgresql/18/main/base/16388/3379
postgresql/18/main/base/16388/3380
postgresql/18/main/base/16388/3381
postgresql/18/main/base/16388/3394
postgresql/18/main/base/16388/3394_fsm
postgresql/18/main/base/16388/3394_vm
postgresql/18/main/base/16388/3395
postgresql/18/main/base/16388/3429
postgresql/18/main/base/16388/3430
postgresql/18/main/base/16388/3431
postgresql/18/main/base/16388/3433
postgresql/18/main/base/16388/3439
postgresql/18/main/base/16388/3440
postgresql/18/main/base/16388/3455
postgresql/18/main/base/16388/3456
postgresql/18/main/base/16388/3456_fsm
postgresql/18/main/base/16388/3456_vm
postgresql/18/main/base/16388/3466
postgresql/18/main/base/16388/3467
postgresql/18/main/base/16388/3468
postgresql/18/main/base/16388/3501
postgresql/18/main/base/16388/3502
postgresql/18/main/base/16388/3503
postgresql/18/main/base/16388/3534
postgresql/18/main/base/16388/3541
postgresql/18/main/base/16388/3541_fsm
postgresql/18/main/base/16388/3541_vm
postgresql/18/main/base/16388/3542
postgresql/18/main/base/16388/3574
postgresql/18/main/base/16388/3575
postgresql/18/main/base/16388/3576
postgresql/18/main/base/16388/3596
postgresql/18/main/base/16388/3597
postgresql/18/main/base/16388/3598
postgresql/18/main/base/16388/3599
postgresql/18/main/base/16388/3600
postgresql/18/main/base/16388/3600_fsm
postgresql/18/main/base/16388/3600_vm
postgresql/18/main/base/16388/3601
postgresql/18/main/base/16388/3601_fsm
postgresql/18/main/base/16388/3601_vm
postgresql/18/main/base/16388/3602
postgresql/18/main/base/16388/3602_fsm
postgresql/18/main/base/16388/3602_vm
postgresql/18/main/base/16388/3603
postgresql/18/main/base/16388/3603_fsm
postgresql/18/main/base/16388/3603_vm
postgresql/18/main/base/16388/3604
postgresql/18/main/base/16388/3605
postgresql/18/main/base/16388/3606
postgresql/18/main/base/16388/3607
postgresql/18/main/base/16388/3608
postgresql/18/main/base/16388/3609
postgresql/18/main/base/16388/3712
postgresql/18/main/base/16388/3764
postgresql/18/main/base/16388/3764_fsm
postgresql/18/main/base/16388/3764_vm
postgresql/18/main/base/16388/3766
postgresql/18/main/base/16388/3767
postgresql/18/main/base/16388/3997
postgresql/18/main/base/16388/4143
postgresql/18/main/base/16388/4144
postgresql/18/main/base/16388/4145
postgresql/18/main/base/16388/4146
postgresql/18/main/base/16388/4147
postgresql/18/main/base/16388/4148
postgresql/18/main/base/16388/4149
postgresql/18/main/base/16388/4150
postgresql/18/main/base/16388/4151
postgresql/18/main/base/16388/4152
postgresql/18/main/base/16388/4153
postgresql/18/main/base/16388/4154
postgresql/18/main/base/16388/4155
postgresql/18/main/base/16388/4156
postgresql/18/main/base/16388/4157
postgresql/18/main/base/16388/4158
postgresql/18/main/base/16388/4159
postgresql/18/main/base/16388/4160
postgresql/18/main/base/16388/4163
postgresql/18/main/base/16388/4164
postgresql/18/main/base/16388/4165
postgresql/18/main/base/16388/4166
postgresql/18/main/base/16388/4167
postgresql/18/main/base/16388/4168
postgresql/18/main/base/16388/4169
postgresql/18/main/base/16388/4170
postgresql/18/main/base/16388/4171
postgresql/18/main/base/16388/4172
postgresql/18/main/base/16388/4173
postgresql/18/main/base/16388/4174
postgresql/18/main/base/16388/5002
postgresql/18/main/base/16388/548
postgresql/18/main/base/16388/549
postgresql/18/main/base/16388/6102
postgresql/18/main/base/16388/6104
postgresql/18/main/base/16388/6106
postgresql/18/main/base/16388/6110
postgresql/18/main/base/16388/6111
postgresql/18/main/base/16388/6112
postgresql/18/main/base/16388/6113
postgresql/18/main/base/16388/6116
postgresql/18/main/base/16388/6117
postgresql/18/main/base/16388/6175
postgresql/18/main/base/16388/6176
postgresql/18/main/base/16388/6228
postgresql/18/main/base/16388/6229
postgresql/18/main/base/16388/6237
postgresql/18/main/base/16388/6238
postgresql/18/main/base/16388/6239
postgresql/18/main/base/16388/6351
postgresql/18/main/base/16388/6352
postgresql/18/main/base/16388/826
postgresql/18/main/base/16388/827
postgresql/18/main/base/16388/828
postgresql/18/main/base/16388/PG_VERSION
postgresql/18/main/base/16388/pg_filenode.map
postgresql/18/main/base/16388/pg_internal.init
postgresql/18/main/base/4/
postgresql/18/main/base/4/112
postgresql/18/main/base/4/113
postgresql/18/main/base/4/1247
postgresql/18/main/base/4/1247_fsm
postgresql/18/main/base/4/1247_vm
postgresql/18/main/base/4/1249
postgresql/18/main/base/4/1249_fsm
postgresql/18/main/base/4/1249_vm
postgresql/18/main/base/4/1255
postgresql/18/main/base/4/1255_fsm
postgresql/18/main/base/4/1255_vm
postgresql/18/main/base/4/1259
postgresql/18/main/base/4/1259_fsm
postgresql/18/main/base/4/1259_vm
postgresql/18/main/base/4/13492
postgresql/18/main/base/4/13492_fsm
postgresql/18/main/base/4/13492_vm
postgresql/18/main/base/4/13495
postgresql/18/main/base/4/13496
postgresql/18/main/base/4/13497
postgresql/18/main/base/4/13497_fsm
postgresql/18/main/base/4/13497_vm
postgresql/18/main/base/4/13500
postgresql/18/main/base/4/13501
postgresql/18/main/base/4/13502
postgresql/18/main/base/4/13502_fsm
postgresql/18/main/base/4/13502_vm
postgresql/18/main/base/4/13505
postgresql/18/main/base/4/13506
postgresql/18/main/base/4/13507
postgresql/18/main/base/4/13507_fsm
postgresql/18/main/base/4/13507_vm
postgresql/18/main/base/4/13510
postgresql/18/main/base/4/13511
postgresql/18/main/base/4/1417
postgresql/18/main/base/4/1418
postgresql/18/main/base/4/174
postgresql/18/main/base/4/175
postgresql/18/main/base/4/2187
postgresql/18/main/base/4/2224
postgresql/18/main/base/4/2228
postgresql/18/main/base/4/2328
postgresql/18/main/base/4/2336
postgresql/18/main/base/4/2337
postgresql/18/main/base/4/2579
postgresql/18/main/base/4/2600
postgresql/18/main/base/4/2600_fsm
postgresql/18/main/base/4/2600_vm
postgresql/18/main/base/4/2601
postgresql/18/main/base/4/2601_fsm
postgresql/18/main/base/4/2601_vm
postgresql/18/main/base/4/2602
postgresql/18/main/base/4/2602_fsm
postgresql/18/main/base/4/2602_vm
postgresql/18/main/base/4/2603
postgresql/18/main/base/4/2603_fsm
postgresql/18/main/base/4/2603_vm
postgresql/18/main/base/4/2604
postgresql/18/main/base/4/2605
postgresql/18/main/base/4/2605_fsm
postgresql/18/main/base/4/2605_vm
postgresql/18/main/base/4/2606
postgresql/18/main/base/4/2606_fsm
postgresql/18/main/base/4/2606_vm
postgresql/18/main/base/4/2607
postgresql/18/main/base/4/2607_fsm
postgresql/18/main/base/4/2607_vm
postgresql/18/main/base/4/2608
postgresql/18/main/base/4/2608_fsm
postgresql/18/main/base/4/2608_vm
postgresql/18/main/base/4/2609
postgresql/18/main/base/4/2609_fsm
postgresql/18/main/base/4/2609_vm
postgresql/18/main/base/4/2610
postgresql/18/main/base/4/2610_fsm
postgresql/18/main/base/4/2610_vm
postgresql/18/main/base/4/2611
postgresql/18/main/base/4/2612
postgresql/18/main/base/4/2612_fsm
postgresql/18/main/base/4/2612_vm
postgresql/18/main/base/4/2613
postgresql/18/main/base/4/2615
postgresql/18/main/base/4/2615_fsm
postgresql/18/main/base/4/2615_vm
postgresql/18/main/base/4/2616
postgresql/18/main/base/4/2616_fsm
postgresql/18/main/base/4/2616_vm
postgresql/18/main/base/4/2617
postgresql/18/main/base/4/2617_fsm
postgresql/18/main/base/4/2617_vm
postgresql/18/main/base/4/2618
postgresql/18/main/base/4/2618_fsm
postgresql/18/main/base/4/2618_vm
postgresql/18/main/base/4/2619
postgresql/18/main/base/4/2619_fsm
postgresql/18/main/base/4/2619_vm
postgresql/18/main/base/4/2620
postgresql/18/main/base/4/2650
postgresql/18/main/base/4/2651
postgresql/18/main/base/4/2652
postgresql/18/main/base/4/2653
postgresql/18/main/base/4/2654
postgresql/18/main/base/4/2655
postgresql/18/main/base/4/2656
postgresql/18/main/base/4/2657
postgresql/18/main/base/4/2658
postgresql/18/main/base/4/2659
postgresql/18/main/base/4/2660
postgresql/18/main/base/4/2661
postgresql/18/main/base/4/2662
postgresql/18/main/base/4/2663
postgresql/18/main/base/4/2664
postgresql/18/main/base/4/2665
postgresql/18/main/base/4/2666
postgresql/18/main/base/4/2667
postgresql/18/main/base/4/2668
postgresql/18/main/base/4/2669
postgresql/18/main/base/4/2670
postgresql/18/main/base/4/2673
postgresql/18/main/base/4/2674
postgresql/18/main/base/4/2675
postgresql/18/main/base/4/2678
postgresql/18/main/base/4/2679
postgresql/18/main/base/4/2680
postgresql/18/main/base/4/2681
postgresql/18/main/base/4/2682
postgresql/18/main/base/4/2683
postgresql/18/main/base/4/2684
postgresql/18/main/base/4/2685
postgresql/18/main/base/4/2686
postgresql/18/main/base/4/2687
postgresql/18/main/base/4/2688
postgresql/18/main/base/4/2689
postgresql/18/main/base/4/2690
postgresql/18/main/base/4/2691
postgresql/18/main/base/4/2692
postgresql/18/main/base/4/2693
postgresql/18/main/base/4/2696
postgresql/18/main/base/4/2699
postgresql/18/main/base/4/2701
postgresql/18/main/base/4/2702
postgresql/18/main/base/4/2703
postgresql/18/main/base/4/2704
postgresql/18/main/base/4/2753
postgresql/18/main/base/4/2753_fsm
postgresql/18/main/base/4/2753_vm
postgresql/18/main/base/4/2754
postgresql/18/main/base/4/2755
postgresql/18/main/base/4/2756
postgresql/18/main/base/4/2757
postgresql/18/main/base/4/2830
postgresql/18/main/base/4/2831
postgresql/18/main/base/4/2832
postgresql/18/main/base/4/2833
postgresql/18/main/base/4/2834
postgresql/18/main/base/4/2835
postgresql/18/main/base/4/2836
postgresql/18/main/base/4/2836_fsm
postgresql/18/main/base/4/2836_vm
postgresql/18/main/base/4/2837
postgresql/18/main/base/4/2838
postgresql/18/main/base/4/2838_fsm
postgresql/18/main/base/4/2838_vm
postgresql/18/main/base/4/2839
postgresql/18/main/base/4/2840
postgresql/18/main/base/4/2840_fsm
postgresql/18/main/base/4/2840_vm
postgresql/18/main/base/4/2841
postgresql/18/main/base/4/2995
postgresql/18/main/base/4/2996
postgresql/18/main/base/4/3079
postgresql/18/main/base/4/3079_fsm
postgresql/18/main/base/4/3079_vm
postgresql/18/main/base/4/3080
postgresql/18/main/base/4/3081
postgresql/18/main/base/4/3085
postgresql/18/main/base/4/3118
postgresql/18/main/base/4/3119
postgresql/18/main/base/4/3164
postgresql/18/main/base/4/3256
postgresql/18/main/base/4/3257
postgresql/18/main/base/4/3258
postgresql/18/main/base/4/3350
postgresql/18/main/base/4/3351
postgresql/18/main/base/4/3379
postgresql/18/main/base/4/3380
postgresql/18/main/base/4/3381
postgresql/18/main/base/4/3394
postgresql/18/main/base/4/3394_fsm
postgresql/18/main/base/4/3394_vm
postgresql/18/main/base/4/3395
postgresql/18/main/base/4/3429
postgresql/18/main/base/4/3430
postgresql/18/main/base/4/3431
postgresql/18/main/base/4/3433
postgresql/18/main/base/4/3439
postgresql/18/main/base/4/3440
postgresql/18/main/base/4/3455
postgresql/18/main/base/4/3456
postgresql/18/main/base/4/3456_fsm
postgresql/18/main/base/4/3456_vm
postgresql/18/main/base/4/3466
postgresql/18/main/base/4/3467
postgresql/18/main/base/4/3468
postgresql/18/main/base/4/3501
postgresql/18/main/base/4/3502
postgresql/18/main/base/4/3503
postgresql/18/main/base/4/3534
postgresql/18/main/base/4/3541
postgresql/18/main/base/4/3541_fsm
postgresql/18/main/base/4/3541_vm
postgresql/18/main/base/4/3542
postgresql/18/main/base/4/3574
postgresql/18/main/base/4/3575
postgresql/18/main/base/4/3576
postgresql/18/main/base/4/3596
postgresql/18/main/base/4/3597
postgresql/18/main/base/4/3598
postgresql/18/main/base/4/3599
postgresql/18/main/base/4/3600
postgresql/18/main/base/4/3600_fsm
postgresql/18/main/base/4/3600_vm
postgresql/18/main/base/4/3601
postgresql/18/main/base/4/3601_fsm
postgresql/18/main/base/4/3601_vm
postgresql/18/main/base/4/3602
postgresql/18/main/base/4/3602_fsm
postgresql/18/main/base/4/3602_vm
postgresql/18/main/base/4/3603
postgresql/18/main/base/4/3603_fsm
postgresql/18/main/base/4/3603_vm
postgresql/18/main/base/4/3604
postgresql/18/main/base/4/3605
postgresql/18/main/base/4/3606
postgresql/18/main/base/4/3607
postgresql/18/main/base/4/3608
postgresql/18/main/base/4/3609
postgresql/18/main/base/4/3712
postgresql/18/main/base/4/3764
postgresql/18/main/base/4/3764_fsm
postgresql/18/main/base/4/3764_vm
postgresql/18/main/base/4/3766
postgresql/18/main/base/4/3767
postgresql/18/main/base/4/3997
postgresql/18/main/base/4/4143
postgresql/18/main/base/4/4144
postgresql/18/main/base/4/4145
postgresql/18/main/base/4/4146
postgresql/18/main/base/4/4147
postgresql/18/main/base/4/4148
postgresql/18/main/base/4/4149
postgresql/18/main/base/4/4150
postgresql/18/main/base/4/4151
postgresql/18/main/base/4/4152
postgresql/18/main/base/4/4153
postgresql/18/main/base/4/4154
postgresql/18/main/base/4/4155
postgresql/18/main/base/4/4156
postgresql/18/main/base/4/4157
postgresql/18/main/base/4/4158
postgresql/18/main/base/4/4159
postgresql/18/main/base/4/4160
postgresql/18/main/base/4/4163
postgresql/18/main/base/4/4164
postgresql/18/main/base/4/4165
postgresql/18/main/base/4/4166
postgresql/18/main/base/4/4167
postgresql/18/main/base/4/4168
postgresql/18/main/base/4/4169
postgresql/18/main/base/4/4170
postgresql/18/main/base/4/4171
postgresql/18/main/base/4/4172
postgresql/18/main/base/4/4173
postgresql/18/main/base/4/4174
postgresql/18/main/base/4/5002
postgresql/18/main/base/4/548
postgresql/18/main/base/4/549
postgresql/18/main/base/4/6102
postgresql/18/main/base/4/6104
postgresql/18/main/base/4/6106
postgresql/18/main/base/4/6110
postgresql/18/main/base/4/6111
postgresql/18/main/base/4/6112
postgresql/18/main/base/4/6113
postgresql/18/main/base/4/6116
postgresql/18/main/base/4/6117
postgresql/18/main/base/4/6175
postgresql/18/main/base/4/6176
postgresql/18/main/base/4/6228
postgresql/18/main/base/4/6229
postgresql/18/main/base/4/6237
postgresql/18/main/base/4/6238
postgresql/18/main/base/4/6239
postgresql/18/main/base/4/6351
postgresql/18/main/base/4/6352
postgresql/18/main/base/4/826
postgresql/18/main/base/4/827
postgresql/18/main/base/4/828
postgresql/18/main/base/4/PG_VERSION
postgresql/18/main/base/4/pg_filenode.map
postgresql/18/main/base/5/
postgresql/18/main/base/5/112
postgresql/18/main/base/5/113
postgresql/18/main/base/5/1247
postgresql/18/main/base/5/1247_fsm
postgresql/18/main/base/5/1247_vm
postgresql/18/main/base/5/1249
postgresql/18/main/base/5/1249_fsm
postgresql/18/main/base/5/1249_vm
postgresql/18/main/base/5/1255
postgresql/18/main/base/5/1255_fsm
postgresql/18/main/base/5/1255_vm
postgresql/18/main/base/5/1259
postgresql/18/main/base/5/1259_fsm
postgresql/18/main/base/5/1259_vm
postgresql/18/main/base/5/13492
postgresql/18/main/base/5/13492_fsm
postgresql/18/main/base/5/13492_vm
postgresql/18/main/base/5/13495
postgresql/18/main/base/5/13496
postgresql/18/main/base/5/13497
postgresql/18/main/base/5/13497_fsm
postgresql/18/main/base/5/13497_vm
postgresql/18/main/base/5/13500
postgresql/18/main/base/5/13501
postgresql/18/main/base/5/13502
postgresql/18/main/base/5/13502_fsm
postgresql/18/main/base/5/13502_vm
postgresql/18/main/base/5/13505
postgresql/18/main/base/5/13506
postgresql/18/main/base/5/13507
postgresql/18/main/base/5/13507_fsm
postgresql/18/main/base/5/13507_vm
postgresql/18/main/base/5/13510
postgresql/18/main/base/5/13511
postgresql/18/main/base/5/1417
postgresql/18/main/base/5/1418
postgresql/18/main/base/5/174
postgresql/18/main/base/5/175
postgresql/18/main/base/5/2187
postgresql/18/main/base/5/2224
postgresql/18/main/base/5/2228
postgresql/18/main/base/5/2328
postgresql/18/main/base/5/2336
postgresql/18/main/base/5/2337
postgresql/18/main/base/5/2579
postgresql/18/main/base/5/2600
postgresql/18/main/base/5/2600_fsm
postgresql/18/main/base/5/2600_vm
postgresql/18/main/base/5/2601
postgresql/18/main/base/5/2601_fsm
postgresql/18/main/base/5/2601_vm
postgresql/18/main/base/5/2602
postgresql/18/main/base/5/2602_fsm
postgresql/18/main/base/5/2602_vm
postgresql/18/main/base/5/2603
postgresql/18/main/base/5/2603_fsm
postgresql/18/main/base/5/2603_vm
postgresql/18/main/base/5/2604
postgresql/18/main/base/5/2605
postgresql/18/main/base/5/2605_fsm
postgresql/18/main/base/5/2605_vm
postgresql/18/main/base/5/2606
postgresql/18/main/base/5/2606_fsm
postgresql/18/main/base/5/2606_vm
postgresql/18/main/base/5/2607
postgresql/18/main/base/5/2607_fsm
postgresql/18/main/base/5/2607_vm
postgresql/18/main/base/5/2608
postgresql/18/main/base/5/2608_fsm
postgresql/18/main/base/5/2608_vm
postgresql/18/main/base/5/2609
postgresql/18/main/base/5/2609_fsm
postgresql/18/main/base/5/2609_vm
postgresql/18/main/base/5/2610
postgresql/18/main/base/5/2610_fsm
postgresql/18/main/base/5/2610_vm
postgresql/18/main/base/5/2611
postgresql/18/main/base/5/2612
postgresql/18/main/base/5/2612_fsm
postgresql/18/main/base/5/2612_vm
postgresql/18/main/base/5/2613
postgresql/18/main/base/5/2615
postgresql/18/main/base/5/2615_fsm
postgresql/18/main/base/5/2615_vm
postgresql/18/main/base/5/2616
postgresql/18/main/base/5/2616_fsm
postgresql/18/main/base/5/2616_vm
postgresql/18/main/base/5/2617
postgresql/18/main/base/5/2617_fsm
postgresql/18/main/base/5/2617_vm
postgresql/18/main/base/5/2618
postgresql/18/main/base/5/2618_fsm
postgresql/18/main/base/5/2618_vm
postgresql/18/main/base/5/2619
postgresql/18/main/base/5/2619_fsm
postgresql/18/main/base/5/2619_vm
postgresql/18/main/base/5/2620
postgresql/18/main/base/5/2650
postgresql/18/main/base/5/2651
postgresql/18/main/base/5/2652
postgresql/18/main/base/5/2653
postgresql/18/main/base/5/2654
postgresql/18/main/base/5/2655
postgresql/18/main/base/5/2656
postgresql/18/main/base/5/2657
postgresql/18/main/base/5/2658
postgresql/18/main/base/5/2659
postgresql/18/main/base/5/2660
postgresql/18/main/base/5/2661
postgresql/18/main/base/5/2662
postgresql/18/main/base/5/2663
postgresql/18/main/base/5/2664
postgresql/18/main/base/5/2665
postgresql/18/main/base/5/2666
postgresql/18/main/base/5/2667
postgresql/18/main/base/5/2668
postgresql/18/main/base/5/2669
postgresql/18/main/base/5/2670
postgresql/18/main/base/5/2673
postgresql/18/main/base/5/2674
postgresql/18/main/base/5/2675
postgresql/18/main/base/5/2678
postgresql/18/main/base/5/2679
postgresql/18/main/base/5/2680
postgresql/18/main/base/5/2681
postgresql/18/main/base/5/2682
postgresql/18/main/base/5/2683
postgresql/18/main/base/5/2684
postgresql/18/main/base/5/2685
postgresql/18/main/base/5/2686
postgresql/18/main/base/5/2687
postgresql/18/main/base/5/2688
postgresql/18/main/base/5/2689
postgresql/18/main/base/5/2690
postgresql/18/main/base/5/2691
postgresql/18/main/base/5/2692
postgresql/18/main/base/5/2693
postgresql/18/main/base/5/2696
postgresql/18/main/base/5/2699
postgresql/18/main/base/5/2701
postgresql/18/main/base/5/2702
postgresql/18/main/base/5/2703
postgresql/18/main/base/5/2704
postgresql/18/main/base/5/2753
postgresql/18/main/base/5/2753_fsm
postgresql/18/main/base/5/2753_vm
postgresql/18/main/base/5/2754
postgresql/18/main/base/5/2755
postgresql/18/main/base/5/2756
postgresql/18/main/base/5/2757
postgresql/18/main/base/5/2830
postgresql/18/main/base/5/2831
postgresql/18/main/base/5/2832
postgresql/18/main/base/5/2833
postgresql/18/main/base/5/2834
postgresql/18/main/base/5/2835
postgresql/18/main/base/5/2836
postgresql/18/main/base/5/2836_fsm
postgresql/18/main/base/5/2836_vm
postgresql/18/main/base/5/2837
postgresql/18/main/base/5/2838
postgresql/18/main/base/5/2838_fsm
postgresql/18/main/base/5/2838_vm
postgresql/18/main/base/5/2839
postgresql/18/main/base/5/2840
postgresql/18/main/base/5/2840_fsm
postgresql/18/main/base/5/2840_vm
postgresql/18/main/base/5/2841
postgresql/18/main/base/5/2995
postgresql/18/main/base/5/2996
postgresql/18/main/base/5/3079
postgresql/18/main/base/5/3079_fsm
postgresql/18/main/base/5/3079_vm
postgresql/18/main/base/5/3080
postgresql/18/main/base/5/3081
postgresql/18/main/base/5/3085
postgresql/18/main/base/5/3118
postgresql/18/main/base/5/3119
postgresql/18/main/base/5/3164
postgresql/18/main/base/5/3256
postgresql/18/main/base/5/3257
postgresql/18/main/base/5/3258
postgresql/18/main/base/5/3350
postgresql/18/main/base/5/3351
postgresql/18/main/base/5/3379
postgresql/18/main/base/5/3380
postgresql/18/main/base/5/3381
postgresql/18/main/base/5/3394
postgresql/18/main/base/5/3394_fsm
postgresql/18/main/base/5/3394_vm
postgresql/18/main/base/5/3395
postgresql/18/main/base/5/3429
postgresql/18/main/base/5/3430
postgresql/18/main/base/5/3431
postgresql/18/main/base/5/3433
postgresql/18/main/base/5/3439
postgresql/18/main/base/5/3440
postgresql/18/main/base/5/3455
postgresql/18/main/base/5/3456
postgresql/18/main/base/5/3456_fsm
postgresql/18/main/base/5/3456_vm
postgresql/18/main/base/5/3466
postgresql/18/main/base/5/3467
postgresql/18/main/base/5/3468
postgresql/18/main/base/5/3501
postgresql/18/main/base/5/3502
postgresql/18/main/base/5/3503
postgresql/18/main/base/5/3534
postgresql/18/main/base/5/3541
postgresql/18/main/base/5/3541_fsm
postgresql/18/main/base/5/3541_vm
postgresql/18/main/base/5/3542
postgresql/18/main/base/5/3574
postgresql/18/main/base/5/3575
postgresql/18/main/base/5/3576
postgresql/18/main/base/5/3596
postgresql/18/main/base/5/3597
postgresql/18/main/base/5/3598
postgresql/18/main/base/5/3599
postgresql/18/main/base/5/3600
postgresql/18/main/base/5/3600_fsm
postgresql/18/main/base/5/3600_vm
postgresql/18/main/base/5/3601
postgresql/18/main/base/5/3601_fsm
postgresql/18/main/base/5/3601_vm
postgresql/18/main/base/5/3602
postgresql/18/main/base/5/3602_fsm
postgresql/18/main/base/5/3602_vm
postgresql/18/main/base/5/3603
postgresql/18/main/base/5/3603_fsm
postgresql/18/main/base/5/3603_vm
postgresql/18/main/base/5/3604
postgresql/18/main/base/5/3605
postgresql/18/main/base/5/3606
postgresql/18/main/base/5/3607
postgresql/18/main/base/5/3608
postgresql/18/main/base/5/3609
postgresql/18/main/base/5/3712
postgresql/18/main/base/5/3764
postgresql/18/main/base/5/3764_fsm
postgresql/18/main/base/5/3764_vm
postgresql/18/main/base/5/3766
postgresql/18/main/base/5/3767
postgresql/18/main/base/5/3997
postgresql/18/main/base/5/4143
postgresql/18/main/base/5/4144
postgresql/18/main/base/5/4145
postgresql/18/main/base/5/4146
postgresql/18/main/base/5/4147
postgresql/18/main/base/5/4148
postgresql/18/main/base/5/4149
postgresql/18/main/base/5/4150
postgresql/18/main/base/5/4151
postgresql/18/main/base/5/4152
postgresql/18/main/base/5/4153
postgresql/18/main/base/5/4154
postgresql/18/main/base/5/4155
postgresql/18/main/base/5/4156
postgresql/18/main/base/5/4157
postgresql/18/main/base/5/4158
postgresql/18/main/base/5/4159
postgresql/18/main/base/5/4160
postgresql/18/main/base/5/4163
postgresql/18/main/base/5/4164
postgresql/18/main/base/5/4165
postgresql/18/main/base/5/4166
postgresql/18/main/base/5/4167
postgresql/18/main/base/5/4168
postgresql/18/main/base/5/4169
postgresql/18/main/base/5/4170
postgresql/18/main/base/5/4171
postgresql/18/main/base/5/4172
postgresql/18/main/base/5/4173
postgresql/18/main/base/5/4174
postgresql/18/main/base/5/5002
postgresql/18/main/base/5/548
postgresql/18/main/base/5/549
postgresql/18/main/base/5/6102
postgresql/18/main/base/5/6104
postgresql/18/main/base/5/6106
postgresql/18/main/base/5/6110
postgresql/18/main/base/5/6111
postgresql/18/main/base/5/6112
postgresql/18/main/base/5/6113
postgresql/18/main/base/5/6116
postgresql/18/main/base/5/6117
postgresql/18/main/base/5/6175
postgresql/18/main/base/5/6176
postgresql/18/main/base/5/6228
postgresql/18/main/base/5/6229
postgresql/18/main/base/5/6237
postgresql/18/main/base/5/6238
postgresql/18/main/base/5/6239
postgresql/18/main/base/5/6351
postgresql/18/main/base/5/6352
postgresql/18/main/base/5/826
postgresql/18/main/base/5/827
postgresql/18/main/base/5/828
postgresql/18/main/base/5/PG_VERSION
postgresql/18/main/base/5/pg_filenode.map
postgresql/18/main/global/
postgresql/18/main/global/1213
postgresql/18/main/global/1213_fsm
postgresql/18/main/global/1213_vm
postgresql/18/main/global/1214
postgresql/18/main/global/1232
postgresql/18/main/global/1233
postgresql/18/main/global/1260
postgresql/18/main/global/1260_fsm
postgresql/18/main/global/1260_vm
postgresql/18/main/global/1261
postgresql/18/main/global/1261_fsm
postgresql/18/main/global/1261_vm
postgresql/18/main/global/1262
postgresql/18/main/global/1262_fsm
postgresql/18/main/global/1262_vm
postgresql/18/main/global/2396
postgresql/18/main/global/2396_fsm
postgresql/18/main/global/2396_vm
postgresql/18/main/global/2397
postgresql/18/main/global/2671
postgresql/18/main/global/2672
postgresql/18/main/global/2676
postgresql/18/main/global/2677
postgresql/18/main/global/2694
postgresql/18/main/global/2695
postgresql/18/main/global/2697
postgresql/18/main/global/2698
postgresql/18/main/global/2846
postgresql/18/main/global/2847
postgresql/18/main/global/2964
postgresql/18/main/global/2965
postgresql/18/main/global/2966
postgresql/18/main/global/2967
postgresql/18/main/global/3592
postgresql/18/main/global/3593
postgresql/18/main/global/4060
postgresql/18/main/global/4061
postgresql/18/main/global/4177
postgresql/18/main/global/4178
postgresql/18/main/global/4183
postgresql/18/main/global/4184
postgresql/18/main/global/4185
postgresql/18/main/global/4186
postgresql/18/main/global/6000
postgresql/18/main/global/6001
postgresql/18/main/global/6002
postgresql/18/main/global/6100
postgresql/18/main/global/6114
postgresql/18/main/global/6115
postgresql/18/main/global/6243
postgresql/18/main/global/6244
postgresql/18/main/global/6245
postgresql/18/main/global/6246
postgresql/18/main/global/6247
postgresql/18/main/global/6302
postgresql/18/main/global/6303
postgresql/18/main/global/pg_control
postgresql/18/main/global/pg_filenode.map
postgresql/18/main/global/pg_internal.init
postgresql/18/main/pg_commit_ts/
postgresql/18/main/pg_dynshmem/
postgresql/18/main/pg_logical/
postgresql/18/main/pg_logical/replorigin_checkpoint
postgresql/18/main/pg_logical/mappings/
postgresql/18/main/pg_logical/snapshots/
postgresql/18/main/pg_multixact/
postgresql/18/main/pg_multixact/members/
postgresql/18/main/pg_multixact/members/0000
postgresql/18/main/pg_multixact/offsets/
postgresql/18/main/pg_multixact/offsets/0000
postgresql/18/main/pg_notify/
postgresql/18/main/pg_replslot/
postgresql/18/main/pg_serial/
postgresql/18/main/pg_snapshots/
postgresql/18/main/pg_stat/
postgresql/18/main/pg_stat/pgstat.stat
postgresql/18/main/pg_stat_tmp/
postgresql/18/main/pg_subtrans/
postgresql/18/main/pg_subtrans/0000
postgresql/18/main/pg_tblspc/
postgresql/18/main/pg_twophase/
postgresql/18/main/pg_wal/
postgresql/18/main/pg_wal/000000010000000000000001
postgresql/18/main/pg_wal/000000010000000000000002
postgresql/18/main/pg_wal/archive_status/
postgresql/18/main/pg_wal/summaries/
postgresql/18/main/pg_xact/
postgresql/18/main/pg_xact/0000

sent 65,749,076 bytes  received 24,447 bytes  43,849,015.33 bytes/sec
total size is 65,657,730  speedup is 1.00
postgres@asvpg:/media/hdd$ 
```

###
Проверяем содержимое новой директории:
###
```sh
postgres@asvpg:/media/hdd/postgresql/18/main$ pwd
/media/hdd/postgresql/18/main
postgres@asvpg:/media/hdd/postgresql/18/main$ ls -altr
total 88
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 ..
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_twophase
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_tblspc
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_stat_tmp
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_snapshots
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_serial
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_replslot
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_notify
drwx------  4 postgres postgres 4096 Apr  7 18:22 pg_multixact
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_dynshmem
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_commit_ts
-rw-------  1 postgres postgres    3 Apr  7 18:22 PG_VERSION
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_xact
drwx------  2 postgres postgres 4096 Apr  7 18:22 pg_subtrans
-rw-------  1 postgres postgres  125 Apr  7 19:53 postgresql.auto.conf
drwx------  6 postgres postgres 4096 Apr  7 20:12 base
drwx------  4 postgres postgres 4096 Apr  7 20:26 pg_wal
-rw-------  1 postgres postgres  130 Apr 10 13:58 postmaster.opts
drwx------  2 postgres postgres 4096 Apr 10 14:09 global
drwx------  4 postgres postgres 4096 Apr 10 14:20 pg_logical
drwx------  2 postgres postgres 4096 Apr 10 14:20 pg_stat
drwx------ 19 postgres postgres 4096 Apr 10 14:20 .
postgres@asvpg:/media/hdd/postgresql/18/main$ cd base
postgres@asvpg:/media/hdd/postgresql/18/main/base$ ls -altr
total 32
drwx------  2 postgres postgres  4096 Apr  7 18:22 4
drwx------  6 postgres postgres  4096 Apr  7 20:12 .
drwx------  2 postgres postgres  4096 Apr 10 13:59 5
drwx------  2 postgres postgres  4096 Apr 10 13:59 1
drwx------  2 postgres postgres 12288 Apr 10 14:09 16388
drwx------ 19 postgres postgres  4096 Apr 10 14:20 ..
postgres@asvpg:/media/hdd/postgresql/18/main/base$
```

###
Переименовываем изначальный подкаталог в 18_bak
###
```sh
postgres@asvpg:/var/lib/postgresql$ ls -altr
total 12
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 .
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 18
drwxr-xr-x 74 root     root     4096 Apr  9 15:16 ..
postgres@asvpg:/var/lib/postgresql$ mv 18 18_bak
postgres@asvpg:/var/lib/postgresql$ ls -altr
total 12
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 18_bak
drwxr-xr-x 74 root     root     4096 Apr  9 15:16 ..
drwxr-xr-x  3 postgres postgres 4096 Apr 10 14:28 .
postgres@asvpg:/var/lib/postgresql$
```

###
Проверяем статус кластера и даже без старта видим, что система его не видит, т.к. обращается по старому пути:
###
```sh
postgres@asvpg:/media/hdd/postgresql/18/main/base$ sudo pg_ctlcluster 18 main status
Error: /var/lib/postgresql/18/main is not accessible or does not exist

postgres@asvpg:/media/hdd/postgresql/18/main/base$ sudo pg_ctlcluster 18 main start
Error: /var/lib/postgresql/18/main is not accessible or does not exist
postgres@asvpg:/media/hdd/postgresql/18/main/base$
```

###
Редактируем конфиг файл postgresql.conf в части параметра data_directory. Т.к. кластер не работает, выполняем именно через текстовый файл (аналог PFILE в Oracle), а не через auto.conf (аналог SPFILE в Oracle):
###
```sh
postgres@asvpg:~$ sudo nano /etc/postgresql/18/main/postgresql.conf 
[sudo] password for postgres: 
postgres@asvpg:~$

postgres@asvpg:/media/hdd/postgresql/18/main/base$ cd /etc/postgresql/18/main/
postgres@asvpg:/etc/postgresql/18/main$ ls -altr
total 68
drwxr-xr-x 3 postgres postgres  4096 Apr  7 18:22 ..
-rw-r--r-- 1 postgres postgres   317 Apr  7 18:22 start.conf
-rw-r--r-- 1 postgres postgres   143 Apr  7 18:22 pg_ctl.conf
-rw-r----- 1 postgres postgres  2681 Apr  7 18:22 pg_ident.conf
-rw-r--r-- 1 postgres postgres   315 Apr  7 18:22 environment
drwxr-xr-x 2 postgres postgres  4096 Apr  7 18:22 conf.d
-rw-r----- 1 postgres postgres  5971 Apr  7 20:01 pg_hba.conf
-rw-r--r-- 1 postgres postgres 32762 Apr 10 14:36 postgresql.conf
drwxr-xr-x 3 postgres postgres  4096 Apr 10 14:36 .
postgres@asvpg:/etc/postgresql/18/main$ cat postgresql.conf
# -----------------------------
# PostgreSQL configuration file
# -----------------------------
#
# This file consists of lines of the form:
#
#   name = value
#
# (The "=" is optional.)  Whitespace may be used.  Comments are introduced with
# "#" anywhere on a line.  The complete list of parameter names and allowed
# values can be found in the PostgreSQL documentation.
#
# The commented-out settings shown in this file represent the default values.
# Re-commenting a setting is NOT sufficient to revert it to the default value;
# you need to reload the server.
#
# This file is read on server startup and when the server receives a SIGHUP
# signal.  If you edit the file on a running system, you have to SIGHUP the
# server for the changes to take effect, run "pg_ctl reload", or execute
# "SELECT pg_reload_conf()".  Some parameters, which are marked below,
# require a server shutdown and restart to take effect.
#
# Any parameter can also be given as a command-line option to the server, e.g.,
# "postgres -c log_connections=all".  Some parameters can be changed at run time
# with the "SET" SQL command.
#
# Memory units:  B  = bytes            Time units:  us  = microseconds
#                kB = kilobytes                     ms  = milliseconds
#                MB = megabytes                     s   = seconds
#                GB = gigabytes                     min = minutes
#                TB = terabytes                     h   = hours
#                                                   d   = days


#------------------------------------------------------------------------------
# FILE LOCATIONS
#------------------------------------------------------------------------------

# The default values of these variables are driven from the -D command-line
# option or PGDATA environment variable, represented here as ConfigDir.

data_directory = '/media/hdd/postgresql/18/main'		# use data in another directory
# (change requires restart)
```

###
Запускаем кластер повторно:
###
```sh
postgres@asvpg:~$ sudo pg_ctlcluster 18 main status
pg_ctl: no server running
postgres@asvpg:~$ sudo pg_ctlcluster 18 main start
postgres@asvpg:~$ sudo pg_ctlcluster 18 main status
pg_ctl: server is running (PID: 4779)
/usr/lib/postgresql/18/bin/postgres "-D" "/media/hdd/postgresql/18/main" "-c" "config_file=/etc/postgresql/18/main/postgresql.conf"
postgres@asvpg:~$ ps -xf
    PID TTY      STAT   TIME COMMAND
   4472 pts/1    S+     0:00 -bash
   4273 pts/0    S      0:00 -bash
   4804 pts/0    R+     0:00  \_ ps -xf
   4779 ?        Ss     0:00 /usr/lib/postgresql/18/bin/postgres -D /media/hdd/p
   4780 ?        Ss     0:00  \_ postgres: 18/main: io worker 0
   4781 ?        Ss     0:00  \_ postgres: 18/main: io worker 1
   4782 ?        Ss     0:00  \_ postgres: 18/main: io worker 2
   4783 ?        Ss     0:00  \_ postgres: 18/main: checkpointer 
   4784 ?        Ss     0:00  \_ postgres: 18/main: background writer 
   4786 ?        Ss     0:00  \_ postgres: 18/main: walwriter 
   4787 ?        Ss     0:00  \_ postgres: 18/main: autovacuum launcher 
   4788 ?        Ss     0:00  \_ postgres: 18/main: logical replication launcher
postgres@asvpg:~$
```

###
Подключаемся, проверяем, что тестовая таблица с данными на месте:
###
```sh
postgres@asvpg:~$ psql -d otus_dba1
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

otus_dba1=# \conninfo
           Connection Information
      Parameter       |        Value        
----------------------+---------------------
 Database             | otus_dba1
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 4809
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=# select * from tb_test;
 id | name  |          now_date          
----+-------+----------------------------
  1 | Test1 | 2026-04-07 22:21:12.575669
  2 | Test2 | 2026-04-07 22:21:16.054283
  3 | Test3 | 2026-04-07 22:21:19.026294
(3 rows)

otus_dba1=# exit
postgres@asvpg:~$
```

###
Возвращаем параметр data_directory в прежнее значение
###


#
С прошлых 2 занятий осталось несколько вопросов. если получится, буду признателен ответам:
1). Довольно часто в литературе оперируют понятиями кластер БД PostgreSQL и экземпляр. В чем отличие между этими понятиями?
Кластер БД - набор БД (пользовательская, рутовая (postgres) и шаблонные (template0, template1), а экземпляр - это набор фоновых процессов и общая память (аля как в Oracle)?
2). Можно ли поменять пути для каталогов файлов конфигурации, данных, логов и бинарников в рамках процесса установки или это хардкод? В Oracle этим процессом можно было управлять и каждый настраивал под себя.
3). В pg_class есть столбцы oid, relfilenode. Часто они идентичны. В чем их отличие?
4). Интересно узнать подробности про общие (глобальные) объекты в кластере БД. Это что то типа multitenant архитектуры в Oracle с CDB\PDB?
#
