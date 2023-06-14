скачал базу
`sudo wget  -P /home/yasha/ "https://edu.postgrespro.ru/demo-small.zip"`
распаковал
`unzip demo-small.zip`
(данные за 3 месяца, буду делать секции по месяцу)
выполнил скрипт 
`sudo psql -p 5432 -f /home/yasha/demo-small-20170815.sql -U postgres`

исходное состоние
`\d bookings`
```                        Table "bookings.bookings"
    Column    |           Type           | Collation | Nullable | Default
--------------+--------------------------+-----------+----------+---------
 book_ref     | character(6)             |           | not null |
 book_date    | timestamp with time zone |           | not null |
 total_amount | numeric(10,2)            |           | not null |
Indexes:
    "bookings_pkey" PRIMARY KEY, btree (book_ref)
Referenced by:
    TABLE "tickets" CONSTRAINT "tickets_book_ref_fkey" FOREIGN KEY (book_ref) REFERENCES bookings(book_ref)
```

копия таблицы куда будут перенесены данные
`CREATE TABLE bookings_t (LIKE bookings INCLUDING ALL ) partition by HASH (book_ref);`
`CREATE TABLE bookings_part0 PARTITION OF bookings_t FOR VALUES WITH (MODULUS 3, REMAINDER 0);`
`CREATE TABLE bookings_part1 PARTITION OF bookings_t FOR VALUES WITH (MODULUS 3, REMAINDER 1);`
`CREATE TABLE bookings_part2 PARTITION OF bookings_t FOR VALUES WITH (MODULUS 3, REMAINDER 2);`

перенос данных 
`INSERT INTO bookings_t SELECT * FROM bookings;`

проверка распределения
`select count(*) from bookings_t;`
 ```
 count
--------
 262788
(1 row)
```
`select count(*) from bookings_part0;`
 ```
 count
-------
 87649
(1 row)
```
`select count(*) from bookings_part1;`
 ```
 count
-------
 87651
(1 row)
```
`select count(*) from bookings_part2;`
 ```
 count
-------
 87488
(1 row)
```
подготовка к переименованию
удаление FK
`Alter table tickets drop constraint tickets_book_ref_fkey;`

переименование
`ALTER TABLE bookings RENAME TO bookings_backup;`
`ALTER TABLE bookings_t RENAME TO bookings;`

возвращаем FK
```ALTER TABLE tickets 
ADD CONSTRAINT tickets_book_ref_fkey 
FOREIGN KEY (book_ref) 
REFERENCES bookings (book_ref);
```


состояние после манипуляций
`\d+ bookings`
```
                                                 Partitioned table "bookings.bookings"
    Column    |           Type           | Collation | Nullable | Default | Storage  | Compression | Stats target |    Description
--------------+--------------------------+-----------+----------+---------+----------+-------------+--------------+--------------------
 book_ref     | character(6)             |           | not null |         | extended |             |              | Booking number
 book_date    | timestamp with time zone |           | not null |         | plain    |             |              | Booking date
 total_amount | numeric(10,2)            |           | not null |         | main     |             |              | Total booking cost
Partition key: HASH (book_ref)
Indexes:
    "bookings_t_pkey" PRIMARY KEY, btree (book_ref)
Referenced by:
    TABLE "tickets" CONSTRAINT "tickets_book_ref_fkey" FOREIGN KEY (book_ref) REFERENCES bookings(book_ref)
Partitions: bookings_part0 FOR VALUES WITH (modulus 3, remainder 0),
            bookings_part1 FOR VALUES WITH (modulus 3, remainder 1),
            bookings_part2 FOR VALUES WITH (modulus 3, remainder 2)
```
