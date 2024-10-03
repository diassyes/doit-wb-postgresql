# Установка PostgreSQL
## Remote VM
Сначала пытался установить PostgreSQL на удаленный VM в ВБ Клауд из официальных исходников,
но столкнулся с проблемой при попытке апдейтнуть добавленный репо в apt-get.
## Local Docker
В виду того, что оставались полчаса до начала следующей лекции решил установить PostgreSQL на локальной машине через Docker.
Запустил контейнер в демоне и подключился к нему через psql.
```bash
docker exec -it some-postgres bash
```
Закачал данные
```bash
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
```
Подключился к базе
```bash
psql -U postgres
```
Посмотрел БД
```psql
\l
```
Подключился к БД
```psql
\c thai
```
Посмотрел схемы
```psql
\dn
```

Посмотрел таблицу
```psql
\dt book.*
```
### Задание:

Посчитать количество поездок - select count(*) from book.tickets; 
```psql
select count(*) from book.tickets;
```

```psql
  count  
---------
 5185505
(1 row)
```