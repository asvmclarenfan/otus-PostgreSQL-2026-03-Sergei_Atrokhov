###
Выполняем подключение к тестовой БД:
###
```sh
ostgres@asvpg:~$ psql -d otus_dba1 -p 5433
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
 Backend PID          | 5323
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

otus_dba1=#
```

###
Настройте PostgreSQL так, чтобы в журнал сообщений записывались ожидания блокировок дольше 200 мс;
###
```sh
--проверяем\настраиваем логирование блокировок в журнал СУБД:
otus_dba1=# show log_lock_waits;
 log_lock_waits 
----------------
 off
(1 row)

otus_dba1=# select name, setting, context from pg_settings where name = 'log_lock_waits';
      name      | setting |  context  
----------------+---------+-----------
 log_lock_waits | off     | superuser
(1 row)

otus_dba1=# alter system set log_lock_waits = 'on';
ALTER SYSTEM
otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# select name, setting, context from pg_settings where name = 'log_lock_waits';
      name      | setting |  context  
----------------+---------+-----------
 log_lock_waits | on      | superuser
(1 row)

otus_dba1=#

--проверяем\настраиваем нужную длительность логируемых блокировок:
otus_dba1=# show deadlock_timeout;
 deadlock_timeout 
------------------
 1s
(1 row)

otus_dba1=# select name, setting, context from pg_settings where name = 'deadlock_timeout';
       name       | setting |  context  
------------------+---------+-----------
 deadlock_timeout | 1000    | superuser
(1 row)

otus_dba1=# alter system set deadlock_timeout  = '200ms';
ALTER SYSTEM
otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# select name, setting, context from pg_settings where name = 'deadlock_timeout';
       name       | setting |  context  
------------------+---------+-----------
 deadlock_timeout | 200     | superuser
(1 row)

otus_dba1=#
```

###
Воспроизведите ожидание блокировки дольше 200 мс и подтвердите это записью в журнале сообщений;
###
```sh
otus_dba1=# create table testnm.tb_lock (id serial, created_at timestamptz, amount numeric);
CREATE TABLE

otus_dba1=# \dt+ testnm.tb_lock
                                        List of tables
 Schema |  Name   | Type  |  Owner   | Persistence | Access method |    Size    | Description 
--------+---------+-------+----------+-------------+---------------+------------+-------------
 testnm | tb_lock | table | postgres | permanent   | heap          | 8192 bytes | 
(1 row)

otus_dba1=# insert into testnm.tb_lock values (1, now(), 100);
INSERT 0 1
otus_dba1=# insert into testnm.tb_lock values (2, now(), 200);
INSERT 0 1
otus_dba1=# insert into testnm.tb_lock values (3, now(), 300);
INSERT 0 1
otus_dba1=#

--первая сессия:
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# lock table testnm.tb_lock;
LOCK TABLE
otus_dba1=*#
otus_dba1=*# select pg_backend_pid();
 pg_backend_pid 
----------------
           5323
(1 row)

otus_dba1=*#

--вторая сессия:
otus_dba1=# select pg_backend_pid();
 pg_backend_pid 
----------------
          23136
(1 row)

otus_dba1=#

otus_dba1=# select * from testnm.tb_lock;

--смотрим лог СУБД:
2026-05-02 20:57:20.961 MSK [5323] postgres@otus_dba1 STATEMENT:  lock table testnm.tb_lock;
2026-05-02 20:58:39.710 MSK [23136] postgres@otus_dba1 LOG:  process 23136 still waiting for AccessShareLock on relation 57376 of database 16388 after 236.011 ms at character 15
2026-05-02 20:58:39.710 MSK [23136] postgres@otus_dba1 DETAIL:  Process holding the lock: 5323. Wait queue: 23136.
2026-05-02 20:58:39.710 MSK [23136] postgres@otus_dba1 STATEMENT:  select * from testnm.tb_lock;
postgres@asvpg:/var/log/postgresql$
```
###
В логе присутствует информация, что сессия с pid=5323 (первая сессия) удерживает блокировку для сессии с pid=23136 (вторая сессия).
Как только в первой сессии выполняем коммит, вторая сессия сразу выдает результат, блокировка освобождается:
###
```sh
--первая сессия:
otus_dba1=# select pg_backend_pid();
 pg_backend_pid 
----------------
           5323
(1 row)
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# lock table testnm.tb_lock;
LOCK TABLE

otus_dba1=*# commit;
COMMIT
otus_dba1=#

--вторая сессия:
otus_dba1=# select * from testnm.tb_lock;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-05-02 20:56:14.535134+03 |    100
  2 | 2026-05-02 20:56:20.491635+03 |    200
  3 | 2026-05-02 20:56:26.197195+03 |    300
(3 rows)

otus_dba1=#
```

