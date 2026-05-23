###
Для выполнения ДЗ создадим 2 таблицы и добавим в каждую несколько строк (для наглядности не будем вставлять много строк; если требуется - просьба сообщить):
###
```sh
postgres@asvpg:~$ psql -d otus_dba1 -p 5433
Password for user postgres: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

otus_dba1=# show search_path;
   search_path   
-----------------
 "$user", public
(1 row)

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
 Backend PID          | 229945
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=# 
otus_dba1=# create table clients (id int primary key, name varchar(32));
CREATE TABLE

otus_dba1=# create table orders (id int primary key, client_id int, product varchar(32));
CREATE TABLE
otus_dba1=#

otus_dba1=# create table orders (id int primary key, client_id int, product varchar(32));
CREATE TABLE
otus_dba1=#

otus_dba1=# insert into clients values (1, 'Ivanov Ivan'), (2, 'Pavlova Anna'), (3, 'Petrov Petr');
INSERT 0 3
otus_dba1=# 
otus_dba1=# insert into orders values (1, 1, 'Book'), (2, 1, 'Pen'), (3, 3, 'Pencil');
INSERT 0 3
otus_dba1=# select * from clients;
 id |     name     
----+--------------
  1 | Ivanov Ivan
  2 | Pavlova Anna
  3 | Petrov Petr
(3 rows)

otus_dba1=# select * from orders;
 id | client_id | product 
----+-----------+---------
  1 |         1 | Book
  2 |         1 | Pen
  3 |         3 | Pencil
(3 rows)

otus_dba1=#
```

###
Реализовать прямое соединение двух или более таблиц
###
```sh
--Прямое соединение (INNER JOIN) — способ объединения двух таблиц, при котором в результат попадают только те строки, для которых есть совпадения по ---указанному условию в объединяемых таблицах. Берутся строки из первой таблицы.
--Для каждой строки ищутся совпадающие строки во второй таблице по заданному условию (обычно по ключу).
--В итоговую выборку попадают только те пары строк, где совпадение найдено. Если совпадений нет — строка не попадает в результат.

otus_dba1=# select c.name, o.product from clients c join orders o on c.id = o.client_id;
    name     | product 
-------------+---------
 Ivanov Ivan | Book
 Ivanov Ivan | Pen
 Petrov Petr | Pencil
(3 rows)

--Для клиента с id = 2 нет товаров, поэтому информация о нем не попала в выборку.

otus_dba1=# explain (analyze, buffers)
otus_dba1-# select c.name, o.product from clients c join orders o on c.id = o.client_id;
QUERY PLAN                                                      
----------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=25.98..44.70 rows=690 width=164) (actual time=0.044..0.047 rows=3.00 loops=1)
   Hash Cond: (o.client_id = c.id)
   Buffers: shared hit=2
   ->  Seq Scan on orders o  (cost=0.00..16.90 rows=690 width=86) (actual time=0.013..0.014 rows=3.00 loops=1)
         Buffers: shared hit=1
   ->  Hash  (cost=17.10..17.10 rows=710 width=86) (actual time=0.017..0.017 rows=3.00 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         Buffers: shared hit=1
         ->  Seq Scan on clients c  (cost=0.00..17.10 rows=710 width=86) (actual time=0.005..0.009 rows=3.00 loops=1)
               Buffers: shared hit=1
 Planning Time: 0.167 ms
 Execution Time: 0.073 ms
(12 rows)
```

