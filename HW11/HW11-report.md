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
```sh
--рассмотрим работу составного индекса с функционалом Index Skip Scan.
--в Oracle этот тип индексного доступа широко известен, а в PostgreSQL он заявлен только с версии 18:
--https://betterstack.com/community/guides/databases/skip-scans-postgres/
--однако, даже в 15-ом ядре работает. Ниже пример:

otus_dba1=# create table tb_index_test (
otus_dba1(# v1 varchar(10), v2 varchar(36), t1 timestamp);
CREATE TABLE
otus_dba1=#

--используем uuid v7 для генерации случайных, но последовательных значений:
select uuidv7();

--вставляем 2 млн строк:
do $$
declare
   n_of_recs bigint := 1000000;
   random_varchar_gender varchar(10);
   random_varchar varchar(36);
   random_timestamp timestamp;
   query text;
   rec record;
begin

   for rows_rec in 1..n_of_recs loop

      --gender distribution
      select CASE WHEN RANDOM() < 0.5 THEN 'M' ELSE 'F' END into random_varchar_gender;

      -- some random varchar with uuid v7
      select uuidv7() into random_varchar; 

      -- some random timestamp between '2026-05-01 00:00:00' and '2026-05-18 00:00:00'
      random_timestamp := timestamp '2026-05-01 00:00:00' + random() * (timestamp '2026-05-18 00:00:00' - timestamp '2026-05-01 00:00:00');

      query := 'insert into tb_index_test values($1, $2, $3)';

      execute query using random_varchar_gender, random_varchar, random_timestamp;

      if rows_rec % 1000000 = 0 then
         raise notice 'Num of recs inserted into the table tb_index_test: %', rows_rec;
      end if;

   end loop;

end$$;

NOTICE:  Num of recs inserted into the table tb_index_test: 1000000
DO
otus_dba1=#

--собираем статистики, проверяем:
otus_dba1=# analyze tb_index_test;
ANALYZE
otus_dba1=# select relname, relpages, reltuples, relallvisible from pg_class where relname = 'tb_index_test';
    relname    | relpages | reltuples | relallvisible 
---------------+----------+-----------+---------------
 tb_index_test |    18692 |     2e+06 |          9345
(1 row)

otus_dba1=#

--селективность:
otus_dba1=# select attname, n_distinct from pg_stats where tablename ='tb_index_test';
 attname | n_distinct 
---------+------------
 v1      |          2
 v2      |         -1
 t1      |         -1
(3 rows)

otus_dba1=#

otus_dba1=# select count(*) from tb_index_test;
  count  
---------
 2000000
(1 row)

otus_dba1=# select * from tb_index_test limit 10;
 v1 |                  v2                  |             t1             
----+--------------------------------------+----------------------------
 M  | 019e3ccb-ecb3-714c-8846-16a87e32ab96 | 2026-05-09 04:05:55.958346
 F  | 019e3ccb-ecb6-7fbe-8b95-c2da30b1d71c | 2026-05-12 19:04:57.24255
 M  | 019e3ccb-ecb7-708d-99d4-dd0d368800ed | 2026-05-11 10:54:10.878341
 F  | 019e3ccb-ecb7-70c3-b672-533945445c5c | 2026-05-04 19:06:42.974892
 F  | 019e3ccb-ecb7-70ea-abf2-317d2f20b969 | 2026-05-07 07:34:47.871821
 F  | 019e3ccb-ecb7-710f-95f6-62fb0a7a238d | 2026-05-12 15:52:40.065894
 F  | 019e3ccb-ecb7-7133-98a8-72d8ac4b9000 | 2026-05-16 19:12:05.811224
 F  | 019e3ccb-ecb7-7157-b1bd-81366cf35129 | 2026-05-14 03:36:00.184626
 M  | 019e3ccb-ecb7-717b-b7c1-dcfb006e62d6 | 2026-05-01 14:02:21.474568
 M  | 019e3ccb-ecb7-719f-ae0d-0d259fb09a44 | 2026-05-06 02:52:32.866078
(10 rows)

otus_dba1=#

--на данный момент индексов нет:
otus_dba1=# select COUNT(*) from pg_indexes where tablename = 'tb_index_test';
 count 
-------
     0
(1 row)

otus_dba1=#

--создаем составной индекс, где лидирующим полем поставим низкоселективный столбец (важное условие применения оптимизатором доступа через IndexSkipScan):
otus_dba1=# create index idx_skip_scan_2cols on tb_index_test (v1, v2);
CREATE INDEX
otus_dba1=# select relname, relpages, reltuples, relallvisible from pg_class where relname = 'idx_skip_scan_2cols';
       relname       | relpages | reltuples | relallvisible 
---------------------+----------+-----------+---------------
 idx_skip_scan_2cols |    14420 |     2e+06 |             0
(1 row)

otus_dba1=#

--Смотрим теоретический план для выборки столбцов на индексе, в предикате укажем второй столбец с высокой селективностью, видим IndexOnlyScan:
otus_dba1=# explain select v1, v2 from tb_index_test where v2 = '019e3ccb-ecb7-719f-ae0d-0d259fb09a44';
                                           QUERY PLAN                                           
------------------------------------------------------------------------------------------------
 Index Only Scan using idx_skip_scan_2cols on tb_index_test  (cost=0.55..13.69 rows=1 width=39)
   Index Cond: (v2 = '019e3ccb-ecb7-719f-ae0d-0d259fb09a44'::text)
(2 rows)

--Если в выборку добавим третий столбец, то увидим IndexScan, но всё равно индексный доступ!
--Кроме как использование Index Skip Scan, по-другому не объясню, т.к. необходимо сначала пройтись по индексному дереву через первый столбец  v1, и потом искать совпадения по второму v2:
otus_dba1=# explain select * from tb_index_test where v2 = '019e3ccb-ecb7-719f-ae0d-0d259fb09a44';
                                        QUERY PLAN                                         
-------------------------------------------------------------------------------------------
 Index Scan using idx_skip_scan_2cols on tb_index_test  (cost=0.55..17.69 rows=1 width=47)
   Index Cond: ((v2)::text = '019e3ccb-ecb7-719f-ae0d-0d259fb09a44'::text)
(2 rows)

otus_dba1=#

--Найдем максимальное число страниц в индексе через ctid и выберем v2 оттуда, проверим число прочитанных страниц.
--ожидал, что число чтений будет большим, но нет..реально оценить, что происходит под капотом не могу, но по факту, Index Skip Scan работает в 15 ядре. При этом в 18-ом, в плане фигурирует тот же IndexScan, без Index Skip Scan, непонятно, в чем смысл внедрения фичи...
otus_dba1=# select max(ctid) from tb_index_test;
    max     
------------
 (18691,63)
(1 row)

                      
otus_dba1=# select * from tb_index_test where ctid = '(18691,62)';
 v1 |                  v2                  |             t1             
----+--------------------------------------+----------------------------
 M  | 019e3ccd-8c57-7648-87f4-601fd99a4a78 | 2026-05-06 02:35:29.720934
(1 row)

otus_dba1=#


otus_dba1=# explain (analyze, buffers)
otus_dba1-# select * from tb_index_test where v2 = '019e3ccd-8c57-7648-87f4-601fd99a4a78';

QUERY PLAN                                                               
----------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_skip_scan_2cols on tb_index_test  (cost=0.55..17.69 rows=1 width=47) (actual time=0.228..0.229 rows=1.00 loops=1)
   Index Cond: ((v2)::text = '019e3ccd-8c57-7648-87f4-601fd99a4a78'::text)
   Index Searches: 3
   Buffers: shared hit=6 read=9
 Planning Time: 0.137 ms
 Execution Time: 0.247 ms
(6 rows)



otus_dba1=# select * from tb_index_test where ctid = '(16691,62)';
 v1 |                  v2                  |             t1             
----+--------------------------------------+----------------------------
 F  | 019e3ccd-8333-7d0e-85f5-ded83952e62f | 2026-05-01 03:43:42.574894
(1 row)

otus_dba1=# explain (analyze, buffers)
select * from tb_index_test where v2 = '019e3ccd-8333-7d0e-85f5-ded83952e62f';

QUERY PLAN                                                               
----------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_skip_scan_2cols on tb_index_test  (cost=0.55..17.69 rows=1 width=47) (actual time=0.153..0.202 rows=1.00 loops=1)
   Index Cond: ((v2)::text = '019e3ccd-8333-7d0e-85f5-ded83952e62f'::text)
   Index Searches: 5
   Buffers: shared hit=17 read=4
 Planning Time: 0.084 ms
 Execution Time: 0.223 ms
(6 rows)



otus_dba1=# drop index idx_skip_scan_2cols;
DROP INDEX

--А вот если создаем тройной индекс, и в предикате указываем столбец на третьем месте, оптимизатор считает, что ему дешевле выполнить SeqScan:
otus_dba1=# create index idx_skip_scan_3cols on tb_index_test (v1, t1, v2);
CREATE INDEX
otus_dba1=#


otus_dba1=# explain (analyze, buffers)
select * from tb_index_test where v2 = '019e3ccd-8c57-7648-87f4-601fd99a4a78';

QUERY PLAN                                                           
--------------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..30108.77 rows=1 width=47) (actual time=29.173..32.098 rows=1.00 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   Buffers: shared hit=18692
   ->  Parallel Seq Scan on tb_index_test  (cost=0.00..29108.67 rows=1 width=47) (actual time=24.769..24.770 rows=0.33 loops=3)
         Filter: ((v2)::text = '019e3ccd-8c57-7648-87f4-601fd99a4a78'::text)
         Rows Removed by Filter: 666666
         Buffers: shared hit=18692
 Planning:
   Buffers: shared hit=22 read=1
 Planning Time: 0.284 ms
 Execution Time: 32.118 ms
(12 rows)



otus_dba1=# drop index idx_skip_scan_3cols;
DROP INDEX
otus_dba1=#
```


