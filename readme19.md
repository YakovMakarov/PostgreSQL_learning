создаю базу

`create database otus19;`

`\c otus19`

создаю структуру и наполняю тестовыми данными

# группа
`CREATE TABLE band (bandid INT,bandName VARCHAR(100));`
`INSERT INTO band (bandid, bandName)VALUES (generate_series(1, 20),  'A  band: ' || trunc(random()*1000));`
# музыкант
`CREATE TABLE musicians (musician_id INT, musician_name VARCHAR(100));`
`INSERT INTO musicians(musician_id , musician_name ) VALUES (generate_series(1, 20),  'A cool member: ' || trunc(random()*1000));`
# участие музыканта в группе
`CREATE TABLE band_musicians(link_id INT, band_id INT,musician_id int)`
`INSERT INTO band_musicians   select  generate_series(1, 40),   trunc(random()*20+1),trunc(random()*10*2+1);`
# стиль
`CREATE TABLE genre (genre_id INT, genre_name VARCHAR(100));`
`INSERT INTO genre(genre_id , genre_name ) VALUES (generate_series(1, 20),  'A  genre: ' || trunc(random()*100));`
# в каких стилях играет группа
`CREATE TABLE band_genre(link_id INT, band_id INT,genre_id int);`
`INSERT INTO band_genre   values  (generate_series(1, 40),   trunc(random()*20+1),trunc(random()*20+1));`

## запросы
- участники не занятые в группах 
- LEFT JOIN

`SELECT m.* FROM musicians m LEFT JOIN band_musicians bm ON m.musician_id = bm.musician_id WHERE bm IS NULL;`
 ```
 musician_id |   musician_name
-------------+--------------------
           6 | A cool member: 349
          14 | A cool member: 987
          20 | A cool member: 758
```

- группы из которых ушли все участники
- RIGHT JOIN

`SELECT m.* FROM band_musicians bm RIGHT  JOIN band b   ON b.bandid = bm.band_id WHERE bm IS NULL;`
 ```
bandid |   bandname
--------+--------------
      6 | A  band: 519
     14 | A  band: 435
     15 | A  band: 41
```

- состав групп
- INNER JOIN

`SELECT b.bandName,m.musician_name FROM band b INNER JOIN band_musicians bm ON b.bandid = bm.band_id INNER join musicians m ON m.musician_id = bm.musician_id ORDER BY b.bandName;`
```
   bandname   |   musician_name
--------------+--------------------
 A  band: 142 | A cool member: 41
 A  band: 142 | A cool member: 770
 A  band: 142 | A cool member: 162
 A  band: 23  | A cool member: 857
 A  band: 23  | A cool member: 514
 A  band: 288 | A cool member: 64
 A  band: 288 | A cool member: 337
 A  band: 354 | A cool member: 447
 A  band: 354 | A cool member: 888
 A  band: 354 | A cool member: 979
 A  band: 41  | A cool member: 64
 A  band: 41  | A cool member: 413
 A  band: 438 | A cool member: 656
 A  band: 502 | A cool member: 770
 A  band: 502 | A cool member: 41
 A  band: 502 | A cool member: 770
 A  band: 502 | A cool member: 337
 A  band: 520 | A cool member: 162
 A  band: 55  | A cool member: 337
 A  band: 55  | A cool member: 826
 A  band: 70  | A cool member: 41
 A  band: 820 | A cool member: 888
 A  band: 820 | A cool member: 857
 A  band: 820 | A cool member: 447
 A  band: 820 | A cool member: 409
 A  band: 820 | A cool member: 41
 A  band: 86  | A cool member: 888
 A  band: 86  | A cool member: 826
 A  band: 861 | A cool member: 826
 A  band: 861 | A cool member: 216
 A  band: 883 | A cool member: 337
 A  band: 892 | A cool member: 514
 A  band: 945 | A cool member: 846
 A  band: 945 | A cool member: 41
 A  band: 945 | A cool member: 979
 A  band: 972 | A cool member: 216
 A  band: 972 | A cool member: 216
 A  band: 972 | A cool member: 979
 A  band: 972 | A cool member: 337
 A  band: 972 | A cool member: 482
```

- если бы в какждой групппе участвовали все возможные музыканты
- CROSS JOIN

`SELECT * FROM band,musicians;`
``` 
bandid |   bandname   | musician_id |   musician_name
--------+--------------+-------------+--------------------
      1 | A  band: 945 |           1 | A cool member: 826
      1 | A  band: 945 |           2 | A cool member: 482
      1 | A  band: 945 |           3 | A cool member: 514
      1 | A  band: 945 |           4 | A cool member: 41
      1 | A  band: 945 |           5 | A cool member: 656
      1 | A  band: 945 |           6 | A cool member: 349
     ............................................... 
(400 rows)
   ```


