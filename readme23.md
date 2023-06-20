`DROP SCHEMA IF EXISTS pract_functions CASCADE;`
`CREATE SCHEMA pract_functions;`

`SET search_path = pract_functions, publ`

-- товары:
```sql
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
```
```sql
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);
```
-- Продажи
```sql
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);
```

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу
```sql
CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);
```
```sql
CREATE OR REPLACE FUNCTION tr_update_sums_each_statement()
   RETURNS trigger
   LANGUAGE plpgsql
AS
$$
DECLARE
    x_rec record;
BEGIN
    IF (TG_OP = 'UPDATE' OR TG_OP = 'DELETE') THEN
        FOR x_rec IN SELECT * FROM old_table LOOP
            BEGIN
            -- уменьшаем сумму
            UPDATE good_sum_mart T
            SET sum_sale = sum_sale - G.good_price*x_rec.sales_qty 
            from  goods as G
            where G.goods_id = x_rec.good_id and  G.good_name = T.good_name;
            -- удаляем строки по которым нет продаж
            delete from good_sum_mart T
            where T.sum_sale=0;
            END;
        END loop;
    END IF;
    IF (TG_OP = 'INSERT' OR TG_OP = 'UPDATE') THEN
        FOR x_rec IN SELECT * FROM new_table LOOP
            BEGIN
            -- обновляем по кторым были продажи
            UPDATE good_sum_mart T
            SET sum_sale = sum_sale + G.good_price*x_rec.sales_qty 
            from  goods as G
            where G.goods_id = x_rec.good_id and  G.good_name = T.good_name;
            -- добавляем те которые ранее не продавали
            insert into good_sum_mart (good_name, sum_sale)
            SELECT G.good_name , G.good_price*x_rec.sales_qty  --INTO g_name, g_price
                  FROM goods as G 
                  left join good_sum_mart as s on s.good_name = G.good_name
                  where G.goods_id = x_rec.good_id and s.good_name is null;
            --raise notice 'g_name: %', g_name;                  
            --raise notice 'g_price: %', g_price; 
            END;
        END loop;
    END IF;

    RETURN NULL;
END;
$$;
```
```sql
CREATE TRIGGER tr_update_sums_insert
   AFTER INSERT ON sales
   REFERENCING NEW TABLE AS new_table
   FOR EACH STATEMENT
   EXECUTE PROCEDURE tr_update_sums_each_statement();
```
```sql
CREATE TRIGGER tr_update_sums_update
   AFTER UPDATE ON sales
   REFERENCING NEW TABLE AS new_table OLD TABLE AS old_table
   FOR EACH STATEMENT
   EXECUTE PROCEDURE tr_update_sums_each_statement();
```
```sql
CREATE TRIGGER tr_update_sums_delete
   AFTER DELETE ON sales
   REFERENCING OLD TABLE AS old_table
   FOR EACH STATEMENT
   EXECUTE PROCEDURE tr_update_sums_each_statement();
```  
-- добавляем продажи
`INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);`
--из витрины
`select * from good_sum_mart;`
```
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 rows)
```
--из отчета
```sql
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```
```
        good_name         |     sum
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        65.50
(2 rows)
```

`update sales set sales_qty=30 where sales_id=1;`
--из витрины
`select * from good_sum_mart;`
```
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        75.50
(2 rows)
```
--из отчета
```sql
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```
```
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        75.50
(2 rows)
```
`delete from sales where sales_id = 2;`
--из витрины
`select * from good_sum_mart;`
```
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        75.00
(2 rows)
```
--из отчета
```sql
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```
```
        good_name         |     sum
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        75.00
(2 rows)
```
`delete from sales where good_id = 1;`
`select * from good_sum_mart;`
--из витрины
```
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
(1 row)
```
--из отчета
```sql
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```
```
        good_name         |     sum
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
(1 row)
```
## Задание со звездочкой*
в данной схеме (без хранения истории цены) витрина предпочтительнее потому что при продаже сохраняем сумму с учетом цены действующей на момент продаж. Но в то же время если после изменения цены произойдет удаление продажи или изменение количества то отчет будет неверен.  Я бы добавл возможность ведения истории цен на товар,и в продажах тоже добавить дату сделки. Эти изменения потребуют изменения отчета и соответственно триггеров. Или как вариант в сделках хранить цену товара по которой производилась сделка.
