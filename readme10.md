- создал новую виртуальную машину 

- изменил параметры : 
    `alter system set deadlock_timeout = 200;`
    `alter system set log_lock_waits = on;`
- перечитал файл конфигурации: `select pg_reload_conf();`
- проверил: 

    `show log_lock_waits;`
    ```
    log_lock_waits
    ----------------
     on
    (1 строка)
    ```
    `show deadlock_timeout;`
     ```
     deadlock_timeout
    ------------------
     200ms
    (1 строка)
    ```
- создал базу и переключился
    `create database for_locks;`
    `\c for_locks`
    ```
    psql (15.2 (Ubuntu 15.2-1.pgdg20.04+1), сервер 14.7 (Ubuntu 14.7-1.pgdg20.04+1))
    Вы подключены к базе данных "for_locks" как пользователь "postgres".
    ```
- создал таблицу
    `create table books (id int, title varchar(100));`
- вставил данные
    `insert into books(id,title) values (1,'Азбука'),(2,'Чтение');
- проверил
    `select * from books;`
    ```
    id | title
    ----+--------
      1 | Азбука
      2 | Чтение
    (2 строки)
    ```   
- открыл еще 2 терминала подключился к базе for_locks
- во всех трех терминалах начал транзакцию и обновление  данных:
`BEGIN;`
`update books set title = 'Букварь' where id=1;`
- в терминал 4 выполнил (пронумеровал для удобства)
    ```
    select row_number() over (order by pid,locktype) as npp, 
    locktype, pid, relation::regclass, virtualxid as virtxid, transactionid as xid, mode, granted
    from pg_locks
    where pid <> pg_backend_pid()
    order by pid, locktype;
    ```
    ```
     npp |   locktype    |  pid   | relation | virtxid | xid |       mode       | granted
    -----+---------------+--------+----------+---------+-----+------------------+---------
       1 | relation      | 277693 | books    |         |     | RowExclusiveLock | t
       2 | transactionid | 277693 |          |         | 739 | ExclusiveLock    | t
       3 | virtualxid    | 277693 |          | 4/13    |     | ExclusiveLock    | t
       4 | relation      | 294211 | books    |         |     | RowExclusiveLock | t
       5 | transactionid | 294211 |          |         | 740 | ExclusiveLock    | t
       6 | transactionid | 294211 |          |         | 739 | ShareLock        | f
       7 | tuple         | 294211 | books    |         |     | ExclusiveLock    | t
       8 | virtualxid    | 294211 |          | 6/22087 |     | ExclusiveLock    | t
       9 | relation      | 294212 | books    |         |     | RowExclusiveLock | t
      10 | transactionid | 294212 |          |         | 741 | ExclusiveLock    | t
      11 | tuple         | 294212 | books    |         |     | ExclusiveLock    | f
      12 | virtualxid    | 294212 |          | 3/26    |     | ExclusiveLock    | t
    ```
- транзакциям были присвоены виртуальные идентификаторы virtualxid => 4/13, 6/22087, 3/26 (строки 3, 8, 12) и эти номера  заблокированы самими транзакциями в режиме ExclusiveLock
- каждая транзакция получила физический номер 4\13 -> 739, 6/22087 -> 740, 3/26 -> 741 (строки 2,5,10)
- update вызвала блокировку relation (строки 1,4,9)
- вторая транзакция блокирована первой (строка 6) (granted = f) XID = 739 и создала блокировку версии строки (tuple) (строка 7)
- третья транзакция не получила блокировку tuple (строка 11)

- добавил  в таблицу запись для взаимоблокировок трёх транзакицй
`insert into books (id,title) values(3,'Учебник') ;`

- терминал 1:
`begin;`
`update books set title = 'Букварь' where id=1;`
- терминал 2
`begin;`
`update books set title = 'Словарь' where id = 2;`
- терминал 3
`begin;`
`update books set title = 'Математика' where id=3;`
- терминал 1
` update books set title = 'Английский словарь' where id=2;`
- терминал 2
`update books set title = 'Алгебра' where id = 3;`
- терминал 3
`update books set title = 'Физика' where id=1;`
    ```
    ОШИБКА:  обнаружена взаимоблокировка
    ПОДРОБНОСТИ:  Процесс 294212 ожидает в режиме ShareLock блокировку "транзакция 751"; заблокирован процессом 277693.
    Процесс 277693 ожидает в режиме ShareLock блокировку "транзакция 752"; заблокирован процессом 294211.
    Процесс 294211 ожидает в режиме ShareLock блокировку "транзакция 753"; заблокирован процессом 294212.
    ПОДСКАЗКА:  Подробности запроса смотрите в протоколе сервера.
    КОНТЕКСТ:  при изменении кортежа (0,10) в отношении "books"
    ```
- просмотр логов
`tail -n 30 /var/log/postgresql/postgresql-14-lesson10.log`
```
2023-03-16 06:54:40.731 UTC [277693] postgres@for_locks СООБЩЕНИЕ:  процесс 277693 продолжает ожидать в режиме ShareLock блокировку "транзакция 752" в течение 200.108 мс
2023-03-16 06:54:40.731 UTC [277693] postgres@for_locks ПОДРОБНОСТИ:  Process holding the lock: 294211. Wait queue: 277693.
2023-03-16 06:54:40.731 UTC [277693] postgres@for_locks КОНТЕКСТ:  при изменении кортежа (0,9) в отношении "books"
2023-03-16 06:54:40.731 UTC [277693] postgres@for_locks ОПЕРАТОР:  update books set title = 'Английский словарь' where id=2;
2023-03-16 06:55:00.242 UTC [294211] postgres@for_locks СООБЩЕНИЕ:  процесс 294211 продолжает ожидать в режиме ShareLock блокировку "транзакция 753" в течение 200.069 мс
2023-03-16 06:55:00.242 UTC [294211] postgres@for_locks ПОДРОБНОСТИ:  Process holding the lock: 294212. Wait queue: 294211.
2023-03-16 06:55:00.242 UTC [294211] postgres@for_locks КОНТЕКСТ:  при изменении кортежа (0,17) в отношении "books"
2023-03-16 06:55:00.242 UTC [294211] postgres@for_locks ОПЕРАТОР:  update books set title = 'Алгебра' where id = 3;
2023-03-16 06:55:17.977 UTC [294212] postgres@for_locks СООБЩЕНИЕ:  процесс 294212 обнаружил взаимоблокировку, ожидая в режиме ShareLock блокировку "транзакция 751" в течение 200.063 мс
2023-03-16 06:55:17.977 UTC [294212] postgres@for_locks ПОДРОБНОСТИ:  Process holding the lock: 277693. Wait queue: .
2023-03-16 06:55:17.977 UTC [294212] postgres@for_locks КОНТЕКСТ:  при изменении кортежа (0,10) в отношении "books"
2023-03-16 06:55:17.977 UTC [294212] postgres@for_locks ОПЕРАТОР:  update books set title = 'Физика' where id=1;
2023-03-16 06:55:17.978 UTC [294212] postgres@for_locks ОШИБКА:  обнаружена взаимоблокировка
2023-03-16 06:55:17.978 UTC [294212] postgres@for_locks ПОДРОБНОСТИ:  Процесс 294212 ожидает в режиме ShareLock блокировку "транзакция 751"; заблокирован процессом 277693.
        Процесс 277693 ожидает в режиме ShareLock блокировку "транзакция 752"; заблокирован процессом 294211.
        Процесс 294211 ожидает в режиме ShareLock блокировку "транзакция 753"; заблокирован процессом 294212.
        Процесс 294212: update books set title = 'Физика' where id=1;
        Процесс 277693: update books set title = 'Английский словарь' where id=2;
        Процесс 294211: update books set title = 'Алгебра' where id = 3;
2023-03-16 06:55:17.978 UTC [294212] postgres@for_locks ПОДСКАЗКА:  Подробности запроса смотрите в протоколе сервера.
2023-03-16 06:55:17.978 UTC [294212] postgres@for_locks КОНТЕКСТ:  при изменении кортежа (0,10) в отношении "books"
2023-03-16 06:55:17.978 UTC [294212] postgres@for_locks ОПЕРАТОР:  update books set title = 'Физика' where id=1;
2023-03-16 06:55:17.978 UTC [294211] postgres@for_locks СООБЩЕНИЕ:  процесс 294211 получил в режиме ShareLock блокировку "транзакция 753" через 17935.532 мс
2023-03-16 06:55:17.978 UTC [294211] postgres@for_locks КОНТЕКСТ:  при изменении кортежа (0,17) в отношении "books"
2023-03-16 06:55:17.978 UTC [294211] postgres@for_locks ОПЕРАТОР:  update books set title = 'Алгебра' where id = 3;
```

- по журналу можно увидеть последовательность команд и процессов вызвавщих блокировку

# Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
 - Раз вопрос задан и по звездочке предлагается воспроизвести, значит могут. воспроизвести не получилось.