###
Создайте тестовую таблицу и добавьте одну строку, которую будут обновлять все сессии - выполнено в предыдущем пункте.
###

###
Откройте три сессии psql к одной базе данных;
Выполните в трёх сессиях команды UPDATE, обновляющие одну и ту же строку, так чтобы одна транзакция удерживала блокировку, а остальные ожидали (выполняем обновление первой строки c id = 1):
###
```sh
otus_dba1=# select * from testnm.tb_lock;
 id |          created_at           | amount 
----+-------------------------------+--------
  1 | 2026-05-02 20:56:14.535134+03 |    100 -->
  2 | 2026-05-02 20:56:20.491635+03 |    200
  3 | 2026-05-02 20:56:26.197195+03 |    300
(3 rows)

otus_dba1=#

--первая сессия:
otus_dba1=# select pg_backend_pid();
 pg_backend_pid 
----------------
           5323
(1 row)

otus_dba1=#
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
UPDATE 1
otus_dba1=*#

--вторая сессия:
otus_dba1=# select pg_backend_pid();
 pg_backend_pid 
----------------
          23136
(1 row)

otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
--зависание

--третья сессия:
otus_dba1=# select pg_backend_pid();
 pg_backend_pid 
----------------
          37998
(1 row)

otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
--зависание

--сообщения в логе СУБД:
2026-05-02 21:15:17.128 MSK [23136] postgres@otus_dba1 LOG:  process 23136 still waiting for ShareLock on transaction 12199112 after 204.400 ms
2026-05-02 21:15:17.128 MSK [23136] postgres@otus_dba1 DETAIL:  Process holding the lock: 5323. Wait queue: 23136.
2026-05-02 21:15:17.128 MSK [23136] postgres@otus_dba1 CONTEXT:  while updating tuple (0,1) in relation "tb_lock"
2026-05-02 21:15:17.128 MSK [23136] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 150 where id = 1;
2026-05-02 21:16:02.549 MSK [37998] postgres@otus_dba1 LOG:  process 37998 still waiting for ExclusiveLock on tuple (0,1) of relation 57376 of database 16388 after 203.063 ms
2026-05-02 21:16:02.549 MSK [37998] postgres@otus_dba1 DETAIL:  Process holding the lock: 23136. Wait queue: 37998.
2026-05-02 21:16:02.549 MSK [37998] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 150 where id = 1;
postgres@asvpg:/var/log/postgresql$
```

