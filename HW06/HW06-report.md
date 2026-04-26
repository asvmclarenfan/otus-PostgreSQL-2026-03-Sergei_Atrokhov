###
Откройте 2 сессии psql к PostgreSQL 17 и отключите autocommit;
###
```sh
postgres@asvpg:~$ psql -d otus_dba1 -p 5433
Password for user postgres: 
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
 Password Used        | true
 GSSAPI Authenticated | false
 Backend PID          | 12012
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=# \set AUTOCOMMIT OFF
otus_dba1=#

postgres@asvpg:~$ psql -d otus_dba1 -p 5433
Password for user postgres: 
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
 Password Used        | true
 GSSAPI Authenticated | false
 Backend PID          | 12586
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=# \set AUTOCOMMIT OFF
otus_dba1=#
```

###
Создайте таблицу orders(id serial, created_at timestamptz, amount numeric) и вставьте 2–3 записи;
###
###
В первой сессии создадим таблицу:
###
```sh
otus_dba1=# create table orders(id serial, created_at timestamptz, amount numeric);
CREATE TABLE
otus_dba1=*#
```

###
В отличие от Oracle, в PostgreSQL DDL команды являются транзакционными, без неявного коммита, т.е. во второй сессии таблица не будет видна:
###
```sh
otus_dba1=*# select * from orders;
ERROR:  relation "orders" does not exist
LINE 1: select * from orders;
                      ^
otus_dba1=!#
```

###
В первой сессии выполним коммит:
###
```sh
otus_dba1=*# commit;
COMMIT
otus_dba1=#
```

###
Во второй сессии после завершения текущей транзакции (откат), таблица будет видна:
###
```sh
otus_dba1=!# select * from orders;
ERROR:  current transaction is aborted, commands ignored until end of transaction block
otus_dba1=!# rollback;
ROLLBACK
otus_dba1=# select * from orders;
 id | created_at | amount 
----+------------+--------
(0 rows)

otus_dba1=*#
```

###
Как понимаю, в PostgreSQL есть особенность: если в открытой транзакции какой то стейтмент завершен неуспешно (например, синтаксическая ошибка), то 
необходимо выполнить откат транзакции, коммит не пройдет. Связано с правилами видимости различных версий строк?
###

