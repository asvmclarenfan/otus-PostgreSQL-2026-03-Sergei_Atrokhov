
###
В БД test_db создать схему my_schema и две одинаковые таблицы (table1, table2).
###
```sh
--используем ранее созданную БД otus_dba1:
otus_dba1=# \l
                                                      List of databases
    Name     |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |   Access privileges   
-------------+----------+----------+-----------------+-------------+-------------+--------+-----------+-----------------------
 otus_dba1   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =Tc/postgres         +
             |          |          |                 |             |             |        |           | postgres=CTc/postgres+
             |          |          |                 |             |             |        |           | readonly=c/postgres
 pgbenchtest | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 postgres    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 template0   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
             |          |          |                 |             |             |        |           | postgres=CTc/postgres
 template1   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
             |          |          |                 |             |             |        |           | postgres=CTc/postgres
(5 rows)

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
 Password Used        | true
 GSSAPI Authenticated | false
 Backend PID          | 5322
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=#

--создадим схему и в текущей сессии сделаем ее по умолчанию:
otus_dba1=# create schema my_schema;
CREATE SCHEMA
otus_dba1=# \dn
        List of schemas
   Name    |       Owner       
-----------+-------------------
 my_schema | postgres
 public    | pg_database_owner
 testnm    | postgres
(3 rows)

otus_dba1=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

otus_dba1=# set search_path = my_schema;
SET
otus_dba1=# show search_path;
 search_path 
-------------
 my_schema
(1 row)

otus_dba1=#

--создадим две одинаковые таблицы (table1, table2) в схеме my_schema:
otus_dba1=# create table table1 (
id int,
name varchar(100),
currtime timestamp default now());
CREATE TABLE
otus_dba1=# 
otus_dba1=# create table table2 (
id int,
name varchar(100),
currtime timestamp default now());
CREATE TABLE
otus_dba1=# \dt
            List of tables
  Schema   |  Name  | Type  |  Owner   
-----------+--------+-------+----------
 my_schema | table1 | table | postgres
 my_schema | table2 | table | postgres
(2 rows)

otus_dba1=#
```

###
Заполнить table1 100 строками с помощью generate_series.
###
```sh
otus_dba1=# insert into table1 (id, name)
select generate_series(1, 100), 'Number of id ' || generate_series(1, 100);
INSERT 0 100
otus_dba1=# select * from table1 limit 1;
 id |      name      |          currtime          
----+----------------+----------------------------
  1 | Number of id 1 | 2026-06-13 15:03:57.744208
(1 row)

otus_dba1=# select COUNT(*) from table1;
 count 
-------
   100
(1 row)

otus_dba1=# select COUNT(*) from table2;
 count 
-------
     0
(1 row)

otus_dba1=#
```

###
Создать каталог /var/lib/postgresql/backups/ под пользователем postgres
###
```sh
postgres@asvpg:~$ cd /var/lib/postgresql
postgres@asvpg:/var/lib/postgresql$ ls -altr
total 12
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 18
drwxr-xr-x 74 root     root     4096 Apr  9 15:16 ..
drwxr-xr-x  3 postgres postgres 4096 Apr 10 14:50 .
postgres@asvpg:/var/lib/postgresql$ mkdir -p backups/
postgres@asvpg:/var/lib/postgresql$ ls -altr
total 16
drwxr-xr-x  3 postgres postgres 4096 Apr  7 18:22 18
drwxr-xr-x 74 root     root     4096 Apr  9 15:16 ..
drwxrwxr-x  2 postgres postgres 4096 Jun 13 15:05 backups
drwxr-xr-x  4 postgres postgres 4096 Jun 13 15:05 .
postgres@asvpg:/var/lib/postgresql$
```

