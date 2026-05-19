###
Создать индекс к какой-либо из таблиц вашей БД
###
```sh
otus_dba1=# create table tb_ios (id bigint, status varchar(10) not null default 'NEW', t1 timestamp);
CREATE TABLE
otus_dba1=#

otus_dba1=# insert into tb_ios (id, t1)
otus_dba1-# select i, now() + (i * interval '1 second')
otus_dba1-# from generate_series(1,1000000) as i;
INSERT 0 1000000
otus_dba1=#
```

###
Проверяем базовые статистики, создаем PK - с уникальным индексом:
###
```sh
otus_dba1=# analyze tb_ios;
ANALYZE
                          ^
otus_dba1=# alter table tb_ios add constraint tb_ios_pk primary key (id);
ALTER TABLE
otus_dba1=# select relname, relpages, reltuples, relallvisible from pg_class where relname in ('tb_ios', 'tb_ios_pk');
  relname  | relpages | reltuples | relallvisible 
-----------+----------+-----------+---------------
 tb_ios    |     6370 |     1e+06 |          6370
 tb_ios_pk |     2745 |     1e+06 |             0
(2 rows)

otus_dba1=#
```

###
Прислать текстом результат команды explain, в которой используется данный индекс
###
```sh
otus_dba1=# select * from tb_ios limit 10;
 id | status |             t1             
----+--------+----------------------------
  1 | NEW    | 2026-05-19 20:39:19.182081
  2 | NEW    | 2026-05-19 20:39:20.182081
  3 | NEW    | 2026-05-19 20:39:21.182081
  4 | NEW    | 2026-05-19 20:39:22.182081
  5 | NEW    | 2026-05-19 20:39:23.182081
  6 | NEW    | 2026-05-19 20:39:24.182081
  7 | NEW    | 2026-05-19 20:39:25.182081
  8 | NEW    | 2026-05-19 20:39:26.182081
  9 | NEW    | 2026-05-19 20:39:27.182081
 10 | NEW    | 2026-05-19 20:39:28.182081
(10 rows)

--теоретический (расчетный) план - estimated в терминах Oracle:
otus_dba1=# explain select * from tb_ios where id = 99999;
                               QUERY PLAN                                
-------------------------------------------------------------------------
 Index Scan using tb_ios_pk on tb_ios  (cost=0.42..8.44 rows=1 width=20)
   Index Cond: (id = 99999)
(2 rows)

--реальный план (с физическим выполнением) - actual в терминах Oracle:
otus_dba1=# explain (analyze, buffers) select * from tb_ios where id = 99999;

QUERY PLAN                                                      
----------------------------------------------------------------------------------------------------------------------
 Index Scan using tb_ios_pk on tb_ios  (cost=0.42..8.44 rows=1 width=20) (actual time=0.118..0.120 rows=1.00 loops=1)
   Index Cond: (id = 99999)
   Index Searches: 1
   Buffers: shared hit=4 read=3
 Planning Time: 0.083 ms
 Execution Time: 0.137 ms
(6 rows)

--в данном случае стоимость и косты совпали
```

