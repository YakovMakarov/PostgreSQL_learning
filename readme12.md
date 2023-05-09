
```sql
create database ot12;
```
настройки от pgtune:
```sql
ALTER SYSTEM SET  max_connections = '200';
ALTER SYSTEM SET  shared_buffers = '512MB';
ALTER SYSTEM SET  effective_cache_size = '1536MB';
ALTER SYSTEM SET  maintenance_work_mem = '128MB';
ALTER SYSTEM SET  checkpoint_completion_target = '0.9';
ALTER SYSTEM SET  wal_buffers = '16MB';
ALTER SYSTEM SET  default_statistics_target = '100';
ALTER SYSTEM SET  random_page_cost = '4';
ALTER SYSTEM SET  effective_io_concurrency = '2';
ALTER SYSTEM SET  work_mem = '1310kB';
ALTER SYSTEM SET  min_wal_size = '1GB';
ALTER SYSTEM SET  max_wal_size = '4GB';
```

```shell
sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 ot12
```
```
pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 152.3 tps, lat 315.705 ms stddev 171.680, 0 failed
progress: 20.0 s, 183.9 tps, lat 272.421 ms stddev 122.291, 0 failed
progress: 30.0 s, 182.4 tps, lat 273.994 ms stddev 117.129, 0 failed
progress: 40.0 s, 172.4 tps, lat 288.538 ms stddev 128.355, 0 failed
progress: 50.0 s, 184.3 tps, lat 271.707 ms stddev 116.379, 0 failed
progress: 60.0 s, 169.6 tps, lat 294.810 ms stddev 131.603, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 10499
number of failed transactions: 0 (0.000%)
latency average = 286.083 ms
latency stddev = 132.833 ms
initial connection time = 225.313 ms
tps = 173.883232 (without initial connection time)
```

--отключил в postgresql.conf
--fsync
--Syncronous_commit
--full_page_writes

```shell
 sudo -u postgres pgbench -i ot12
```
```
pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 529.5 tps, lat 91.734 ms stddev 27.650, 0 failed
progress: 20.0 s, 548.3 tps, lat 91.253 ms stddev 37.200, 0 failed
progress: 30.0 s, 548.9 tps, lat 91.044 ms stddev 28.120, 0 failed
progress: 40.0 s, 528.5 tps, lat 94.448 ms stddev 31.164, 0 failed
progress: 50.0 s, 483.8 tps, lat 103.547 ms stddev 75.700, 0 failed
progress: 60.0 s, 511.1 tps, lat 97.522 ms stddev 41.304, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 31551
number of failed transactions: 0 (0.000%)
latency average = 94.887 ms
latency stddev = 43.166 ms
initial connection time = 235.028 ms
tps = 525.937113 (without initial connection time)
```

```sql
ALTER SYSTEM SET  work_mem  = '256MB';
```

 
```shell
sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 ot12
```
```
pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 575.7 tps, lat 84.577 ms stddev 23.759, 0 failed
progress: 20.0 s, 598.5 tps, lat 83.537 ms stddev 22.535, 0 failed
progress: 30.0 s, 589.7 tps, lat 84.772 ms stddev 22.488, 0 failed
progress: 40.0 s, 580.6 tps, lat 86.127 ms stddev 31.108, 0 failed
progress: 50.0 s, 593.6 tps, lat 84.214 ms stddev 22.206, 0 failed
progress: 60.0 s, 561.9 tps, lat 89.023 ms stddev 25.653, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 35050
number of failed transactions: 0 (0.000%)
latency average = 85.430 ms
latency stddev = 25.004 ms
initial connection time = 217.368 ms
tps = 584.217474 (without initial connection time)
```

```sql
ALTER SYSTEM SET  shared_buffers = '1024MB';
```

```shell
sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 ot12
```

```
pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 477.5 tps, lat 101.817 ms stddev 43.564, 0 failed
progress: 20.0 s, 525.2 tps, lat 95.272 ms stddev 39.918, 0 failed
progress: 30.0 s, 449.7 tps, lat 111.102 ms stddev 53.971, 0 failed
progress: 40.0 s, 469.7 tps, lat 105.059 ms stddev 60.394, 0 failed
progress: 50.0 s, 355.8 tps, lat 142.424 ms stddev 114.092, 0 failed
progress: 60.0 s, 506.4 tps, lat 97.172 ms stddev 42.684, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 27893
number of failed transactions: 0 (0.000%)
latency average = 107.637 ms
latency stddev = 66.299 ms
initial connection time = 224.367 ms
tps = 461.085571 (without initial connection time)
```

 ```sql
ALTER SYSTEM SET  shared_buffers = '128MB';
```

