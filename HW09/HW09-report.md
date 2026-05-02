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

```