###
Бэкап через COPY: Выгрузить table1 в CSV командой \copy
###
```sh
otus_dba1=# copy table1  to '/var/lib/postgresql/backups/table1.csv' with delimiter ',' ;
COPY 100
otus_dba1=#

--проверяем:
postgres@asvpg:/var/lib/postgresql$ cd backups/
postgres@asvpg:/var/lib/postgresql/backups$ ls -altr
total 16
drwxr-xr-x 4 postgres postgres 4096 Jun 13 15:05 ..
drwxrwxr-x 2 postgres postgres 4096 Jun 13 15:09 .
-rw-r--r-- 1 postgres postgres 4584 Jun 13 15:09 table1.csv
postgres@asvpg:/var/lib/postgresql/backups$ cat table1.csv
1,Number of id 1,2026-06-13 15:03:57.744208
2,Number of id 2,2026-06-13 15:03:57.744208
3,Number of id 3,2026-06-13 15:03:57.744208
4,Number of id 4,2026-06-13 15:03:57.744208
5,Number of id 5,2026-06-13 15:03:57.744208
6,Number of id 6,2026-06-13 15:03:57.744208
7,Number of id 7,2026-06-13 15:03:57.744208
8,Number of id 8,2026-06-13 15:03:57.744208
9,Number of id 9,2026-06-13 15:03:57.744208
10,Number of id 10,2026-06-13 15:03:57.744208
11,Number of id 11,2026-06-13 15:03:57.744208
12,Number of id 12,2026-06-13 15:03:57.744208
13,Number of id 13,2026-06-13 15:03:57.744208
14,Number of id 14,2026-06-13 15:03:57.744208
15,Number of id 15,2026-06-13 15:03:57.744208
16,Number of id 16,2026-06-13 15:03:57.744208
17,Number of id 17,2026-06-13 15:03:57.744208
18,Number of id 18,2026-06-13 15:03:57.744208
19,Number of id 19,2026-06-13 15:03:57.744208
20,Number of id 20,2026-06-13 15:03:57.744208
21,Number of id 21,2026-06-13 15:03:57.744208
22,Number of id 22,2026-06-13 15:03:57.744208
23,Number of id 23,2026-06-13 15:03:57.744208
24,Number of id 24,2026-06-13 15:03:57.744208
25,Number of id 25,2026-06-13 15:03:57.744208
26,Number of id 26,2026-06-13 15:03:57.744208
27,Number of id 27,2026-06-13 15:03:57.744208
28,Number of id 28,2026-06-13 15:03:57.744208
29,Number of id 29,2026-06-13 15:03:57.744208
30,Number of id 30,2026-06-13 15:03:57.744208
31,Number of id 31,2026-06-13 15:03:57.744208
32,Number of id 32,2026-06-13 15:03:57.744208
33,Number of id 33,2026-06-13 15:03:57.744208
34,Number of id 34,2026-06-13 15:03:57.744208
35,Number of id 35,2026-06-13 15:03:57.744208
36,Number of id 36,2026-06-13 15:03:57.744208
37,Number of id 37,2026-06-13 15:03:57.744208
38,Number of id 38,2026-06-13 15:03:57.744208
39,Number of id 39,2026-06-13 15:03:57.744208
40,Number of id 40,2026-06-13 15:03:57.744208
41,Number of id 41,2026-06-13 15:03:57.744208
42,Number of id 42,2026-06-13 15:03:57.744208
43,Number of id 43,2026-06-13 15:03:57.744208
44,Number of id 44,2026-06-13 15:03:57.744208
45,Number of id 45,2026-06-13 15:03:57.744208
46,Number of id 46,2026-06-13 15:03:57.744208
47,Number of id 47,2026-06-13 15:03:57.744208
48,Number of id 48,2026-06-13 15:03:57.744208
49,Number of id 49,2026-06-13 15:03:57.744208
50,Number of id 50,2026-06-13 15:03:57.744208
51,Number of id 51,2026-06-13 15:03:57.744208
52,Number of id 52,2026-06-13 15:03:57.744208
53,Number of id 53,2026-06-13 15:03:57.744208
54,Number of id 54,2026-06-13 15:03:57.744208
55,Number of id 55,2026-06-13 15:03:57.744208
56,Number of id 56,2026-06-13 15:03:57.744208
57,Number of id 57,2026-06-13 15:03:57.744208
58,Number of id 58,2026-06-13 15:03:57.744208
59,Number of id 59,2026-06-13 15:03:57.744208
60,Number of id 60,2026-06-13 15:03:57.744208
61,Number of id 61,2026-06-13 15:03:57.744208
62,Number of id 62,2026-06-13 15:03:57.744208
63,Number of id 63,2026-06-13 15:03:57.744208
64,Number of id 64,2026-06-13 15:03:57.744208
65,Number of id 65,2026-06-13 15:03:57.744208
66,Number of id 66,2026-06-13 15:03:57.744208
67,Number of id 67,2026-06-13 15:03:57.744208
68,Number of id 68,2026-06-13 15:03:57.744208
69,Number of id 69,2026-06-13 15:03:57.744208
70,Number of id 70,2026-06-13 15:03:57.744208
71,Number of id 71,2026-06-13 15:03:57.744208
72,Number of id 72,2026-06-13 15:03:57.744208
73,Number of id 73,2026-06-13 15:03:57.744208
74,Number of id 74,2026-06-13 15:03:57.744208
75,Number of id 75,2026-06-13 15:03:57.744208
76,Number of id 76,2026-06-13 15:03:57.744208
77,Number of id 77,2026-06-13 15:03:57.744208
78,Number of id 78,2026-06-13 15:03:57.744208
79,Number of id 79,2026-06-13 15:03:57.744208
80,Number of id 80,2026-06-13 15:03:57.744208
81,Number of id 81,2026-06-13 15:03:57.744208
82,Number of id 82,2026-06-13 15:03:57.744208
83,Number of id 83,2026-06-13 15:03:57.744208
84,Number of id 84,2026-06-13 15:03:57.744208
85,Number of id 85,2026-06-13 15:03:57.744208
86,Number of id 86,2026-06-13 15:03:57.744208
87,Number of id 87,2026-06-13 15:03:57.744208
88,Number of id 88,2026-06-13 15:03:57.744208
89,Number of id 89,2026-06-13 15:03:57.744208
90,Number of id 90,2026-06-13 15:03:57.744208
91,Number of id 91,2026-06-13 15:03:57.744208
92,Number of id 92,2026-06-13 15:03:57.744208
93,Number of id 93,2026-06-13 15:03:57.744208
94,Number of id 94,2026-06-13 15:03:57.744208
95,Number of id 95,2026-06-13 15:03:57.744208
96,Number of id 96,2026-06-13 15:03:57.744208
97,Number of id 97,2026-06-13 15:03:57.744208
98,Number of id 98,2026-06-13 15:03:57.744208
99,Number of id 99,2026-06-13 15:03:57.744208
100,Number of id 100,2026-06-13 15:03:57.744208
postgres@asvpg:/var/lib/postgresql/backups$
```