```shell
sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 ot12
```
```
pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 580.9 tps, lat 83.752 ms stddev 27.803, 0 failed
progress: 20.0 s, 611.8 tps, lat 81.743 ms stddev 21.750, 0 failed
progress: 30.0 s, 596.2 tps, lat 83.829 ms stddev 22.591, 0 failed
progress: 40.0 s, 585.9 tps, lat 85.327 ms stddev 27.165, 0 failed
progress: 50.0 s, 606.7 tps, lat 82.444 ms stddev 23.551, 0 failed
progress: 60.0 s, 611.6 tps, lat 81.752 ms stddev 21.697, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 35982
number of failed transactions: 0 (0.000%)
latency average = 83.203 ms
latency stddev = 24.351 ms
initial connection time = 223.074 ms
tps = 599.802856 (without initial connection time)
```
### результат pgbench
после отключения "надежности" максимально получен tps -  599.802856.
С включенной "надежностью" -  tps -  525.937113. Прирост 14 %

## sysbench-tpcc
### установка
```shell
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```
### подготовка
```shell
sysbench /usr/share/sysbench/tpcc.lua --pgsql-user=postgres --pgsql-password=pwd  --pgsql-db=ot12 --time=120 --threads=4 --report-interval=1 --tables=1 --scale=5 --use_fk=0  --trx_level=RC --db-driver=pgsql prepare
```
--включил в postgresql.conf
--fsync
--Syncronous_commit
--full_page_writes

### тест
```shell
sysbench /usr/share/sysbench/tpcc.lua --pgsql-user=postgres --pgsql-password=pwd  --pgsql-db=ot12 --time=360 --threads=4 --report-interval=1 --tables=1 --scale=5 --use_fk=0  --trx_level=RC --db-driver=pgsq run
```
```
SQL statistics:
    queries performed:
        read:                            391275
        write:                           407055
        other:                           60156
        total:                           858486
    transactions:                        30074  (83.52 per sec.)
    queries:                             858486 (2384.24 per sec.)
    ignored errors:                      123    (0.34 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          360.0657s
    total number of events:              30074

Latency (ms):
         min:                                    1.17
         avg:                                   47.88
         max:                                  498.16
         95th percentile:                      114.72
         sum:                              1439967.09

Threads fairness:
    events (avg/stddev):           7518.5000/61.84
    execution time (avg/stddev):   359.9918/0.03
```
- tps - 83.52

--отключил в postgresql.conf
--fsync
--Syncronous_commit
--full_page_writes

```shell
sysbench /usr/share/sysbench/tpcc.lua --pgsql-user=postgres --pgsql-password=pwd  --pgsql-db=ot12 --time=360 --threads=4 --report-interval=1 --tables=1 --scale=5 --use_fk=0  --trx_level=RC --db-driver=pgsq run
```
```
SQL statistics:
    queries performed:
        read:                            418426
        write:                           434247
        other:                           64586
        total:                           917259
    transactions:                        32289  (89.68 per sec.)
    queries:                             917259 (2547.51 per sec.)
    ignored errors:                      133    (0.37 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          360.0592s
    total number of events:              32289

Latency (ms):
         min:                                    1.22
         avg:                                   44.60
         max:                                 1376.71
         95th percentile:                      114.72
         sum:                              1439971.40

Threads fairness:
    events (avg/stddev):           8072.2500/86.11
    execution time (avg/stddev):   359.9929/0.02
```
- tps - 89.68

```sql
ALTER SYSTEM SET  shared_buffers = '128MB';
```

```
SQL statistics:
    queries performed:
        read:                            353700
        write:                           367302
        other:                           54432
        total:                           775434
    transactions:                        27212  (75.56 per sec.)
    queries:                             775434 (2153.09 per sec.)
    ignored errors:                      111    (0.31 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          360.1386s
    total number of events:              27212

Latency (ms):
         min:                                    1.36
         avg:                                   52.93
         max:                                 3100.78
         95th percentile:                      132.49
         sum:                              1440208.06

Threads fairness:
    events (avg/stddev):           6803.0000/100.73
    execution time (avg/stddev):   360.0520/0.03
```
- tps - 75.56

```sql
ALTER SYSTEM SET  shared_buffers = '1024MB';
```
 
```
SQL statistics:
    queries performed:
        read:                            293516
        write:                           304558
        other:                           45396
        total:                           643470
    transactions:                        22694  (63.02 per sec.)
    queries:                             643470 (1786.94 per sec.)
    ignored errors:                      110    (0.31 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          360.0949s
    total number of events:              22694

Latency (ms):
         min:                                    1.48
         avg:                                   63.46
         max:                                 2225.75
         95th percentile:                      186.54
         sum:                              1440060.38

Threads fairness:
    events (avg/stddev):           5673.5000/22.69
    execution time (avg/stddev):   360.0151/0.04
```
-tps - 63.02

### результат тестирования sysbench-tpcc
- отключение "надежности" не так заметно влияет  на производительность как при тестировании pg_bench.
максимальный tps - 89.68. С включенной "надежностью" -  tps - 83.52. Прирост 7 %
