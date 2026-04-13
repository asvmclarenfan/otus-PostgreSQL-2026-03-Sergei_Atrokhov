###
Ранее был создан кластер MAIN:
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
Проверим состав БД в кластере, перейдем в пользовательскую БД (создана ранее) под пользователем postgres:
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
```

####
Проверим состав схем и текущую схему:
####
```sh
otus_dba1=# select current_schema();
 current_schema 
----------------
 public
(1 row)

otus_dba1=# select current_schemas(true);
   current_schemas   
---------------------
 {pg_catalog,public}
(1 row)

otus_dba1=# 
otus_dba1=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

otus_dba1=#
```

###
Создайте схему testnm, таблицу t1(c1 int) и вставьте строку
###
```sh
otus_dba1=# create schema testnm;
CREATE SCHEMA
otus_dba1=# 
otus_dba1=# select current_schema();
 current_schema 
----------------
 public
(1 row)

otus_dba1=# select current_schemas(true);
   current_schemas   
---------------------
 {pg_catalog,public}
(1 row)

otus_dba1=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

otus_dba1=#

otus_dba1=# create table t1 (c1 int);
CREATE TABLE
otus_dba1=# insert into t1 values (1);
INSERT 0 1
otus_dba1=# insert into t1 values (2);
INSERT 0 1
otus_dba1=# insert into t1 values (3);
INSERT 0 1
otus_dba1=# select * from t1;
 c1 
----
  1
  2
  3
(3 rows)

otus_dba1=#
```

###
Проверяем в какой схеме создана таблица:
###
```sh
otus_dba1=# \dt public.*
           List of tables
 Schema |  Name   | Type  |  Owner   
--------+---------+-------+----------
 public | t1      | table | postgres
 public | tb_test | table | postgres
(2 rows)

otus_dba1=#
```

###
!!! На занятии Михаил подсвечивал, что после создания схемы таблица будет неявно создана в созданной схеме...
###


###
Создайте роль readonly, выдайте ей CONNECT к testdb, USAGE на testnm, SELECT на таблицы схемы testnm
###

```sh
otus_dba1=# CREATE ROLE readonly;
CREATE ROLE
otus_dba1=# \du+ readonly
             List of roles
 Role name |  Attributes  | Description 
-----------+--------------+-------------
 readonly  | Cannot login | 

otus_dba1=# GRANT CONNECT ON DATABASE otus_dba1 TO readonly;
GRANT

otus_dba1=# 
otus_dba1=# grant USAGE on schema testnm to readonly;
GRANT
otus_dba1=# 
otus_dba1=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
otus_dba1=#
```

###
Создайте пользователя testread/test123, назначьте роль readonly, выполните select * from t1; и зафиксируйте результат
###

```sh
otus_dba1=# create user testread password 'test123';
CREATE ROLE
otus_dba1=# grant readonly to testread;
GRANT ROLE
otus_dba1=#

postgres@asvpg:~$ psql -h localhost -d otus_dba1 -U testread -p 5433
Password for user testread: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

otus_dba1=> \conninfo
            Connection Information
      Parameter       |         Value          
----------------------+------------------------
 Database             | otus_dba1
 Client User          | testread
 Host                 | localhost
 Host Address         | 127.0.0.1
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | true
 GSSAPI Authenticated | false
 Backend PID          | 7215
 SSL Connection       | true
 SSL Library          | OpenSSL
 SSL Protocol         | TLSv1.3
 SSL Key Bits         | 256
 SSL Cipher           | TLS_AES_256_GCM_SHA384
 SSL Compression      | false
 ALPN                 | postgresql
 Superuser            | off
 Hot Standby          | off
(19 rows)

otus_dba1=> select * from t1;
ERROR:  permission denied for table t1
otus_dba1=>
```

###
Таблица t1 по умолчанию создана в схеме public. Пользователь testread не имеет прав на данную схему.
###
```sh
otus_dba1=> \dp testnm.*
                            Access privileges
 Schema | Name | Type | Access privileges | Column privileges | Policies 
--------+------+------+-------------------+-------------------+----------
(0 rows)

otus_dba1=> \dp public.*
                                   Access privileges
 Schema |      Name      |   Type   | Access privileges | Column privileges | Policies 
--------+----------------+----------+-------------------+-------------------+----------
 public | t1             | table    |                   |                   | 
 public | tb_test        | table    |                   |                   | 
 public | tb_test_id_seq | sequence |                   |                   | 
(3 rows)

otus_dba1=> 
otus_dba1=> 
otus_dba1=> select current_schemas(true);
   current_schemas   
---------------------
 {pg_catalog,public}
(1 row)

otus_dba1=>
```

###
Пересоздайте таблицу как testnm.t1, проверьте select * from testnm.t1; и настройте поведение, чтобы обращение к t1 было предсказуемым
###
```sh
postgres@asvpg:~$ psql -d otus_dba1 -p 5433
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

otus_dba1=# drop table t1;
DROP TABLE
otus_dba1=#

postgres@asvpg:~$ psql -d otus_dba1 -p 5433
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

otus_dba1=# drop table t1;
DROP TABLE
otus_dba1=# create table testnm.t1 (c1 int);
CREATE TABLE
otus_dba1=# insert into testnm.t1 values (1);
INSERT 0 1
otus_dba1=# insert into testnm.t1 values (2);
INSERT 0 1
otus_dba1=# insert into testnm.t1 values (3);
INSERT 0 1
otus_dba1=#

otus_dba1=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

otus_dba1=# select current_schemas(true);
   current_schemas   
---------------------
 {pg_catalog,public}
(1 row)

otus_dba1=#

otus_dba1=# select * from testnm.t1;
 c1 
----
  1
  2
  3
