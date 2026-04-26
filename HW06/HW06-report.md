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

###
Вставляем 3 строки в первой сесиии:
###
```sh
otus_dba1=# insert into orders values (1, now(), 100);
INSERT 0 1
otus_dba1=*# insert into orders values (2, now(), 200);
INSERT 0 1
otus_dba1=*# insert into orders values (3, now(), 300);
INSERT 0 1
otus_dba1=*# commit;
COMMIT
otus_dba1=# select * from orders;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-04-26 12:41:32.560266+03 |    100
  2 | 2026-04-26 12:41:32.560266+03 |    200
  3 | 2026-04-26 12:41:32.560266+03 |    300
(3 rows)

otus_dba1=*# commit;
COMMIT
otus_dba1=#
```

###
После коммита вторая сессия также увидит данные:
###
```sh
otus_dba1=# select * from orders;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-04-26 12:41:32.560266+03 |    100
  2 | 2026-04-26 12:41:32.560266+03 |    200
  3 | 2026-04-26 12:41:32.560266+03 |    300
(3 rows)

otus_dba1=*# commit;
COMMIT
otus_dba1=#
```

###
Read committed: в сессии 2 начните транзакцию и выполните select count(*), sum(amount) ... за последнюю минуту; в сессии 1 вставьте новый заказ и выполните commit; в сессии 2 повторите тот же select и зафиксируйте изменение; завершите транзакции;
###
```sh
--Проверяем используемый уровень изоляции в БД:
otus_dba1=*# show transaction_isolation;
 transaction_isolation 
-----------------------
 read committed
(1 row)

otus_dba1=*# commit;
COMMIT
otus_dba1=#

--вторая сессия:
otus_dba1=# select count(*), sum(amount) from orders;
 count | sum 
-------+-----
     3 | 600
(1 row)
otus_dba1=#

--первая сессия:
otus_dba1=# insert into orders values (4, now(), 400);
INSERT 0 1
otus_dba1=*# commit;
COMMIT
otus_dba1=# select * from orders;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-04-26 12:41:32.560266+03 |    100
  2 | 2026-04-26 12:41:32.560266+03 |    200
  3 | 2026-04-26 12:41:32.560266+03 |    300
  4 | 2026-04-26 12:54:47.520813+03 |    400
(4 rows)

otus_dba1=*# commit;
COMMIT
otus_dba1=*#

--вторая сессия:
otus_dba1=*# select count(*), sum(amount) from orders;
 count | sum  
-------+------
     4 | 1000
(1 row)

otus_dba1=*# commit;
COMMIT
otus_dba1=#
```
###
На уровне изоляции READ COMMITTED снимок создается вначале каждого оператора транзакции и остается активным все время работы этого оператора.
###

###
Repeatable read: повторите сценарий, но обе транзакции начните с set transaction isolation level repeatable read;; сравните результаты внутри транзакции и после её завершения;
###
```sh
--первая сессия:
otus_dba1=# set transaction isolation level repeatable read;
SET
otus_dba1=*# show transaction_isolation;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)

otus_dba1=*#

--вторая сессия:
otus_dba1=# set transaction isolation level repeatable read;
SET
otus_dba1=*# show transaction_isolation;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)

otus_dba1=*# select count(*), sum(amount) from orders;
 count | sum  
-------+------
     4 | 1000
(1 row)

otus_dba1=*#

--первая сессия:
otus_dba1=*# insert into orders values (5, now(), 500);
INSERT 0 1
otus_dba1=*# select * from orders;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-04-26 12:41:32.560266+03 |    100
  2 | 2026-04-26 12:41:32.560266+03 |    200
  3 | 2026-04-26 12:41:32.560266+03 |    300
  4 | 2026-04-26 12:54:47.520813+03 |    400
  5 | 2026-04-26 13:02:06.902634+03 |    500
(5 rows)

otus_dba1=*#
otus_dba1=*# commit;
COMMIT
otus_dba1=# select * from orders;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-04-26 12:41:32.560266+03 |    100
  2 | 2026-04-26 12:41:32.560266+03 |    200
  3 | 2026-04-26 12:41:32.560266+03 |    300
  4 | 2026-04-26 12:54:47.520813+03 |    400
  5 | 2026-04-26 13:02:06.902634+03 |    500
(5 rows)

otus_dba1=*#

--вторая сессия:
otus_dba1=*# select count(*), sum(amount) from orders;
 count | sum  
-------+------
     4 | 1000
(1 row)

otus_dba1=*#

otus_dba1=*# commit;
COMMIT
otus_dba1=# select count(*), sum(amount) from orders;
 count | sum  
-------+------
     5 | 1500
(1 row)

otus_dba1=*# commit;
COMMIT
otus_dba1=#
```

###
На уровне изоляции REPEATABLE READ снимок создается один раз в начале первого оператора транзакции и остается активным до самого конца транзакции.
Таким образом, в отличие от уровня изоляции READ COMMITTED, видимость новой строки из первой сессии во второй сессии появится только после начала новой транзакции. До этого будет возвращен предыдущий наьбор строк, котоырй актуален на момент начала транзакции во второй сессии.
###

###
В отчёте укажите, где и почему меняются значения, и какой уровень изоляции нужен для отчёта;
###

####
Для согласованного отчета при параллельной записи данных рекомендуется использовать уровень изоляции REPEATABLE READ, а не READ COMMITTED.
####