###
Во время ожидания снимите список блокировок из представления pg_locks и сохраните его для отчёта;
###
```sh
otus_dba1=# SELECT locktype, relation::regclass, virtualxid AS virtxid, transactionid AS xid, mode, granted             
FROM pg_locks;
   locktype    |    relation    | virtxid |   xid    |       mode       | granted 
---------------+----------------+---------+----------+------------------+---------
 relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 virtualxid    |                | 14/33   |          | ExclusiveLock    | t
 relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 virtualxid    |                | 16/5    |          | ExclusiveLock    | t
 relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 virtualxid    |                | 18/6    |          | ExclusiveLock    | t
 relation      | pg_locks       |         |          | AccessShareLock  | t
 virtualxid    |                | 20/4    |          | ExclusiveLock    | t
 transactionid |                |         | 12199113 | ExclusiveLock    | t
 tuple         | testnm.tb_lock |         |          | ExclusiveLock    | f --!
 tuple         | testnm.tb_lock |         |          | ExclusiveLock    | t
 transactionid |                |         | 12199112 | ShareLock        | f --!
 transactionid |                |         | 12199112 | ExclusiveLock    | t
 transactionid |                |         | 12199114 | ExclusiveLock    | t
(14 rows)

otus_dba1=#
```
###
На самом деле информация не до конца информативна, явно надо было предварительно сохранить xid каждой транзакции, а не только pid.
Смотрми информацию из вью pg_stta_activity на уровне pid, должно быть понятнее и увидим очередь блокировок:
###
```sh
otus_dba1=# select pid, pg_blocking_pids(pid), wait_event_type, wait_event, state, SUBSTR(query,1,40) as query from pg_stat_activity where pid in (5323, 23136, 37998) \gx
-[ RECORD 1 ]----+-----------------------------------------
pid              | 5323
pg_blocking_pids | {}
wait_event_type  | Client
wait_event       | ClientRead
state            | idle in transaction
query            | update testnm.tb_lock set amount = 150 w
-[ RECORD 2 ]----+-----------------------------------------
pid              | 23136
pg_blocking_pids | {5323}
wait_event_type  | Lock
wait_event       | transactionid
state            | active
query            | update testnm.tb_lock set amount = 150 w
-[ RECORD 3 ]----+-----------------------------------------
pid              | 37998
pg_blocking_pids | {23136}
wait_event_type  | Lock
wait_event       | tuple
state            | active
query            | update testnm.tb_lock set amount = 150 w

otus_dba1=#
```
###
Первая сессия (pid=5323): апдейт выполнен, но транзакция не подтверждена\откачена (state=idle in transaction).
Вторая сессия (pid=23136): ожидание завершения транзакции в первой сессии.
Третья сессия (pid=37998): ожидание завершения транзакции во второй сессии.
###

###
При подтверждении транзакции в первой сессии, блокировка второй отпадет, а третья будет ждать подтверждения\отката транзакции во второй.
###
```sh
--первая сессия:
otus_dba1=*# commit;
COMMIT
otus_dba1=#

--вторая сессия:
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
UPDATE 1
otus_dba1=*#

--третья сессия продолжает висеть.

--смотрим информацию из pg_stat_activity:
otus_dba1=# select pid, pg_blocking_pids(pid), wait_event_type, wait_event, state, SUBSTR(query,1,40) as query from pg_stat_activity where pid in (5323, 23136, 37998) \gx
-[ RECORD 1 ]----+-----------------------------------------
pid              | 5323
pg_blocking_pids | {}
wait_event_type  | Client
wait_event       | ClientRead
state            | idle
query            | commit;
-[ RECORD 2 ]----+-----------------------------------------
pid              | 23136
pg_blocking_pids | {}
wait_event_type  | Client
wait_event       | ClientRead
state            | idle in transaction
query            | update testnm.tb_lock set amount = 150 w
-[ RECORD 3 ]----+-----------------------------------------
pid              | 37998
pg_blocking_pids | {23136}
wait_event_type  | Lock
wait_event       | transactionid
state            | active
query            | update testnm.tb_lock set amount = 150 w

otus_dba1=#

--смотрим информацию из pg_locks:
otus_dba1=# SELECT locktype, relation::regclass, virtualxid AS virtxid, transactionid AS xid, mode, granted                                                               
FROM pg_locks;
   locktype    |    relation    | virtxid |   xid    |       mode       | granted 
---------------+----------------+---------+----------+------------------+---------
 relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 virtualxid    |                | 16/5    |          | ExclusiveLock    | t
 relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 virtualxid    |                | 18/6    |          | ExclusiveLock    | t
 relation      | pg_locks       |         |          | AccessShareLock  | t
 virtualxid    |                | 20/7    |          | ExclusiveLock    | t
 transactionid |                |         | 12199113 | ExclusiveLock    | t
 transactionid |                |         | 12199113 | ShareLock        | f
 transactionid |                |         | 12199114 | ExclusiveLock    | t
(9 rows)

otus_dba1=#
```

