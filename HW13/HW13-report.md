###
Выбор таблицы для секционирования:
Основной акцент делается на секционировании таблицы bookings. Но вы можете выбрать и другие таблицы, если видите в этом смысл для оптимизации производительности (например, flights, boarding_passes).
Обоснуйте свой выбор: почему именно эта таблица требует секционирования? Какой тип данных является ключевым для секционирования?

Определение типа секционирования:
Определитесь с типом секционирования, которое наилучшим образом подходит для ваших данных:

По диапазону (например, по дате бронирования или дате рейса).
По списку (например, по пунктам отправления или по номерам рейсов).
По хэшированию (для равномерного распределения данных).
###
```sh
--Устанавливаем демонстрационную БД demo на 6 месяцев, проверяем список таблиц и число строк в каждой:
postgres@asvpg:~$ psql 
Password for user postgres: 
psql (18.3 (Ubuntu 18.3-1.pgdg24.04+1))
Type "help" for help.

postgres=# \l
                                                      List of databases
    Name     |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |   Access privileges   
-------------+----------+----------+-----------------+-------------+-------------+--------+-----------+-----------------------
 demo        | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 otus_dba1   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =Tc/postgres         +
             |          |          |                 |             |             |        |           | postgres=CTc/postgres+
             |          |          |                 |             |             |        |           | readonly=c/postgres
 pgbenchtest | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 postgres    | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | 
 template0   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
             |          |          |                 |             |             |        |           | postgres=CTc/postgres
 template1   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
             |          |          |                 |             |             |        |           | postgres=CTc/postgres
(6 rows)

postgres=# \c demo
You are now connected to database "demo" as user "postgres".

demo=# select * from pg_namespace;
  oid   |      nspname       | nspowner |                            nspacl                             
--------+--------------------+----------+---------------------------------------------------------------
     99 | pg_toast           |       10 | 
     11 | pg_catalog         |       10 | {postgres=UC/postgres,=U/postgres}
   2200 | public             |     6171 | {pg_database_owner=UC/pg_database_owner,=U/pg_database_owner}
  13309 | information_schema |       10 | {postgres=UC/postgres,=U/postgres}
 114712 | bookings           |       10 | 
(5 rows)

demo=# select tablename from pg_tables where schemaname = 'bookings';
    tablename    
-----------------
 airports_data
 seats
 airplanes_data
 tickets
 flights
 bookings
 routes
 segments
 boarding_passes
(9 rows)

demo=# select relname, relpages, reltuples from pg_class where relname in (select tablename from pg_tables where schemaname = 'bookings');
     relname     | relpages |  reltuples   
-----------------+----------+--------------
 airports_data   |      155 |         5501
 seats           |       10 |         1741
 airplanes_data  |        1 |           10
 tickets         |    57816 | 5.598473e+06
 flights         |      368 |        37671
 bookings        |    15856 |  2.48671e+06
 routes          |       34 |         2030
 segments        |    61256 | 7.349504e+06
 boarding_passes |    49800 | 5.982418e+06
(9 rows)

demo=#

--Если изучать содержимое pg_stats, то таблица bookings - хороший кандидат для партиционирования, а столбец book_date - для ключа партиционирования.
--Селективность столбца высокая (n_distinct = -1)!

--Демонстрационная БД выбрана за пол-года, посмотрим на число строк за каждый месяц:
demo=# select DATE_TRUNC('month', book_date), COUNT(*) from bookings.bookings group by DATE_TRUNC('month', book_date) order by 1;
       date_trunc       | count  
------------------------+--------
 2025-09-01 00:00:00+03 | 446319
 2025-10-01 00:00:00+03 | 434287
 2025-11-01 00:00:00+03 | 410680
 2025-12-01 00:00:00+03 | 411025
 2026-01-01 00:00:00+03 | 412089
 2026-02-01 00:00:00+03 | 370770
 2026-03-01 00:00:00+03 |   1540
(7 rows)

demo=#

--Предлагается выбрать архитектуру RANGE партиционирования, с шагом 1 месяц. 
```

