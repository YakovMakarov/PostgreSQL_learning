# Запуск с параметрами по умолчанию

pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
starting vacuum...end.
...
tps = 264.897224 (without initial connection time)

# Вариант 1 
- изменения относительно по молчанию

 autovacuum_analyze_scale_factor       | 0.05
 autovacuum_naptime                    | 1 
 autovacuum_vacuum_cost_delay          | 2 
 autovacuum_vacuum_insert_scale_factor | 0.2 
 autovacuum_vacuum_scale_factor        | 0.05
 
 pgbench -c8 -P 60 -T 600 -U postgres postgres
Password:
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
starting vacuum...end.
...
tps = 265.793894 (without initial connection time)

# Вариант 2
- изменения относительно предыдущего варианта

 autovacuum_analyze_scale_factor       | 0.15      
 autovacuum_vacuum_insert_scale_factor | 0.2
 autovacuum_vacuum_scale_factor        | 0.15 
 autovacuum_vacuum_threshold           | 50        

pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
starting vacuum...end.
...
tps = 293.115407 (without initial connection time)

# вариант 3
 autovacuum_analyze_scale_factor       | 0.01     
 autovacuum_vacuum_scale_factor        | 0.01
 autovacuum_vacuum_threshold           | 20  
 
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
starting vacuum...end.
...
tps = 262.606562 (without initial connection time)

# вариант 4
autovacuum_vacuum_insert_scale_factor | 0.2       
autovacuum_work_mem                   | 65535     

pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
starting vacuum...end.

tps = 264.453438 (without initial connection time)

# вариант 5

 autovacuum_analyze_threshold          | 100     
 
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
starting vacuum...end.
tps = 265.794196 (without initial connection time)

# вывод 
Судя по тому что изменения параметров не сильно влияют на TPS упираемся в производительность дисковой подсистемы
