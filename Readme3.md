 создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
 - в HyperV развернул виртуалку VM1: Ubuntu 20.04 Server

поставить на нем Docker Engine
- curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER

сделать каталог /var/lib/postgres
- sudo mkdir /var/lib/postgres

 развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
 - sudo docker network create pg-net
 - sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

 развернуть контейнер с клиентом postgres
 подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
- sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres
- create table test(iid as int);
- insert into test (iid) values (1):
- insert into test (iid) values (2):
- insert into test (iid) values (3):
 
подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
- развернул еще одну виртуалку с ubuntu VM2
- psql -p 5432 -U postgres -h <IP VM1 >  -d postgres -W
- select iid from test; (для проверки наличия данных)

удалить контейнер с сервером
- на VM1 
- sudo docker rm pg-server

 создать его заново
- sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

 подключится снова из контейнера с клиентом к контейнеру с сервером
 - sudo docker stop pg-client
 - sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres
 
 проверить, что данные остались на месте
- select iid from test;
