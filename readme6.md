попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
напишите получилось или нет и почему
Не получилось, потому что перемещен каталог с данными и не обнаружив его выдает ошибку
```
Error: /var/lib/postgresql/14/main is not accessible or does not exist
```
задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его
напишите что и почему поменяли
- в файле _postgresql.conf_ изменил параметр _data_directory_ на новое расположение _'/mnt/data/main'_
postgres запустился, потому что в настройках указано верное расположение каталога с данными