###
Перепроведем тест, предварительно вернув данные как были и добавим информацию по xid и pid из pg_locks:
###
```sh
--начальное состояние:
otus_dba1=# select * from testnm.tb_lock;
 id |          created_at           | amount 
----+-------------------------------+--------
  2 | 2026-05-02 20:56:20.491635+03 |    200
  3 | 2026-05-02 20:56:26.197195+03 |    300
  1 | 2026-05-02 20:56:14.535134+03 |    100
(3 rows)

otus_dba1=#

--первая сессия:
otus_dba1=# start transaction;
START TRANSACTION

otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
           5323 |     12199116
(1 row)

otus_dba1=*#
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
UPDATE 1
otus_dba1=*#

--вторая сессия:
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
          23136 |     12199117
(1 row)

otus_dba1=*#
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
--зависание

--третья сессия:
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
          37998 |     12199118
(1 row)

otus_dba1=*#
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
--зависание

--смотрим содержимое pg_locks:
otus_dba1=# SELECT pid, locktype, relation::regclass, virtualxid AS virtxid, transactionid AS xid, mode, granted
FROM pg_locks;
  pid  |   locktype    |    relation    | virtxid |   xid    |       mode       | granted 
-------+---------------+----------------+---------+----------+------------------+---------
  5323 | relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
  5323 | virtualxid    |                | 14/36   |          | ExclusiveLock    | t
 23136 | relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 23136 | virtualxid    |                | 16/6    |          | ExclusiveLock    | t
 37998 | relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 37998 | virtualxid    |                | 18/7    |          | ExclusiveLock    | t
 23136 | transactionid |                |         | 12199116 | ShareLock        | f
 37998 | transactionid |                |         | 12199118 | ExclusiveLock    | t
 37998 | tuple         | testnm.tb_lock |         |          | ExclusiveLock    | f
 23136 | transactionid |                |         | 12199117 | ExclusiveLock    | t
  5323 | transactionid |                |         | 12199116 | ExclusiveLock    | t
 23136 | tuple         | testnm.tb_lock |         |          | ExclusiveLock    | t
(14 rows)

otus_dba1=#

--смотрим содержимое вью pg_stat_activity:
otus_dba1=# select pid, pg_blocking_pids(pid), wait_event_type, wait_event, state, SUBSTR(query,1,40) as query from pg_stat_activity where pid in (5323, 23136, 37998) \gx
-[ RECORD 1 ]----+-----------------------------------------
pid              | 5323
pg_blocking_pids | {}
wait_event_type  | Client
wait_event       | ClientRead
state            | idle in transaction
query            | update testnm.tb_lock set amount = 150 w
-[ RECORD 2 ]----+-----------------------------------------
pid              | 23136
pg_blocking_pids | {5323}
wait_event_type  | Lock
wait_event       | transactionid
state            | active
query            | update testnm.tb_lock set amount = 150 w
-[ RECORD 3 ]----+-----------------------------------------
pid              | 37998
pg_blocking_pids | {23136}
wait_event_type  | Lock
wait_event       | tuple
state            | active
query            | update testnm.tb_lock set amount = 150 w

otus_dba1=#
```