(3 rows)

otus_dba1=# select * from t1;
ERROR:  relation "t1" does not exist
LINE 1: select * from t1;
                      ^
otus_dba1=#
```

###
Если явно не указать схему, то будет ошибка, т.к. СУБД будет искать таблицу не в схеме testnm.
###

###
Выполним еще раз выдачу прав на таблицы в схеме testnm и проверим привилегии: видим, что права на чтение роли readonly выдал пользователь postgres:
###
```sh
otus_dba1=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
otus_dba1=# \dp testnm.*
                                 Access privileges
 Schema | Name | Type  |     Access privileges      | Column privileges | Policies 
--------+------+-------+----------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxtm/postgres+|                   | 
        |      |       | readonly=r/postgres        |                   | 
(1 row)

otus_dba1=#
```

###
Меняем значение параметра search_path, указывая имя схемы testnm:
###
```sh
otus_dba1=# select name, setting, context from pg_settings where name = 'search_path';
    name     |     setting     | context 
-------------+-----------------+---------
 search_path | "$user", public | user
(1 row)

otus_dba1=# alter system set search_path = 'testnm';
ALTER SYSTEM

otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# select name, setting, context from pg_settings where name = 'search_path';
    name     |           setting           | context 
-------------+-----------------------------+---------
 search_path | testnm                      | user
(1 row)

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
 Backend PID          | 11090
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=#
```

###
Теперь даже без явного указания схемы запрос успешно выполняется:
###
```sh
otus_dba1=# select * from t1;
 c1 
----
  1
  2
  3
(3 rows)

otus_dba1=#
```

###
!!! вопрос: если в search_path указать не только testnm, запрос работать не будет. Ожидал, что проблем не будет. Почему такое поведение?
###


###
Под testread проверьте попытку create table и insert; при необходимости запретите и подтвердите запрет повторной проверкой
###
```sh
asvpg@asvpg:~$ psql -h localhost -d otus_dba1 -U testread -p 5433
Password for user testread: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

otus_dba1=> \conninfo
            Connection Information
      Parameter       |         Value          
----------------------+------------------------
 Database             | otus_dba1
 Client User          | testread
 Host                 | localhost
 Host Address         | 127.0.0.1
 Server Port          | 5433
 Options              | 
 Protocol Version     | 3.0
 Password Used        | true
 GSSAPI Authenticated | false
 Backend PID          | 25212
 SSL Connection       | true
 SSL Library          | OpenSSL
 SSL Protocol         | TLSv1.3
 SSL Key Bits         | 256
 SSL Cipher           | TLS_AES_256_GCM_SHA384
 SSL Compression      | false
 ALPN                 | postgresql
 Superuser            | off
 Hot Standby          | off
(19 rows)

otus_dba1=>
```

###
Если не добавлять ключа -h, то будет ошибка: psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5433" failed: FATAL:  Peer authentication failed for user "testread"
###

###
При попытке создания таблицы будет ошибка отсутствия прав, т.к. на схему testnm есть только права USAGE - а это не учитывает DDL:
###
```sh
otus_dba1=> create table t2 (c2 int);
ERROR:  permission denied for schema testnm
LINE 1: create table t2 (c2 int);
                     ^
otus_dba1=> 
otus_dba1=> 
otus_dba1=> create table testnm.t2 (c2 int);
ERROR:  permission denied for schema testnm
LINE 1: create table testnm.t2 (c2 int);
                     ^
otus_dba1=>
```

###
При попытке вставки в таблицу будет ошибка отсутствия прав, т.к. на схему testnm есть только права на SELECT
###
```sh
otus_dba1=> insert into t1 values (4);
ERROR:  permission denied for table t1
otus_dba1=>
```

###
Проведем эксперимент: временно добавим привилегию на INSERT для роли readonly. Убедимся, что в сочетании с USAGE вставка будет проходить:
###
```sh
postgres@asvpg:~$ psql -d otus_dba1
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

otus_dba1=# grant insert on all tables in schema testnm to readonly;
GRANT
otus_dba1=# \dp testnm.*
                                 Access privileges
 Schema | Name | Type  |     Access privileges      | Column privileges | Policies 
--------+------+-------+----------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxtm/postgres+|                   | 
        |      |       | readonly=ar/postgres       |                   | 
(1 row)

otus_dba1=#

postgres@asvpg:~$ psql -d otus_dba1 -U testread -p 5433
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5433" failed: FATAL:  Peer authentication failed for user "testread"
postgres@asvpg:~$ psql -d otus_dba1 -U testread -p 5433 -h localhost
Password for user testread: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

otus_dba1=> select * from t1;
 c1 
----
  1
  2
  3
(3 rows)

otus_dba1=> insert into t1 values (4);
INSERT 0 1
otus_dba1=> select * from t1;
 c1 
----
  1
  2
  3
  4
(4 rows)

otus_dba1=> 
```

###
Отнимем права на INSERT:
###
```sh
postgres@asvpg:~$ psql -d otus_dba1
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

otus_dba1=# revoke insert on all tables in schema testnm from readonly;
REVOKE
otus_dba1=# \dp testnm.*
                                 Access privileges
 Schema | Name | Type  |     Access privileges      | Column privileges | Policies 
--------+------+-------+----------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxtm/postgres+|                   | 
        |      |       | readonly=r/postgres        |                   | 
(1 row)

otus_dba1=# exit
postgres@asvpg:~$ psql -d otus_dba1 -U testread -p 5433 -h localhost
Password for user testread: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

otus_dba1=> select * from t1;
 c1 
----
  1
  2
  3
  4
(4 rows)

otus_dba1=> insert into t1 values (5);
ERROR:  permission denied for table t1
otus_dba1=>
```