###
Восстановление из COPY: Загрузить данные из CSV в table2
###
```sh
otus_dba1=# copy table2 from '/var/lib/postgresql/backups/table1.csv' with delimiter ',' ;
COPY 100
otus_dba1=# select COUNT(*) from table2;
 count 
-------
   100
(1 row)

otus_dba1=# select * from table2 limit 10;
 id |      name       |          currtime          
----+-----------------+----------------------------
  1 | Number of id 1  | 2026-06-13 15:03:57.744208
  2 | Number of id 2  | 2026-06-13 15:03:57.744208
  3 | Number of id 3  | 2026-06-13 15:03:57.744208
  4 | Number of id 4  | 2026-06-13 15:03:57.744208
  5 | Number of id 5  | 2026-06-13 15:03:57.744208
  6 | Number of id 6  | 2026-06-13 15:03:57.744208
  7 | Number of id 7  | 2026-06-13 15:03:57.744208
  8 | Number of id 8  | 2026-06-13 15:03:57.744208
  9 | Number of id 9  | 2026-06-13 15:03:57.744208
 10 | Number of id 10 | 2026-06-13 15:03:57.744208
(10 rows)

otus_dba1=#
```

###
Бэкап через pg_dump: создать кастомный сжатый дамп (-Fc) только схемы my_schema
###
```sh

```