###
Объясните смысл каждой блокировки из pg_locks (что блокируется, какой режим, кто держит, кто ожидает);
###
```sh
Видим по одной строке на каждый pid с типом relation (на таблицу testnm.tb_lock) с уровнем блокировки на уровне строки (RowExclusiveLock) с успешным флагом получения блокировки (granted = t) - выполняется DML и пока он не подтвержден\откачен, DDL над таблицей выполнить нельзя.
Здесь проблем нет, так и должно быть. Блокировка на самом низком уровне.
virtualxid по одной строке на каждый pid не рассматриваем - это виртуальынй номер транзакции в эксклюзивном режиме (ExclusiveLock) с успешным флагом получения блокировки (granted = t).
Каждая сессия (pid) успешно получила блокировку на реальный номер транзакции в сессии (transactionid).
Первая сессия успешно выполнила запрашиваемый апдейт и СУБД ожидает команды от клиента (wait_event = ClientRead, а state = idle in transaction).
Апдейт второй сессии висит на блокировке. СУБД понимает, что строка заблокирована в несовместимом режиме, происходит захват блокировки ExclusiveLock на версию изменяемой строки (locktype = tuple), бит granted = t. Одновременно с этим запрашивается блокировка xid (locktype = transactionid) в разделяемом режиме SharedLock, бит granted = f(!!!), т.к. транзакция из первой сессии еще активна!
Третья сессия не может даже захватить блокировку версии строки (locktype = tuple), в эксклюзивном режиме (mode = ExclusiveLock), бит grahted = f (!!!).
При завершении транзакции в первой сессии дерево блокировок сместится. Во второй сессии их не будет, а в третьей сессии будет как сейчас во второй (блокировка на версию строки (tuple) будет получена, а на номер транзакции - нет):

--первая сессия:
otus_dba1=*# rollback;
ROLLBACK
otus_dba1=#

--вторая сессия:
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
UPDATE 1
otus_dba1=*#

--третья сессия будет продолжать висеть

--
otus_dba1=# select pid, pg_blocking_pids(pid), wait_event_type, wait_event, state, SUBSTR(query,1,40) as query from pg_stat_activity where pid in (5323, 23136, 37998) \gx
-[ RECORD 1 ]----+-----------------------------------------
pid              | 5323
pg_blocking_pids | {}
wait_event_type  | Client
wait_event       | ClientRead
state            | idle
query            | rollback;
-[ RECORD 2 ]----+-----------------------------------------
pid              | 23136
pg_blocking_pids | {}
wait_event_type  | Client
wait_event       | ClientRead
state            | idle in transaction
query            | update testnm.tb_lock set amount = 150 w
-[ RECORD 3 ]----+-----------------------------------------
pid              | 37998
pg_blocking_pids | {23136}
wait_event_type  | Lock
wait_event       | transactionid
state            | active
query            | update testnm.tb_lock set amount = 150 w

otus_dba1=#

--
otus_dba1=# SELECT pid, locktype, relation::regclass, virtualxid AS virtxid, transactionid AS xid, mode, granted                                                          
FROM pg_locks;
  pid  |   locktype    |    relation    | virtxid |   xid    |       mode       | granted 
-------+---------------+----------------+---------+----------+------------------+---------
 23136 | relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 23136 | virtualxid    |                | 16/6    |          | ExclusiveLock    | t
 37998 | relation      | testnm.tb_lock |         |          | RowExclusiveLock | t
 37998 | virtualxid    |                | 18/7    |          | ExclusiveLock    | t
 37998 | transactionid |                |         | 12199118 | ExclusiveLock    | t
 37998 | tuple         | testnm.tb_lock |         |          | ExclusiveLock    | t
 23136 | transactionid |                |         | 12199117 | ExclusiveLock    | t
 37998 | transactionid |                |         | 12199117 | ShareLock        | f
(10 rows)

otus_dba1=#

--в оставшихся 2 сессиях откатим активные транзакции.
```