###
Создание секционированной таблицы:
Преобразуйте таблицу в секционированную с выбранным типом секционирования.
Например, если вы выбрали секционирование по диапазону дат бронирования, создайте секции по месяцам или годам.
###
```sh
demo=# CREATE TABLE bookings.bookings_copy
(
book_ref char(6) not null,
book_date timestamptz not null,
total_amount numeric(10,2) not null
) PARTITION BY RANGE (book_date);
CREATE TABLE
demo=

--Создаем PK и помним, что в ванильном PostgreSQL нет глобальных индексов (есть, например в форке Pangolin), поэтому необходимо включать ключ партиционирования, иначе будет ошибка:
demo=# alter table bookings.bookings_copy add constraint bookings_pk primary key (book_ref);
ERROR:  unique constraint on partitioned table must include all partitioning columns
DETAIL:  PRIMARY KEY constraint on table "bookings_copy" lacks column "book_date" which is part of the partition key.

demo=# alter table bookings.bookings_copy add constraint bookings_pk primary key (book_ref, book_date);
ALTER TABLE
demo=#
--других индексов на исходную таблицу нет

--Создаем секции:
demo=# CREATE TABLE bookings.bookings_copy_2025_09 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2025-09-01') TO ('2025-10-01');
CREATE TABLE bookings.bookings_copy_2025_10 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');
CREATE TABLE bookings.bookings_copy_2025_11 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');
CREATE TABLE bookings.bookings_copy_2025_12 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');
CREATE TABLE bookings.bookings_copy_2026_01 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE bookings.bookings_copy_2026_02 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
CREATE TABLE bookings.bookings_copy_2026_03 PARTITION OF bookings.bookings_copy FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
demo=#

--Проверяем связь:
demo=# SELECT parentrelid::regclass AS parent_name, relid::regclass AS part_name, isleaf, level
FROM pg_partition_tree('bookings_copy');
  parent_name  |       part_name       | isleaf | level 
---------------+-----------------------+--------+-------
               | bookings_copy         | f      |     0
 bookings_copy | bookings_copy_2025_09 | t      |     1
 bookings_copy | bookings_copy_2025_10 | t      |     1
 bookings_copy | bookings_copy_2025_11 | t      |     1
 bookings_copy | bookings_copy_2025_12 | t      |     1
 bookings_copy | bookings_copy_2026_01 | t      |     1
 bookings_copy | bookings_copy_2026_02 | t      |     1
 bookings_copy | bookings_copy_2026_03 | t      |     1
(8 rows)

demo=#
```

###
Миграция данных:

Перенесите существующие данные из исходной таблицы в секционированную структуру.
Убедитесь, что все данные правильно распределены по секциям.
###
```sh
demo=# insert into bookings.bookings_copy select * from bookings.bookings;
INSERT 0 2486710
demo=#

--проверяем число строк в исходной\целевой таблицах:
demo=# select COUNT(*) from bookings.bookings;
  count  
---------
 2486710
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy;
  count  
---------
 2486710
(1 row)

demo=#

--проверяем распределение по секциям:
demo=# select COUNT(*) from bookings.bookings_copy_2025_09;
 count  
--------
 446319
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy_2025_10;
 count  
--------
 434287
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy_2025_11;
 count  
--------
 410680
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy_2025_12;
 count  
--------
 411025
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy_2026_01;
 count  
--------
 412089
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy_2026_02;
 count  
--------
 370770
(1 row)

demo=# select COUNT(*) from bookings.bookings_copy_2026_03;
 count 
-------
  1540
(1 row)

demo=#

--совпадает с исходной таблицей:
demo=# select DATE_TRUNC('month', book_date), COUNT(*) from bookings.bookings group by DATE_TRUNC('month', book_date) order by 1;
       date_trunc       | count  
------------------------+--------
 2025-09-01 00:00:00+03 | 446319
 2025-10-01 00:00:00+03 | 434287
 2025-11-01 00:00:00+03 | 410680
 2025-12-01 00:00:00+03 | 411025
 2026-01-01 00:00:00+03 | 412089
 2026-02-01 00:00:00+03 | 370770
 2026-03-01 00:00:00+03 |   1540
(7 rows)
```

###
Оптимизация запросов:

