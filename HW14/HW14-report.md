###
В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales)
###
```sh
otus_dba1=# CREATE SCHEMA pract_functions;
SET search_path = pract_functions, public
CREATE SCHEMA

otus_dba1=# select nspname from pg_namespace;
      nspname       
--------------------
 pg_toast
 pg_catalog
 public
 information_schema
 testnm
 my_schema
 pract_functions
(7 rows)

otus_dba1=#

--товары
otus_dba1=# CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
CREATE TABLE
otus_dba1=# 
otus_dba1=# INSERT INTO goods (goods_id, good_name, good_price)
VALUES  (1, 'Спички хозяйственные', .50),
                (2, 'Автомобиль Ferrari FXX K', 185000000.01);
INSERT 0 2
otus_dba1=# 
otus_dba1=# select * from goods;
 goods_id |        good_name         |  good_price  
----------+--------------------------+--------------
        1 | Спички хозяйственные     |         0.50
        2 | Автомобиль Ferrari FXX K | 185000000.01
(2 rows)

otus_dba1=#


--продажи
otus_dba1=# CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    goods_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);
CREATE TABLE
otus_dba1=# 
otus_dba1=# INSERT INTO sales (goods_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
INSERT 0 4
otus_dba1=#

otus_dba1=# select * from sales;
 sales_id | goods_id |          sales_time           | sales_qty 
----------+----------+-------------------------------+-----------
        1 |        1 | 2026-07-18 21:13:21.403206+03 |        10
        2 |        1 | 2026-07-18 21:13:21.403206+03 |         1
        3 |        1 | 2026-07-18 21:13:21.403206+03 |       120
        4 |        2 | 2026-07-18 21:13:21.403206+03 |         1
(4 rows)

otus_dba1=#

--запрос для генерации отчета – сумма продаж по каждому товару.
otus_dba1=# SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.goods_id = G.goods_id
GROUP BY G.good_name;
        good_name         |     sum      
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозяйственные     |        65.50
(2 rows)

otus_dba1=#


--БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.
otus_dba1=# CREATE TABLE good_sum_mart
(
        good_name   varchar(63) NOT NULL,
        sum_sale        numeric(16, 2)NOT NULL
);
CREATE TABLE
otus_dba1=#
```

###
Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину).
Не забыть, что кроме INSERT есть еще UPDATE и DELETE
###
```sh
otus_dba1=# CREATE OR REPLACE FUNCTION tr_mart()
RETURNS trigger
AS
$$
DECLARE
    v_name      text;
    v_summa_ins numeric(16, 2);
    v_summa_del numeric(16, 2);

BEGIN
    RAISE NOTICE 'TG_OP = %', TG_OP;
    
    CASE TG_OP
        WHEN 'INSERT' THEN
            SELECT G.good_name, (G.good_price * NEW.sales_qty)
            INTO v_name, v_summa_ins
            FROM goods G
            WHERE G.goods_id = NEW.goods_id;
    
            INSERT INTO good_sum_mart (good_name, sum_sale)
            VALUES(v_name, v_summa_ins)
            ON CONFLICT (good_name)
            DO UPDATE 
            SET sum_sale = good_sum_mart.sum_sale + v_summa_ins;
        
            RETURN NEW;
       
        WHEN 'DELETE' THEN
            SELECT G.good_name, (G.good_price * OLD.sales_qty)
            INTO v_name, v_summa_del
            FROM goods G
            WHERE G.goods_id = OLD.goods_id;        
        
            UPDATE good_sum_mart
            SET sum_sale = sum_sale - v_summa_del
            WHERE good_name = v_name;
        
            RETURN NULL;
        
        WHEN 'UPDATE' THEN
            SELECT G.good_name, (G.good_price * NEW.sales_qty), (G.good_price * OLD.sales_qty)
            INTO v_name, v_summa_ins, v_summa_del
            FROM goods G
            WHERE G.goods_id = OLD.goods_id;

            RAISE NOTICE '%  %', v_summa_ins, v_summa_del;
        
            UPDATE good_sum_mart
            SET sum_sale = sum_sale + v_summa_ins - v_summa_del
            WHERE good_name = v_name;
        
            RETURN NEW;

    END CASE;

END;
$$ LANGUAGE plpgsql
    SET search_path = pract_functions, public
    SECURITY DEFINER;
CREATE FUNCTION
otus_dba1=#

otus_dba1=# CREATE OR REPLACE TRIGGER r_mart
AFTER INSERT OR UPDATE OR DELETE
ON sales
FOR ROW
EXECUTE FUNCTION tr_mart();
CREATE TRIGGER
otus_dba1=#
```


