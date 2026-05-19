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