Проверьте, как секционирование влияет на производительность запросов. Выполните несколько выборок данных до и после секционирования для оценки времени выполнения.
Оптимизируйте запросы при необходимости (например, добавьте индексы на ключевые столбцы).
###
```sh

demo=# select * from bookings.bookings limit 10;
 book_ref |           book_date           | total_amount 
----------+-------------------------------+--------------
 2EW1SQ   | 2025-09-01 03:00:12.557744+03 |      8125.00
 3ZY3O5   | 2025-09-03 03:25:35.278453+03 |      7500.00
 756UAS   | 2025-09-10 19:24:02.978559+03 |      6325.00
 GC7I6S   | 2025-09-03 03:26:37.085293+03 |     18000.00
 RBUUIQ   | 2025-09-01 03:00:33.126206+03 |     13000.00
 GB7E53   | 2025-09-10 18:58:23.948989+03 |     11000.00
 6TAS39   | 2025-09-01 03:00:06.265219+03 |      5000.00
 UEX925   | 2025-09-03 03:27:39.815145+03 |     19550.00
 GGY5EU   | 2025-09-01 03:00:07.565193+03 |      5750.00
 MS148V   | 2025-09-03 03:29:12.70515+03  |      6750.00
(10 rows)

demo=# 

--найдем запись для конкретной book_date на исходной таблице:
demo=# explain (analyze, buffers) select * from bookings.bookings where book_date = '2025-09-03 03:27:39.815145+03';
                                                        QUERY PLAN                                                         
---------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..29807.71 rows=1 width=21) (actual time=0.355..37.513 rows=1.00 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   Buffers: shared hit=15856
   ->  Parallel Seq Scan on bookings  (cost=0.00..28807.61 rows=1 width=21) (actual time=11.855..23.009 rows=0.33 loops=3)
         Filter: (book_date = '2025-09-03 03:27:39.815145+03'::timestamp with time zone)
         Rows Removed by Filter: 828903
         Buffers: shared hit=15856
 Planning Time: 0.076 ms
 Execution Time: 37.532 ms
(10 rows)

demo=#

--сделаем то же самое для целевой таблицы с нвоой архитектурой:
demo=# explain (analyze, buffers) select * from bookings.bookings_copy where book_date = '2025-09-03 03:27:39.815145+03';
                                                                    QUERY PLAN                                                                     
---------------------------------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..7124.86 rows=1 width=21) (actual time=0.359..17.134 rows=1.00 loops=1)
   Workers Planned: 1
   Workers Launched: 1
   Buffers: shared hit=2843
   ->  Parallel Seq Scan on bookings_copy_2025_09 bookings_copy  (cost=0.00..6124.76 rows=1 width=21) (actual time=0.008..6.834 rows=0.50 loops=2)
         Filter: (book_date = '2025-09-03 03:27:39.815145+03'::timestamp with time zone)
         Rows Removed by Filter: 223159
         Buffers: shared hit=2843
 Planning Time: 0.142 ms
 Execution Time: 17.158 ms
(10 rows)

demo=#

--Видим уменьшение времени выполнения в 2 раза; в целевой таблице оптимизатор использует оптимизацию partition pruning (сканирование идет не по всем партцииям, а по конкретной - bookings_copy_2025_09); число обработанных блоков данных\страниц стало меньше примерно в 4 раза, но в первом случае использовалось 2 параллельных воркера, а во втором случае - один; те же цифры имеем на объем фильтрации ненужных строк.

--Если сравнивать параметры обращения по PK, то для текущих объемом (менее 3 млн строк), они сопоставимы. Но если число строк будет сильно больше, то число чтений по индексу будет увеличиваться (пусть и не кардинально) и сканирование по локальному PK конкретной партиции будет эффективнее:
demo=# explain (analyze, buffers) select * from bookings.bookings where book_ref = 'UEX925' and book_date = '2025-09-03 03:27:39.815145+03';
                                                         QUERY PLAN                                                         
----------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_pkey on bookings  (cost=0.43..8.45 rows=1 width=21) (actual time=0.039..0.039 rows=1.00 loops=1)
   Index Cond: (book_ref = 'UEX925'::bpchar)
   Filter: (book_date = '2025-09-03 03:27:39.815145+03'::timestamp with time zone)
   Index Searches: 1
   Buffers: shared hit=1 read=3
 Planning Time: 0.079 ms
 Execution Time: 0.056 ms
(7 rows)

demo=# explain (analyze, buffers) select * from bookings.bookings_copy where book_ref = 'UEX925' and book_date = '2025-09-03 03:27:39.815145+03';
                                                                             QUERY PLAN                                                                             
-----------------------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_copy_2025_09_pkey on bookings_copy_2025_09 bookings_copy  (cost=0.42..8.44 rows=1 width=21) (actual time=0.026..0.028 rows=1.00 loops=1)
   Index Cond: ((book_ref = 'UEX925'::bpchar) AND (book_date = '2025-09-03 03:27:39.815145+03'::timestamp with time zone))
   Index Searches: 1
   Buffers: shared hit=4
 Planning Time: 0.096 ms
 Execution Time: 0.039 ms
(6 rows)

demo=#

--Однако, в случае исключения из предикатов ключа партиционирования, будем наблюдать Append с индексным доступом по всем партициям! Что создает больше блокировок, использует больше ресурсов, вследствие чего время выполнения запроса увеличивается. Это еще раз говорит о том, что очень важно не забыть добавлять ключ партиционирования для ванильного PostgreSQL:
demo=# explain (analyze, buffers) select * from bookings.bookings where book_ref = 'UEX925';
                                                         QUERY PLAN                                                         
----------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_pkey on bookings  (cost=0.43..8.45 rows=1 width=21) (actual time=0.033..0.034 rows=1.00 loops=1)
   Index Cond: (book_ref = 'UEX925'::bpchar)
   Index Searches: 1
   Buffers: shared hit=4
 Planning Time: 0.075 ms
 Execution Time: 0.051 ms
(6 rows)

demo=# explain (analyze, buffers) select * from bookings.bookings_copy where book_ref = 'UEX925';
                                                                                 QUERY PLAN                                                                                 
-----------------------------------------------------------------------------------------------------------------------------------------------------
 Append  (cost=0.42..58.97 rows=7 width=21) (actual time=0.023..0.089 rows=1.00 loops=1)
   Buffers: shared hit=21
   ->  Index Scan using bookings_copy_2025_09_pkey on bookings_copy_2025_09 bookings_copy_1  (cost=0.42..8.44 rows=1 width=21) (actual time=0.022..0.023 rows=1.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=4
   ->  Index Scan using bookings_copy_2025_10_pkey on bookings_copy_2025_10 bookings_copy_2  (cost=0.42..8.44 rows=1 width=21) (actual time=0.013..0.013 rows=0.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=3
   ->  Index Scan using bookings_copy_2025_11_pkey on bookings_copy_2025_11 bookings_copy_3  (cost=0.42..8.44 rows=1 width=21) (actual time=0.011..0.011 rows=0.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=3
   ->  Index Scan using bookings_copy_2025_12_pkey on bookings_copy_2025_12 bookings_copy_4  (cost=0.42..8.44 rows=1 width=21) (actual time=0.011..0.011 rows=0.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=3
   ->  Index Scan using bookings_copy_2026_01_pkey on bookings_copy_2026_01 bookings_copy_5  (cost=0.42..8.44 rows=1 width=21) (actual time=0.011..0.011 rows=0.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=3
   ->  Index Scan using bookings_copy_2026_02_pkey on bookings_copy_2026_02 bookings_copy_6  (cost=0.42..8.44 rows=1 width=21) (actual time=0.010..0.010 rows=0.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=3
   ->  Index Scan using bookings_copy_2026_03_pkey on bookings_copy_2026_03 bookings_copy_7  (cost=0.28..8.29 rows=1 width=21) (actual time=0.007..0.007 rows=0.00 loops=1)
         Index Cond: (book_ref = 'UEX925'::bpchar)
         Index Searches: 1
         Buffers: shared hit=2
 Planning:
   Buffers: shared hit=69 dirtied=2
 Planning Time: 0.534 ms
 Execution Time: 0.124 ms
(34 rows)

demo=#
```

--Для выбранной таблицы дополнительные индексы не требуются, таблица имеет всего лишь 3 столбца.