###
Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
###
```sh
--тест проводим на ранее созданной таблице:
otus_dba1=# select * from testnm.tb_lock;
 id |          created_at           | amount 
----+-------------------------------+--------
  2 | 2026-05-02 20:56:20.491635+03 |    200
  3 | 2026-05-02 20:56:26.197195+03 |    300
  1 | 2026-05-02 20:56:14.535134+03 |    100
(3 rows)

otus_dba1=#

--предварительно откроем 3 сессии, начнем в каждой из них транзакцию, сохраним информацию по pid, xid:

--сессия 1
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
           4985 |     12199119
(1 row)

otus_dba1=*#

--сессия 2
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
           5632 |     12199120
(1 row)

otus_dba1=*#

--сессия 3
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
           6413 |     12199121
(1 row)

otus_dba1=*#

--сессия 1 для id=1 увеличивает amount с 100 до 150:
otus_dba1=*# update testnm.tb_lock set amount = 150 where id = 1;
UPDATE 1
otus_dba1=*#

--сессия 2 для id=2 увеличивает amount с 200 до 250, а затем пытается для id=1 увеличить amount до 200
otus_dba1=*# update testnm.tb_lock set amount = 250 where id = 2;
UPDATE 1
otus_dba1=*# update testnm.tb_lock set amount = 200 where id = 1;
--зависание

--сессия 3 для id=1 увеличивает amount до 250, а затем пытается для id=2 увеличить amount до 300
otus_dba1=*# update testnm.tb_lock set amount = 250 where id = 1;
--зависание, не дошли до второго апдейта

--сессия 1 для id=2 увеличивает amount до 350
--получаем информацию об обнаружении deadlock, после чего СУБД автоматически его разрешает по истечении установленного deadlock_timeout (200мс), инициируя проверку и принудительно прервав сессию (pid=4985), которая вызвала эту проверку:
otus_dba1=*# update testnm.tb_lock set amount = 350 where id = 2;
ERROR:  deadlock detected
DETAIL:  Process 4985 waits for ShareLock on transaction 12199120; blocked by process 5632.
Process 5632 waits for ShareLock on transaction 12199119; blocked by process 4985.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,2) in relation "tb_lock"
otus_dba1=!#

--смотрим лог СУБД:
2026-05-03 11:39:53.835 MSK [5632] postgres@otus_dba1 LOG:  process 5632 still waiting for ShareLock on transaction 12199119 after 201.565 ms
2026-05-03 11:39:53.835 MSK [5632] postgres@otus_dba1 DETAIL:  Process holding the lock: 4985. Wait queue: 5632.
2026-05-03 11:39:53.835 MSK [5632] postgres@otus_dba1 CONTEXT:  while updating tuple (0,7) in relation "tb_lock"
2026-05-03 11:39:53.835 MSK [5632] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 200 where id = 1;
2026-05-03 11:40:03.597 MSK [6413] postgres@otus_dba1 LOG:  process 6413 still waiting for ExclusiveLock on tuple (0,7) of relation 57376 of database 16388 after 200.922 ms
2026-05-03 11:40:03.597 MSK [6413] postgres@otus_dba1 DETAIL:  Process holding the lock: 5632. Wait queue: 6413.
2026-05-03 11:40:03.597 MSK [6413] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 250 where id = 1;
2026-05-03 11:40:18.188 MSK [1439] LOG:  checkpoint starting: time
2026-05-03 11:40:18.310 MSK [1439] LOG:  checkpoint complete: wrote 1 buffers (0.0%), wrote 0 SLRU buffers; 0 WAL file(s) added, 0 removed, 0 recycled; write=0.105 s, sync=0.004 s, total=0.123 s; sync files=1, longest=0.004 s, average=0.004 s; distance=1 kB, estimate=1 kB; lsn=E/B8994E8, redo lsn=E/B899488
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 LOG:  process 4985 detected deadlock while waiting for ShareLock on transaction 12199120 after 201.078 ms
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 DETAIL:  Process holding the lock: 5632. Wait queue: .
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 CONTEXT:  while updating tuple (0,2) in relation "tb_lock"
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 350 where id = 2;
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 ERROR:  deadlock detected
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 DETAIL:  Process 4985 waits for ShareLock on transaction 12199120; blocked by process 5632.
	Process 5632 waits for ShareLock on transaction 12199119; blocked by process 4985.
	Process 4985: update testnm.tb_lock set amount = 350 where id = 2;
	Process 5632: update testnm.tb_lock set amount = 200 where id = 1;
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 HINT:  See server log for query details.
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 CONTEXT:  while updating tuple (0,2) in relation "tb_lock"
2026-05-03 11:41:02.603 MSK [4985] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 350 where id = 2;
2026-05-03 11:41:02.603 MSK [5632] postgres@otus_dba1 LOG:  process 5632 acquired ShareLock on transaction 12199119 after 68969.812 ms
2026-05-03 11:41:02.603 MSK [5632] postgres@otus_dba1 CONTEXT:  while updating tuple (0,7) in relation "tb_lock"
2026-05-03 11:41:02.603 MSK [5632] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 200 where id = 1;
2026-05-03 11:41:02.603 MSK [6413] postgres@otus_dba1 LOG:  process 6413 acquired ExclusiveLock on tuple (0,7) of relation 57376 of database 16388 after 59207.073 ms
2026-05-03 11:41:02.603 MSK [6413] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 250 where id = 1;
2026-05-03 11:41:02.804 MSK [6413] postgres@otus_dba1 LOG:  process 6413 still waiting for ShareLock on transaction 12199120 after 200.416 ms
2026-05-03 11:41:02.804 MSK [6413] postgres@otus_dba1 DETAIL:  Process holding the lock: 5632. Wait queue: 6413.
2026-05-03 11:41:02.804 MSK [6413] postgres@otus_dba1 CONTEXT:  while updating tuple (0,7) in relation "tb_lock"
2026-05-03 11:41:02.804 MSK [6413] postgres@otus_dba1 STATEMENT:  update testnm.tb_lock set amount = 250 where id = 1;
postgres@asvpg:/var/log/postgresql$
```
###
Информацию по взаимоблокировке можно найти в логе СУБД: явно указывается номер pid и xid.
Для возникновения взаимоблокировки достаточно 2 сессий, в 3 уже начинается нагромождение дополнительных блокировок.
В любом случае, со времен Oracle наличие взаимоблокировке - скорее вопрос не к работе СУБД, а к логике написания приложения - критерий некорректной логики работы.
###

