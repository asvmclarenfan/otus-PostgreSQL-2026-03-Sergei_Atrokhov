###
Реализовать свой миникластер на трех виртуальных машинах
###

###
1. Настройте ВМ1:
Создайте таблицу test, которая будет для операций записи
Создайте таблицу test2, которая будет для чтения
Настройте публикацию таблицы test
###
```sh
otus_dba1=# 
otus_dba1=# select name, setting, context from pg_settings where name = 'wal_level';
   name    | setting |  context   
-----------+---------+------------
 wal_level | replica | postmaster
(1 row)

otus_dba1=# alter system set wal_level = 'logical';
ALTER SYSTEM
otus_dba1=# \q
postgres@asvpg:~$


```
