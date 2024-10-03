# ДЗ
1. Развернуть ВМ (Linux) с PostgreSQL
2. Залить Тайские перевозки https://github.com/aeuge/postgres16book/tree/main/database
3. Посчитать количество поездок - select count(*) from book.tickets;

# Выполнение
## Установка PostgreSQL
### Remote VM
Сначала пытался установить PostgreSQL на удаленный VM в ВБ Клауд из официальных исходников,
но столкнулся с проблемой при попытке апдейтнуть добавленный репо в apt-get.

\* _К следующему домашнему заданию постараюсь сделать на remote VM_

### Local Docker
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
postgres=# \l

   Name    |  Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+------------+------------+--------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 template0 | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
           |          |          |                 |            |            |        |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
           |          |          |                 |            |            |        |           | postgres=CTc/postgres
 thai      | postgres | UTF8     | libc            | C.UTF-8    | C.UTF-8    |        |           | 
(4 rows)

```
Подключился к БД
```psql
postgres=# \c thai
```
Посмотрел схемы
```psql
thai=# \dn

      List of schemas
  Name  |       Owner       
--------+-------------------
 book   | postgres
 public | pg_database_owner
(2 rows)


```

Посмотрел таблицу
```psql
thai=# \dt book.*

            List of relations
 Schema |     Name     | Type  |  Owner   
--------+--------------+-------+----------
 book   | bus          | table | postgres
 book   | busroute     | table | postgres
 book   | busstation   | table | postgres
 book   | fam          | table | postgres
 book   | nam          | table | postgres
 book   | ride         | table | postgres
 book   | schedule     | table | postgres
 book   | seat         | table | postgres
 book   | seatcategory | table | postgres
 book   | tickets      | table | postgres
(10 rows)
```

Посчитать количество поездок - select count(*) from book.tickets; 
```psql
select count(*) from book.tickets;

  count  
---------
 5185505
(1 row)
```