###
Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
###

####
Возможно, если выполняем UPDATE множества строк в нескольких сессиях (для простоты рассмотрим две), с разным порядком апдейта строк. Строки блокируются не одномоментно, а последовательно, по мере их обновления. Для демонстрации возьмем за основу книгу Егора Рогова - PostgreSQL 17. Изнутри.
####
```sh
--предварительно вернем значение deadlock_timeout в исходное:
otus_dba1=# show deadlock_timeout;
 deadlock_timeout 
------------------
 200ms
(1 row)

otus_dba1=# alter system set deadlock_timeout = '1s';
ALTER SYSTEM

otus_dba1=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

otus_dba1=# show deadlock_timeout;
 deadlock_timeout 
------------------
 1s
(1 row)

otus_dba1=#

--очистим таблицу testnm.tb_lock
otus_dba1=# truncate table testnm.tb_lock;
TRUNCATE TABLE
otus_dba1=# select * from testnm.tb_lock;
 id | created_at | amount 
----+------------+--------
(0 rows)

otus_dba1=#

--вставим те же 3 строки, и убедимся, что версии сттрок расположены в блоке данных (странице) в порядке возрастания amount; затем соберем статистику на таблицу и просмотрим содержимое таблицы для проверки:
otus_dba1=# insert into testnm.tb_lock values (1, now(), 100);
INSERT 0 1
otus_dba1=# insert into testnm.tb_lock values (2, now(), 200);
INSERT 0 1
otus_dba1=# insert into testnm.tb_lock values (3, now(), 300);
INSERT 0 1
otus_dba1=# analyze testnm.tb_lock;
ANALYZE
otus_dba1=# select ctid, * from testnm.tb_lock;
 ctid  | id |          created_at           | amount 
-------+----+-------------------------------+--------
 (0,1) |  1 | 2026-05-03 12:04:46.780734+03 |    100
 (0,2) |  2 | 2026-05-03 12:04:52.594175+03 |    200
 (0,3) |  3 | 2026-05-03 12:04:58.434865+03 |    300
(3 rows)

otus_dba1=#

--создадим индекс по полю amount в обратном порядке на убывание; проверим информацию по индексам рассматриваемой таблицы и проверим отсутствие невалидных индексов в СУБД:
otus_dba1=# create index idx_tb_lock_amount on testnm.tb_lock (amount DESC);
CREATE INDEX

otus_dba1=# select * from pg_indexes where tablename = 'tb_lock' \gx
-[ RECORD 1 ]---------------------------------------------------------------------------
schemaname | testnm
tablename  | tb_lock
indexname  | idx_tb_lock_amount
tablespace | 
indexdef   | CREATE INDEX idx_tb_lock_amount ON testnm.tb_lock USING btree (amount DESC)

otus_dba1=# select * from pg_index where indisvalid = false or indisready = false or indislive = false \gx
(0 rows)

otus_dba1=#

--для того, чтобы поймать взаимоблокировку для 2 сессий, создадим функцию с pg_sleep, которая будет ждать 3 секунды для перехода на обработку новой строки (т.к. строк всего 3):
otus_dba1=# create function testnm.inc_slow (n numeric)
returns numeric
as $$
select pg_sleep(3);
select n + 100;
$$ language sql;
CREATE FUNCTION
otus_dba1=# 

otus_dba1=# \df testnm.*
                         List of functions
 Schema |   Name   | Result data type | Argument data types | Type 
--------+----------+------------------+---------------------+------
 testnm | inc_slow | numeric          | n numeric           | func
(1 row)

otus_dba1=#

--сессия 1 будет обновлять все строки таблицы в том порядке, в котором они расположены в таблице.
--Оптимизатор использует SeqScan, т.к. ему выгоднее не делать лишние чтения по индексу (размер таблицы минимальный, она рсполагается в 1 странице).

otus_dba1=# select relname, relpages, reltuples from pg_class where relname = 'tb_lock';
 relname | relpages | reltuples 
---------+----------+-----------
 tb_lock |        1 |         3
(1 row)

otus_dba1=#

--смотрим теоретический план выполнения апдейта:
otus_dba1=# explain
otus_dba1-# update testnm.tb_lock set amount = testnm.inc_slow(amount);
                          QUERY PLAN                          
--------------------------------------------------------------
 Update on tb_lock  (cost=0.00..1.78 rows=0 width=0)
   ->  Seq Scan on tb_lock  (cost=0.00..1.78 rows=3 width=38)
(2 rows)

otus_dba1=#

--открываем транзакцию в сессии 1, сохраняем информацию по pid и xid и запускаем апдейт всех строк (оптимизатор идет через SeqScan):
otus_dba1=# start transaction;
START TRANSACTION
otus_dba1=*# select pg_backend_pid(), txid_current();
 pg_backend_pid | txid_current 
----------------+--------------
           4985 |     12199131
(1 row)

otus_dba1=*#
otus_dba1=*# update testnm.tb_lock set amount = testnm.inc_slow(amount);

--открываем транзакцию в сессии 2, сохраняем информацию по pid и xid, запускаем апдейт всех строк таблицы. Чтобы апдейт пошел в обратном порядке, дадим инструкцию оптмиизатору не использовать метод доступа SeqScan - пойдет по имеющемуся индексу.
!!!Не получилось, оптимизатор по-прежнему, идет через SeqScan - странно. Даже, когда сильно снижаешь стоимость рандомного доступа:
otus_dba1=# show enable_seqscan;
 enable_seqscan 
----------------
 on
(1 row)

otus_dba1=# set enable_seqscan = off;
SET
otus_dba1=# show enable_seqscan;
 enable_seqscan 
----------------
 off
(1 row)

otus_dba1=# explain
otus_dba1-# update testnm.tb_lock set amount = testnm.inc_slow(amount);
                          QUERY PLAN                          
--------------------------------------------------------------
 Update on tb_lock  (cost=0.00..1.78 rows=0 width=0)
   ->  Seq Scan on tb_lock  (cost=0.00..1.78 rows=3 width=38)
         Disabled: true
(3 rows)

otus_dba1=#

otus_dba1=# select name, setting from pg_settings where name like '%cost%';
             name             | setting 
------------------------------+---------
 autovacuum_vacuum_cost_delay | 2
 autovacuum_vacuum_cost_limit | -1
 cpu_index_tuple_cost         | 0.005
 cpu_operator_cost            | 0.0025
 cpu_tuple_cost               | 0.01
 jit_above_cost               | 100000
 jit_inline_above_cost        | 500000
 jit_optimize_above_cost      | 500000
 parallel_setup_cost          | 1000
 parallel_tuple_cost          | 0.1
 random_page_cost             | 4
 seq_page_cost                | 1
 track_cost_delay_timing      | off
 vacuum_cost_delay            | 0
 vacuum_cost_limit            | 200
 vacuum_cost_page_dirty       | 20
 vacuum_cost_page_hit         | 1
 vacuum_cost_page_miss        | 2
(18 rows)

otus_dba1=# set random_page_cost = 1;
SET
otus_dba1=# show enable_seqscan;                                 
 enable_seqscan 
----------------
 off
(1 row)

otus_dba1=# explain                                              
update testnm.tb_lock set amount = testnm.inc_slow(amount);
                          QUERY PLAN                          
--------------------------------------------------------------
 Update on tb_lock  (cost=0.00..1.78 rows=0 width=0)
   ->  Seq Scan on tb_lock  (cost=0.00..1.78 rows=3 width=38)
         Disabled: true
(3 rows)

otus_dba1=#

otus_dba1=# show seq_page_cost;
 seq_page_cost 
---------------
 10
(1 row)

otus_dba1=# show random_page_cost;
 random_page_cost 
------------------
 1
(1 row)

otus_dba1=# show enable_indexscan;
 enable_indexscan 
------------------
 on
(1 row)

otus_dba1=# show enable_seqscan;
 enable_seqscan 
----------------
 off
(1 row)

otus_dba1=#

--Смущает наличие в теоретическом плане строки Disabled: true. Говорит о том, что планировщик вынужден использовать запрещенную Node в плане - SeqScan, т.к. другие использовать не может (индекс).
```