###
Реализовать левостороннее (или правостороннее) соединение двух или более таблиц
###
```sh
--LEFT JOIN и RIGHT JOIN — это типы внешних соединений, которые позволяют объединять строки из двух таблиц, даже если между ними нет совпадений по --ключу. Главное различие — в том, какая таблица считается 'основной' (выводятся все запрашиваемые строки), а какая — 'дополнительной'.
--LEFT JOIN: берёт все строки из левой таблицы (та, что указана первой в запросе). Добавляет к ним соответствующие строки из правой таблицы.
--Если совпадений нет, в столбцах правой таблицы будут NULL.
--RIGHT JOIN: берёт все строки из правой таблицы (та, что указана второй). Добавляет к ним соответствующие строки из левой таблицы.
--Если совпадений нет, в столбцах левой таблицы будут NULL.

otus_dba1=# select c.name, o.product from clients c left join orders o on c.id = o.client_id;   
     name     | product 
--------------+---------
 Ivanov Ivan  | Book
 Ivanov Ivan  | Pen
 Petrov Petr  | Pencil
 Pavlova Anna | 
(4 rows)

--Отличие от прямого соединения состоит в добавлении четвертой строки для клиента (Pavlova Anna), т.е. выводятся все клипенты, даже у которых нет товаров.
--В плане выполнения оптимизатор поменял таблицы в объединении местами и получился Right Join. Почему принято такое решение - не знаю. По возможности, просьба уточнить. Можно использовать хинт LEADING, чтобы указать нужный порядок таблиц в объединении.
QUERY PLAN                                                      
----------------------------------------------------------------------------------------------------------------------
 Hash Right Join  (cost=25.98..44.70 rows=710 width=164) (actual time=0.032..0.035 rows=4.00 loops=1)
   Hash Cond: (o.client_id = c.id)
   Buffers: shared hit=2
   ->  Seq Scan on orders o  (cost=0.00..16.90 rows=690 width=86) (actual time=0.002..0.002 rows=3.00 loops=1)
         Buffers: shared hit=1
   ->  Hash  (cost=17.10..17.10 rows=710 width=86) (actual time=0.015..0.015 rows=3.00 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         Buffers: shared hit=1
         ->  Seq Scan on clients c  (cost=0.00..17.10 rows=710 width=86) (actual time=0.012..0.012 rows=3.00 loops=1)
               Buffers: shared hit=1
 Planning Time: 0.084 ms
 Execution Time: 0.066 ms
(12 rows)

--Если поменяем местами таблицы в соединении, укажем RIGHT JOIN, будет тот же результат, что выше с LEFT JOIN:
otus_dba1=# select c.name, o.product from orders o right join clients c on c.id = o.client_id;
     name     | product 
--------------+---------
 Ivanov Ivan  | Book
 Ivanov Ivan  | Pen
 Petrov Petr  | Pencil
 Pavlova Anna | 
(4 rows)

otus_dba1=#
```

###
Реализовать кросс соединение двух или более таблиц
###
```sh
--Кросс-соединение (или декартово произведение) — это тип соединения, при котором каждая строка одной таблицы объединяется с каждой строкой другой (других) таблицы. В результате количество строк в выборке равно произведению количества строк во всех участвующих таблицах.
--Обычно используется для генерации тестовых данных без указания условия. В данном случае в результате будет 3 * 3 = 9 строк. 

otus_dba1=# select c.name, o.product from clients c cross join orders o;
     name     | product 
--------------+---------
 Ivanov Ivan  | Book
 Ivanov Ivan  | Pen
 Ivanov Ivan  | Pencil
 Pavlova Anna | Book
 Pavlova Anna | Pen
 Pavlova Anna | Pencil
 Petrov Petr  | Book
 Petrov Petr  | Pen
 Petrov Petr  | Pencil
(9 rows)
QUERY PLAN                                                      
---------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.00..6159.48 rows=489900 width=164) (actual time=0.021..0.029 rows=9.00 loops=1)
   Buffers: shared hit=2
   ->  Seq Scan on clients c  (cost=0.00..17.10 rows=710 width=82) (actual time=0.011..0.012 rows=3.00 loops=1)
         Buffers: shared hit=1
   ->  Materialize  (cost=0.00..20.35 rows=690 width=82) (actual time=0.003..0.004 rows=3.00 loops=3)
         Storage: Memory  Maximum Storage: 17kB
         Buffers: shared hit=1
         ->  Seq Scan on orders o  (cost=0.00..16.90 rows=690 width=82) (actual time=0.004..0.005 rows=3.00 loops=1)
               Buffers: shared hit=1
 Planning Time: 0.070 ms
 Execution Time: 0.046 ms
(11 rows)
```

