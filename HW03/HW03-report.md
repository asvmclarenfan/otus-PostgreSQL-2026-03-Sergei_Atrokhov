
###
В предыдущем ДЗ был создан кластер MAIN:
есть вопрос, который не успел задать на занятии:
довольно часто в литературе оперируют понятиями кластер БД PostgreSQL и экземпляр. В чем отличие между этими понятиями?
Кластер БД - набор БД (пользовательская, рутовая (postgres) и шаблонные (template0, template1), а экземпляр - это набор фоновых процессов и общая память (аля как в Oracle)?
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
``