- список групп с стилями в которых исполняет группа, в том числе группы с неопределнными стилями
- LEFT JOIN

  `select b.bandname,g.genre_name from band b left join band_genre bg on b.bandid = bg.band_id left join genre g on g.genre_id = bg.genre_id order by bandname;`
```
   bandname   |  genre_name
--------------+--------------
 A  band: 142 | A  genre: 96
 A  band: 142 | A  genre: 37
 A  band: 142 | A  genre: 96
 A  band: 142 | A  genre: 37
 A  band: 23  | A  genre: 92
 A  band: 288 |
 A  band: 354 | A  genre: 80
 A  band: 41  | A  genre: 11
 A  band: 41  | A  genre: 41
 A  band: 41  | A  genre: 53
 A  band: 435 | A  genre: 53
 A  band: 435 | A  genre: 17
 A  band: 438 |
 A  band: 502 | A  genre: 54
 A  band: 502 | A  genre: 41
 A  band: 502 | A  genre: 25
 A  band: 519 | A  genre: 80
 A  band: 519 | A  genre: 92
 A  band: 519 | A  genre: 80
 A  band: 519 | A  genre: 92
 A  band: 519 | A  genre: 11
 A  band: 520 | A  genre: 96
 A  band: 520 | A  genre: 49
 A  band: 55  | A  genre: 41
 A  band: 55  | A  genre: 48
 A  band: 70  |
 A  band: 820 | A  genre: 80
 A  band: 820 | A  genre: 37
 A  band: 820 | A  genre: 80
 A  band: 820 | A  genre: 41
 A  band: 86  | A  genre: 11
 A  band: 86  | A  genre: 49
 A  band: 86  | A  genre: 25
 A  band: 86  | A  genre: 14
 A  band: 861 | A  genre: 30
 A  band: 883 | A  genre: 53
 A  band: 883 | A  genre: 96
 A  band: 883 | A  genre: 96
 A  band: 892 | A  genre: 41
 A  band: 945 | A  genre: 17
 A  band: 972 | A  genre: 53
 A  band: 972 | A  genre: 65
 A  band: 972 | A  genre: 53
```
- список групп участники которых не определились со стилем
- INNER JOIN + LEFT JOIN

  `SELECT distinct b.bandname FROM band b INNER JOIN band_musicians bm ON b.bandid = bm.band_id LEFT JOIN band_genre bg on b.bandid = bg.band_id WHERE bg.band_id IS NULL;`
```   
   bandname
--------------
 A  band: 288
 A  band: 438
 A  band: 70
```
# структура
```
              List of relations
 Schema |      Name      | Type  |  Owner
--------+----------------+-------+----------
 public | band           | table | postgres
 public | band_genre     | table | postgres
 public | band_musicians | table | postgres
 public | genre          | table | postgres
 public | musicians      | table | postgres
```
```
                                                   Table "public.band"
  Column  |          Type          | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
----------+------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
 bandid   | integer                |           |          |         | plain    |             |              |
 bandname | character varying(100) |           |          |         | extended |             |              |
```
```
                                                    Table "public.genre"
   Column   |          Type          | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
------------+------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
 genre_id   | integer                |           |          |         | plain    |             |              |
 genre_name | character varying(100) |           |          |         | extended |             |              |
```
```
                                                   Table "public.musicians"
    Column     |          Type          | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
---------------+------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
 musician_id   | integer                |           |          |         | plain    |             |              |
 musician_name | character varying(100) |           |          |         | extended |             |              |
```
```
                                          Table "public.band_genre"
  Column  |  Type   | Collation | Nullable | Default | Storage | Compression | Stats target | Description
----------+---------+-----------+----------+---------+---------+-------------+--------------+-------------
 link_id  | integer |           |          |         | plain   |             |              |
 band_id  | integer |           |          |         | plain   |             |              |
 genre_id | integer |           |          |         | plain   |             |              |
```
```
                                          Table "public.band_musicians"
   Column    |  Type   | Collation | Nullable | Default | Storage | Compression | Stats target | Description
-------------+---------+-----------+----------+---------+---------+-------------+--------------+-------------
 link_id     | integer |           |          |         | plain   |             |              |
 band_id     | integer |           |          |         | plain   |             |              |
 musician_id | integer |           |          |         | plain   |             |              |
```
