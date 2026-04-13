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


```
