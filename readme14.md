
## TERMINAL 1

`sudo pg_createcluster 14 vm1`
`sudo pg_ctlcluster 14 vm1 start`

`create table test (t1ID integer,textvalue varchar(50));`
`create table test2 (t2ID integer,textvalue varchar(50));`

`ALTER SYSTEM SET wal_level = logical;`
`CREATE PUBLICATION vm1_vm2 FOR TABLE test;`
`\dRp+`
```                            Publication vm1_vm2
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test"
```    

`\password`

## TERMINAL 2
`sudo pg_createcluster 14 vm2`
`sudo pg_ctlcluster 14 vm2 start`
`create table test (t1ID integer,textvalue varchar(50));`
`create table test2 (t2ID integer,textvalue varchar(50));`

`ALTER SYSTEM SET wal_level = logical;`
`CREATE PUBLICATION vm2_vm1 FOR TABLE test2;`
`\dRp+`
```                            Publication vm2_vm1
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test2"
```
`CREATE SUBSCRIPTION vm1_vm2_S
CONNECTION 'host=localhost port=5433 user=postgres password=1234567 dbname=otus14'
PUBLICATION vm1_vm2 WITH (copy_data = true);`

` \dRs`
```            List of subscriptions
   Name    |  Owner   | Enabled | Publication
-----------+----------+---------+-------------
 vm1_vm2_s | postgres | t       | {vm1_vm2}
```
## TERMINAL 1
`CREATE SUBSCRIPTION vm2_vm1_S
CONNECTION 'host=localhost port=5434 user=postgres password=1234567 dbname=otus14' 
PUBLICATION vm2_vm1 WITH (copy_data = true);`

`\dRs`
```            List of subscriptions
   Name    |  Owner   | Enabled | Publication
-----------+----------+---------+-------------
 vm2_vm1_s | postgres | t       | {vm2_vm1}
```


## TERMINAL 3
`sudo pg_createcluster 14 vm3`
`sudo pg_ctlcluster 14 vm3 start`
`create table test (t1ID integer,textvalue varchar(50));`
`create table test2 (t2ID integer,textvalue varchar(50));`


`CREATE SUBSCRIPTION vm1_vm3_S
CONNECTION 'host=localhost port=5433 user=postgres password=1234567 dbname=otus14' 
PUBLICATION vm1_vm2 WITH (copy_data = true);`

`CREATE SUBSCRIPTION vm2_vm3_S
CONNECTION 'host=localhost port=5434 user=postgres password=1234567 dbname=otus14' 
PUBLICATION vm2_vm1 WITH (copy_data = true);`

## TERMINAL 4
`sudo pg_createcluster 14 vm4`

`sudo -u postgres pg_basebackup -p 5435 -R -D /var/lib/postgresql/14/vm4`
меняем настройки порта
`sudo echo 'port = 5436' >> /var/lib/postgresql/14/vm4/postgresql.auto.conf`
включаем доступ на чтение
`sudo echo 'hot_standby = on' >> /var/lib/postgresql/14/vm4/postgresql.auto.conf`
запускаем
`sudo pg_ctlcluster 14 vm4 start`

# вставка данных
## TERMINAL 1
`insert into test(t1ID,textvalue) values (1,'TESTVAL');`
# TERMINAL 2
`insert into test2(t2ID,textvalue) values(2,'TEST2_VAL');`

проверка вставки
## TERMINAL 1 
`select * from test2;`
``` t2id | textvalue
------+-----------
    2 | TEST2_VAL
```
## TERMINAL 2
```select * from test;
 t1id | textvalue
------+-----------
    1 | TESTVAL
```
## TERMINAL 3
`select * from test;`
``` t1id | textvalue
------+-----------
    1 | TESTVAL
```
` select * from test2;`
``` t2id | textvalue
------+------------
    2 | TEST2_VAL
```
## TERMINAL 4
`select * from test;`

``` t1id | textvalue
------+-----------
    1 | TESTVAL
```

`select * from test2;`
``` t2id | textvalue
------+------------
    2 | TEST2_VAL
```
Получили логическую репликацию таблицы test c VM1 на VM2 и VM3, логическую репликацию таблицы test2  c VM2 на VM1 и VM3
и репликацию высокой доступности VM3 на VM4