###
Реализовать полное соединение двух или более таблиц
###
```sh
--Полное соединение (FULL JOIN\FULL OUTER JOIN) — это тип объединения двух таблиц, при котором в результат попадают все строки из левой таблицы и
--все строки из правой таблицы. Если для какой-то строки нет совпадения в другой таблице, в соответствующих столбцах будут значения NULL. Т.е., это -комбинация LEFT JOIN и RIGHT JOIN.
--Обычно используется, когда требуется увидеть все данные из обеих таблиц, даже если между ними нет связей.

otus_dba1=# select c.name, o.product from clients c full join orders o on c.id = o.client_id;
     name     | product 
--------------+---------
 Ivanov Ivan  | Book
 Ivanov Ivan  | Pen
 Petrov Petr  | Pencil
 Pavlova Anna | 
(4 rows)

otus_dba1=#

otus_dba1=# explain (analyze, buffers) select c.name, o.product from clients c full join orders o on c.id = o.client_id;
QUERY PLAN                                                      
----------------------------------------------------------------------------------------------------------------------
 Hash Full Join  (cost=25.98..44.70 rows=710 width=164) (actual time=0.028..0.035 rows=4.00 loops=1)
   Hash Cond: (o.client_id = c.id)
   Buffers: shared hit=2
   ->  Seq Scan on orders o  (cost=0.00..16.90 rows=690 width=86) (actual time=0.003..0.003 rows=3.00 loops=1)
         Buffers: shared hit=1
   ->  Hash  (cost=17.10..17.10 rows=710 width=86) (actual time=0.019..0.020 rows=3.00 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         Buffers: shared hit=1
         ->  Seq Scan on clients c  (cost=0.00..17.10 rows=710 width=86) (actual time=0.014..0.015 rows=3.00 loops=1)
               Buffers: shared hit=1
 Planning Time: 0.117 ms
 Execution Time: 0.060 ms
(12 rows)
```

###
Реализовать запрос, в котором будут использованы разные типы соединений
###
```sh
--Сделал через UNION ALL на тех же 2 таблицах. Если требуется что то другое, просьба сообщить.
otus_dba1=# select 'INNER JOIN' as join_type, c.name, o.product from clients c join orders o on c.id = o.client_id
union all
select 'LEFT JOIN' as join_type, c.name, o.product from clients c left join orders o on c.id = o.client_id
union all
select 'CROSS JOIN' as join_type, c.name, o.product from clients c cross join orders o;
 join_type  |     name     | product 
------------+--------------+---------
 INNER JOIN | Ivanov Ivan  | Book
 INNER JOIN | Ivanov Ivan  | Pen
 INNER JOIN | Petrov Petr  | Pencil
 LEFT JOIN  | Ivanov Ivan  | Book
 LEFT JOIN  | Ivanov Ivan  | Pen
 LEFT JOIN  | Petrov Petr  | Pencil
 LEFT JOIN  | Pavlova Anna | 
 CROSS JOIN | Ivanov Ivan  | Book
 CROSS JOIN | Ivanov Ivan  | Pen
 CROSS JOIN | Ivanov Ivan  | Pencil
 CROSS JOIN | Pavlova Anna | Book
 CROSS JOIN | Pavlova Anna | Pen
 CROSS JOIN | Pavlova Anna | Pencil
 CROSS JOIN | Petrov Petr  | Book
 CROSS JOIN | Petrov Petr  | Pen
 CROSS JOIN | Petrov Petr  | Pencil
(16 rows)

otus_dba1=#
--3 + 4 + 9 строк, соответственно
```