###
Реализовать индекс для полнотекстового поиска
###
```sh
--Никогда их не использовал, сначала попробую на простом примере.

--преобразование строки в tsvector с помощью функции to_tsvector
otus_dba1=# select to_tsvector('My name is Sergei and I''m a DBA.');
            to_tsvector            
-----------------------------------
 'dba':9 'm':7 'name':2 'sergei':4
(1 row)

--проверка сопоставления tsvector с tsquery через оператор @@ (true):
otus_dba1=# select to_tsvector('Hello, world!') @@ plainto_tsquery('hello world') as match;
 match 
-------
 t
(1 row)

--проверка сопоставления tsvector с tsquery через оператор @@ (false):
otus_dba1=# select to_tsvector('Hello, world!') @@ plainto_tsquery('ello world') as match;
 match 
-------
 f
(1 row)

otus_dba1=#


--Для показа принципа работы использую пример из книги Егора Рогова с небольшими правками.

--Создадим таблицу с двумя столбцами: в первом будет храниться документ, во втором — значение tsvector.
--Второй столбец объявляется как генерируемый:

otus_dba1=# CREATE TABLE ts(
doc text,
doc_tsv tsvector GENERATED ALWAYS AS (
to_tsvector('pg_catalog.russian', doc)
) STORED
);
CREATE TABLE
otus_dba1=#

--вставляем набор строк:
otus_dba1=# INSERT INTO ts(doc) VALUES
('Во поле береза стояла'), ('Во поле кудрявая стояла'),
('Люли, люли, стояла'), ('Люли, люли, стояла'),
('Некому березу заломати'), ('Некому кудряву заломати'),
('Люли, люли, заломати'), ('Люли, люли, заломати'),
('Я пойду погуляю'), ('Белую березу заломаю'),
('Люли, люли, заломаю'), ('Люли, люли, заломаю')
RETURNING doc_tsv;
            doc_tsv             
--------------------------------
 'берез':3 'пол':2 'стоя':4
 'кудряв':3 'пол':2 'стоя':4
 'люл':1,2 'стоя':3
 'люл':1,2 'стоя':3
 'берез':2 'заломат':3 'нек':1
 'заломат':3 'кудряв':2 'нек':1
 'заломат':3 'люл':1,2
 'заломат':3 'люл':1,2
 'погуля':3 'пойд':2
 'бел':1 'берез':2 'залома':3
 'залома':3 'люл':1,2
 'залома':3 'люл':1,2
(12 rows)

INSERT 0 12
otus_dba1=#


otus_dba1=# CREATE INDEX ts_gin_idx ON ts USING gin (doc_tsv);
CREATE INDEX
otus_dba1=#

otus_dba1=# SELECT * FROM ts WHERE doc_tsv @@ to_tsquery('russian', 'берез:*');
          doc           |            doc_tsv            
------------------------+-------------------------------
 Во поле береза стояла  | 'берез':3 'пол':2 'стоя':4
 Некому березу заломати | 'берез':2 'заломат':3 'нек':1
 Белую березу заломаю   | 'бел':1 'берез':2 'залома':3
(3 rows)

otus_dba1=#

--изначально оптимизатор использует SeqScan, т.к. строк мало и это дешевле, но если выключить возможность использования SeqScan, будет использоваться созданный индекс:
otus_dba1=# explain (analyze, buffers) SELECT * FROM ts WHERE doc_tsv @@ to_tsquery('russian', 'берез:*');
                                           QUERY PLAN                                           
------------------------------------------------------------------------------------------------
 Seq Scan on ts  (cost=0.00..1.15 rows=1 width=64) (actual time=0.016..0.018 rows=3.00 loops=1)
   Filter: (doc_tsv @@ '''берез'':*'::tsquery)
   Rows Removed by Filter: 9
   Buffers: shared hit=1
 Planning:
   Buffers: shared hit=1
 Planning Time: 0.087 ms
 Execution Time: 0.027 ms
(8 rows)

otus_dba1=# set enable_seqscan = off;
SET
otus_dba1=#

otus_dba1=# explain (analyze, buffers) SELECT * FROM ts WHERE doc_tsv @@ to_tsquery('russian', 'берез:*');
QUERY PLAN                                                       
-----------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on ts  (cost=12.77..16.79 rows=1 width=64) (actual time=0.018..0.018 rows=3.00 loops=1)
   Recheck Cond: (doc_tsv @@ '''берез'':*'::tsquery)
   Heap Blocks: exact=1
   Buffers: shared hit=3
   ->  Bitmap Index Scan on ts_gin_idx  (cost=0.00..12.77 rows=1 width=0) (actual time=0.010..0.010 rows=3.00 loops=1)
         Index Cond: (doc_tsv @@ '''берез'':*'::tsquery)
         Index Searches: 1
         Buffers: shared hit=2
 Planning:
   Buffers: shared hit=1
 Planning Time: 0.098 ms
 Execution Time: 0.034 ms
(12 rows)
```

###
Реализовать индекс на часть таблицы или индекс на поле с функцией
###
```sh
--создадим partial (частичный) индекс

--Предварительно вставим строку со значением status = 'COMPLETE':
otus_dba1=# insert into tb_ios (id, status, t1) values (1000001, 'COMPLETE', now());
INSERT 0 1
otus_dba1=# analyze tb_ios;
ANALYZE
otus_dba1=#

otus_dba1=# insert into tb_ios (id, status, t1) values (1000001, 'COMPLETE', now());
INSERT 0 1
otus_dba1=# analyze tb_ios;
ANALYZE
otus_dba1=# create index idx_tb_ios_status_partial on tb_ios (status) where status = 'COMPLETE';
CREATE INDEX
otus_dba1=#

--проверяем, что число строк в индексе только одна:
otus_dba1=# select relname, relpages, reltuples from pg_class where relname = 'idx_tb_ios_status_partial';
          relname          | relpages | reltuples 
---------------------------+----------+-----------
 idx_tb_ios_status_partial |        2 |         1
(1 row)

otus_dba1=#

--проверяем, что оптимизатор использует Index Scan, т.к. у значения COMPLETE очень хорошая селективность:
otus_dba1=# explain (analyze, buffers) select * from tb_ios where status = 'COMPLETE';
QUERY PLAN                                                              
--------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_tb_ios_status_partial on tb_ios  (cost=0.12..4.14 rows=1 width=20) (actual time=0.020..0.021 rows=1.00 loops=1)
   Index Searches: 1
   Buffers: shared hit=1 read=1
 Planning:
   Buffers: shared hit=43 read=1 dirtied=1
 Planning Time: 0.386 ms
 Execution Time: 0.033 ms
(7 rows)
```

###
Создать индекс на несколько полей
###