###
На занятии обсуждали сиутацию, когда при использовании Index Only Scan всё равно может быть обращение к табличным страницам.
Далее постараюсь это показать:
###
```sh

--тест на 15-ом ядре (15.5)

--создаем таблицу:
testdb=# CREATE TABLE tb_ios_test (
    id bigint,
    status TEXT NOT NULL DEFAULT 'NEW',
    t1 TIMESTAMP WITHOUT TIME ZONE NOT NULL
);
CREATE TABLE
testdb=#

--вставляем 1 млн строк:
testdb=# INSERT INTO tb_ios (id, t1)
SELECT
    i,
    NOW() + (i * INTERVAL '1 second')
FROM generate_series(1, 1000000) AS i;
INSERT 0 1000000
testdb=#

 
--создаем составной индекс на 2 поля:
testdb=# create index idx_ios on tb_ios_test (id, status);
CREATE INDEX
testdb=#

--смотрим базовые статистики:
testdb=# select relname, relpages, reltuples, relallvisible from pg_class where relname in ('tb_ios_test', 'idx_ios');
   relname   | relpages | reltuples | relallvisible
-------------+----------+-----------+---------------
tb_ios_test |     6411 |     1e+06 |          6411
idx_ios     |     3848 |     1e+06 |             0
(2 rows)
testdb=#

 

testdb=# select * from tb_ios_test limit 10;
id | status |             t1
----+--------+----------------------------
  1 | NEW    | 2026-05-19 11:56:26.808125
  2 | NEW    | 2026-05-19 11:56:27.808125
  3 | NEW    | 2026-05-19 11:56:28.808125
  4 | NEW    | 2026-05-19 11:56:29.808125
  5 | NEW    | 2026-05-19 11:56:30.808125
  6 | NEW    | 2026-05-19 11:56:31.808125
  7 | NEW    | 2026-05-19 11:56:32.808125
  8 | NEW    | 2026-05-19 11:56:33.808125
  9 | NEW    | 2026-05-19 11:56:34.808125
10 | NEW    | 2026-05-19 11:56:35.808125
(10 rows)
testdb=#

 
--делаем выборку по 2 полям, на которые создам индекс.
--видим, что Heap Fetches = 0, обращений к таблице нет:
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id = 1000;
                                                        QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.42..2.44 rows=1 width=12) (actual time=0.068..0.069 rows=1 loops=1)
   Index Cond: (id = 1000)
   Heap Fetches: 0
   Buffers: shared hit=4 read=3
   I/O Timings: shared/local read=0.019
Planning:
   Buffers: shared hit=5
Planning Time: 0.098 ms
Execution Time: 0.209 ms
(9 rows)
testdb=#

 
--Немного теории:
--при Index Only Scan важна видимость версии строки, есть проверка бита "видимости всех записей" нужной страницы с помощью Visibility Map.
--Бит видимости присутствует в табличных данных, не в индексных.
--Если бит не установлен, то как и в случае с Index Scan, происходит извлечение записи из файла таблицы, удостоверяясь в ее "видимости" для текущей --транзакции (по значениям xmin/xmax).
--Проверка одного бита VM существенно дешевле, чем извлечение целой записи из файла таблицы.

--Важно: биты карты видимости устанавливаются только при очистке, а сбрасываются при любых операциях, изменяющих данные на странице.

 --Т.е. Index Only Scan неэффективен, когда:
--происходит извлечение "свежевставленных" записей, на которые еще не успел прийти [auto]VACUUM

--Heap Fetches - показывает нам количество записей, которые пришлось "достать" из файла таблицы.

--отключим автовакуум на рассматриваемую таблицу:
testdb=# alter table tb_ios_test set (autovacuum_enabled=off);
ALTER TABLE

testdb=# select relname, reloptions from pg_class where relname = 'tb_ios_test';
   relname   |        reloptions
-------------+--------------------------
tb_ios_test | {autovacuum_enabled=off}
(1 row)

--выполним обновление поля status на всех строках:
testdb=# update tb_ios_test set status = 'UPDATE';
UPDATE 1000000

 
--автовакуум не сработал, информация по VM (relallvisible) не отобразилась:
testdb=# select relname, relpages, reltuples, relallvisible from pg_class where relname in ('tb_ios_test', 'idx_ios');
   relname   | relpages | reltuples | relallvisible
-------------+----------+-----------+---------------
tb_ios_test |     6411 |     1e+06 |          6411
idx_ios     |     3848 |     1e+06 |             0
(2 rows)
testdb=#

 

testdb=# select relname, last_autovacuum, last_autoanalyze from pg_stat_all_tables where relname in ('tb_ios_test', 'idx_ios');
   relname   |        last_autovacuum        |       last_autoanalyze
-------------+-------------------------------+-------------------------------
tb_ios_test | 2026-05-19 11:56:35.503592+03 | 2026-05-19 11:56:35.609684+03
(1 row)
testdb=#

 
--Выпролним тот же азпрос для конкретного id.
--Видим, что heap Fetches уже больше 0, т.е. обращение к табличнвым старницам уже есть, хотя в плане видим Index Only Scan:
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id = 1000;
                                                        QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.43..4.45 rows=1 width=12) (actual time=0.050..0.051 rows=1 loops=1)
   Index Cond: (id = 1000)
   Heap Fetches: 2
   Buffers: shared hit=6
Planning Time: 0.087 ms
Execution Time: 0.180 ms
(6 rows)
testdb=#

 


testdb=# select * from pg_indexes where tablename = 'tb_ios_test';
schemaname |  tablename  |    indexname     | tablespace |                                    indexdef
------------+-------------+------------------+------------+--------------------------------------------------------------------------------
asvschema  | tb_ios_test | idx_ios          |            | CREATE INDEX idx_ios ON asvschema.tb_ios_test USING btree (id, status)
(1 row)
testdb=#

--Укажем в предикате выборку не по одному конкретному id, а по 1000.
--Сразу фиксируем сильно увеличение Heap Fetches:
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 1000;
                                                            QUERY PLAN
-----------------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.43..800.11 rows=1814 width=12) (actual time=0.028..0.671 rows=1000 loops=1)
   Index Cond: (id <= 1000)
   Heap Fetches: 1998
   Buffers: shared hit=2012
Planning:
   Buffers: shared hit=5
Planning Time: 0.150 ms
Execution Time: 0.907 ms
(8 rows)
testdb=#

 

 
--Увеличивая выборку наблюдаем увеличение Heap Fetches и число сканируемых блоков данных (страниц) и обращаем внимание на рост времени выполнения:
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 2000;
                                                             QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.43..1504.50 rows=3629 width=12) (actual time=0.011..1.075 rows=2000 loops=1)
   Index Cond: (id <= 2000)
   Heap Fetches: 3000
   Buffers: shared hit=2033
Planning:
   Buffers: shared hit=5
Planning Time: 0.132 ms
Execution Time: 1.289 ms
(8 rows)
testdb=#

 

 
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 10000;
                                                              QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.43..5211.57 rows=18151 width=12) (actual time=0.010..6.602 rows=10000 loops=1)
   Index Cond: (id <= 10000)
   Heap Fetches: 18000
   Buffers: shared hit=16132
Planning:
   Buffers: shared hit=5
Planning Time: 0.141 ms
Execution Time: 7.222 ms
(8 rows)
testdb=#

 

 
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 100000;
                                                                QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.43..11659.81 rows=196015 width=12) (actual time=0.027..47.888 rows=100000 loops=1)
   Index Cond: (id <= 100000)
   Heap Fetches: 150000
   Buffers: shared hit=101474
Planning Time: 0.101 ms
Execution Time: 52.485 ms
(6 rows)
testdb=#

 

 
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 500000;
                                                                 QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.43..36578.35 rows=1003685 width=12) (actual time=0.030..118.971 rows=500000 loops=1)
   Index Cond: (id <= 500000)
   Heap Fetches: 500000
   Buffers: shared hit=8956
Planning Time: 0.096 ms
Execution Time: 140.139 ms
(6 rows)
testdb=#

 

 
--Достигаем такого числа сканирования блоков данных, что оптимизатору выгоднее перейти на SeqScan:
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 600000;
                                                      QUERY PLAN
-----------------------------------------------------------------------------------------------------------------------
Seq Scan on tb_ios_test  (cost=0.00..37819.05 rows=1200670 width=12) (actual time=7.457..110.948 rows=600000 loops=1)
   Filter: (id <= 600000)
   Rows Removed by Filter: 400000
   Buffers: shared hit=12821
Planning Time: 0.086 ms
Execution Time: 134.773 ms
(6 rows)


--Выполянеем ручной вакуум, VM пришла в актуальное состояние, Heap Fetches падает в 0, время выполнения сокращается:
testdb=# vacuum analyze tb_ios_test;
VACUUM
testdb=#

 
testdb=# explain (analyze, buffers) select id, status from tb_ios_test where id <= 500000;
                                                                QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------
Index Only Scan using idx_ios on tb_ios_test  (cost=0.42..20173.83 rows=496880 width=15) (actual time=0.022..61.001 rows=500000 loops=1)
   Index Cond: (id <= 500000)
   Heap Fetches: 0
   Buffers: shared hit=5750
Planning Time: 0.085 ms
Execution Time: 81.965 ms
(6 rows)
testdb=#



--Данным примером хотелось показать, что наличие в плане выполнения Index Only Scan не является критерием отсутствия обращения к табличнвм страницам.
--Если сделал неправильные выводы - просьба указать на ошибки. Спасибо.