###
Проверка
###
```sh
otus_dba1=# select * from good_sum_mart;
 good_name | sum_sale 
-----------+----------
(0 rows)

otus_dba1=#

otus_dba1=# SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
        good_name         |     sum      
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозяйственные     |        65.50
(2 rows)

otus_dba1=#


--пробуем вставить 2 строки в таблицу sales:
otus_dba1=# INSERT INTO sales (goods_id, sales_qty) VALUES (1, 100), (2, 2);
NOTICE:  TG_OP = INSERT
ERROR:  there is no unique or exclusion constraint matching the ON CONFLICT specification
CONTEXT:  SQL statement "INSERT INTO good_sum_mart (good_name, sum_sale)
            VALUES(v_name, v_summa_ins)
            ON CONFLICT (good_name)
            DO UPDATE 
            SET sum_sale = good_sum_mart.sum_sale + v_summa_ins"
PL/pgSQL function tr_mart() line 17 at SQL statement
otus_dba1=#

--Получаем ошибку, т.к. конструкция ON CONFLICT (good_name) требует наличия в таблице ограничения уникальности или PK по указанному столбцу
--добавляем ограничение на уникальность (не PK) на столбец good_name, но данный столбец является не лучшим вариантом для уникальности, лучше перейти на goods_id, но придется вносить изменения в денормализацию (оставим good_name):
otus_dba1=# ALTER TABLE good_sum_mart ADD CONSTRAINT unuque_good_sum_mart_name UNIQUE (good_name);
ALTER TABLE
otus_dba1=#

--Теперь вставка проходит:
otus_dba1=# INSERT INTO sales (goods_id, sales_qty) VALUES (1, 100), (2, 2);
NOTICE:  TG_OP = INSERT
NOTICE:  TG_OP = INSERT
INSERT 0 2
otus_dba1=#

--Проверяем витрину, данные о двух вставках посчитаны:
otus_dba1=# select * from good_sum_mart;
        good_name         |   sum_sale   
--------------------------+--------------
 Спички хозяйственные     |        50.00
 Автомобиль Ferrari FXX K | 370000000.02
(2 rows)

otus_dba1=#

--Триггер не видит исторических данных, а срабатывает только в момент INSERT, UPDATE, DELETE на таблице sales.
--Для того, чтобы уже имеющиеся данные были включены в состав витрины выполняем очистку витрины и вставку через запрос с группировкой для первоначальной инициализации:

otus_dba1=# TRUNCATE TABLE good_sum_mart;
TRUNCATE TABLE
otus_dba1=# 
otus_dba1=# select * from good_sum_mart;
 good_name | sum_sale 
-----------+----------
(0 rows)

otus_dba1=# 
otus_dba1=# INSERT INTO good_sum_mart (good_name, sum_sale)
SELECT G.good_name, SUM(G.good_price * S.sales_qty)
FROM goods G
JOIN sales S ON S.goods_id = G.goods_id
GROUP BY G.good_name;
INSERT 0 2
otus_dba1=# 
otus_dba1=# select * from good_sum_mart;
        good_name         |   sum_sale   
--------------------------+--------------
 Автомобиль Ferrari FXX K | 555000000.03
 Спички хозяйственные     |       115.50
(2 rows)

otus_dba1=#


--Теперь выполним проверку на удаление данных о продаже товаров, которые были добавлены в позапрошлом шаге (DELETE):
otus_dba1=# select * from sales;
 sales_id | goods_id |          sales_time           | sales_qty 
----------+----------+-------------------------------+-----------
        1 |        1 | 2026-07-18 21:13:21.403206+03 |        10
        2 |        1 | 2026-07-18 21:13:21.403206+03 |         1
        3 |        1 | 2026-07-18 21:13:21.403206+03 |       120
        4 |        2 | 2026-07-18 21:13:21.403206+03 |         1
       17 |        1 | 2026-07-18 21:43:21.44593+03  |       100
       18 |        2 | 2026-07-18 21:43:21.44593+03  |         2
(6 rows)

otus_dba1=# delete from sales where sales_id in (17,18);
NOTICE:  TG_OP = DELETE
NOTICE:  TG_OP = DELETE
DELETE 2
otus_dba1=#

--Проверяем, что всё вернулось к начальным цифрам:
otus_dba1=# select * from good_sum_mart;
        good_name         |   sum_sale   
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 rows)

otus_dba1=#


--Теперь выполним обновление числа товаров (спички уменьшим на 20 шт, а автомобиль увеличим на единицу) и проверим изменения в витрине:
otus_dba1=# update sales set sales_qty = 100 where sales_id = 3;
NOTICE:  TG_OP = UPDATE
NOTICE:  50.00  60.00
UPDATE 1
otus_dba1=# update sales set sales_qty = 2 where sales_id = 4;
NOTICE:  TG_OP = UPDATE
NOTICE:  370000000.02  185000000.01
UPDATE 1
otus_dba1=#

otus_dba1=# select * from sales;
 sales_id | goods_id |          sales_time           | sales_qty 
----------+----------+-------------------------------+-----------
        1 |        1 | 2026-07-18 21:13:21.403206+03 |        10
        2 |        1 | 2026-07-18 21:13:21.403206+03 |         1
        3 |        1 | 2026-07-18 21:13:21.403206+03 |       100
        4 |        2 | 2026-07-18 21:13:21.403206+03 |         2
(4 rows)

otus_dba1=# 
otus_dba1=# select * from good_sum_mart;
        good_name         |   sum_sale   
--------------------------+--------------
 Спички хозяйственные     |        55.50
 Автомобиль Ferrari FXX K | 370000000.02
(2 rows)

otus_dba1=#

--Данные корректны.
```


###
Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
Подсказка: В реальной жизни возможны изменения цен.
###
####

Витрина позволяет хранить и обновлять информацию в онлайне\без задержек. Если добавить в витрину столбец с датой (например, с DEFAULT now()), товместо обновления имеющейся строки будет добавляться новая с указанием времени и можно проследить историю изменения - аля аудит.

Изменение цены в таблице goods не изменит уже записанные в витрину суммы за прошлые периоды. Витрина хранит исторические данные по продажам, а не формулу их расчета. Важно разделять онлайн 'справочники' (текущие цены) и исторические данные (информация по уже осуществленным продажам на момент в прошлом).

Цена товара хранится только в таблице goods, если использовать 'отчет по требованию', то при изменении цены (например, по ошибке), будет выполнен перерасчет по этой ошибочной цене, вследствие чего будет потеря данных\информации о финансовых операциях за какое-то время в прошлом (искажение финансовой отчетности).
####
