# ДЗ
1. открыть консоль и зайти по ssh на ВМ
2. открыть вторую консоль и также зайти по ssh на ту же ВМ (можно в докере 2 сеанса)
3. запустить везде psql из под пользователя postgres
4. сделать в первой сессии новую таблицу и наполнить ее данными
5. посмотреть текущий уровень изоляции:
6. начать новую транзакцию в обеих сессиях с дефолтным (не меняя) уровнем
   изоляции
7. в первой сессии добавить новую запись
8. сделать запрос на выбор всех записей во второй сессии
9. видите ли вы новую запись и если да то почему? После задания можете сверить
   правильный ответ с эталонным (будет доступен после 3 лекции)
10. завершить транзакцию в первом окне
11. сделать запрос на выбор всех записей второй сессии
12. видите ли вы новую запись и если да то почему?
13. завершите транзакцию во второй сессии
14. начать новые транзакции, но уже на уровне repeatable read в ОБЕИХ сессиях
15. в первой сессии добавить новую запись
16. сделать запрос на выбор всех записей во второй сессии
17. видите ли вы новую запись и если да то почему?
18. завершить транзакцию в первом окне
19. сделать запрос во выбор всех записей второй сессии
20. видите ли вы новую запись и если да то почему?

# Выполнение в PostgreSQL 16.4
1. Открываем консоль и подключаемся к ВМ
```bash
# 1 session
tsh ssh ...@...
```
2. Открываем вторую консоль и подключаемся к ВМ
```bash
# 2 session
tsh ssh ...@...
```
3. Запускаем psql из под пользователя postgres
```bash
# 1 session
psql -U postgres
```
```bash
# 2 session
psql -U postgres
```
4. Создаем таблицу и наполняем ее данными
```sql
-- 1 session
begin;
CREATE TABLE wallet (id SERIAL PRIMARY KEY, owner VARCHAR(255), amount INT);
INSERT INTO wallet (owner, amount) VALUES ('Alice', 100), ('Bob', 200), ('Charlie', 300);
commit;
```
5. Проверяем текущий уровень изоляции
```sql
-- 1 session
SHOW default_transaction_isolation;
-- Output: read committed
```
6. Начинаем новую транзакцию в обеих сессиях с дефолтным уровнем изоляции
```sql
-- 1 session
begin;
```
```sql
-- 2 session
begin;
```
7. Добавляем новую запись в первой сессии
```sql
-- 1 session
INSERT INTO wallet (owner, amount) VALUES ('David', 400);
```
8. Делаем запрос на выбор всех записей во второй сессии
```sql
-- 2 session
SELECT * FROM wallet;
```
9. Не видим новую запись, потому что транзакция в первой сессии не завершена и данные не зафиксированы при уровне изоляции read committed
```text
id | owner   | amount
----+---------+-------
 1 | Alice   |   100
 2 | Bob     |   200
 3 | Charlie |   300
(3 rows)
```
10. Завершаем транзакцию в первом окне
```sql
-- 1 session
commit;
```
11. Делаем запрос на выбор всех записей во второй сессии
```sql
-- 2 session
SELECT * FROM wallet;
```
12. Видим новую запись, потому что транзакция в первой сессии завершена и данные зафиксированы, а вторая транзакция еще в процессе при уровне изоляции read committed.
Мы столкнулись с проблемой **фантомного чтения**. Read committed говорит за себя, что данные зафиксированы и видны другим транзакциям.
```text
 id |  owner  | amount 
----+---------+--------
  1 | Alice   |    100
  2 | Bob     |    200
  3 | Charlie |    300
  4 | David   |    400
(4 rows)
```
13. Завершаем транзакцию во второй сессии
```sql
-- 2 session
commit;
```
14. Начинаем новые транзакции на уровне repeatable read в обеих сессиях
```sql
-- 1 session
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```
```sql
-- 2 session
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```
15. Добавляем новую запись в первой сессии
```sql
-- 1 session
INSERT INTO wallet (owner, amount) VALUES ('Eve', 500);
```
16. Делаем запрос на выбор всех записей во второй сессии
```sql
-- 2 session
SELECT * FROM wallet;
```
17. Не видим новую запись, потому что транзакция в первой сессии не завершена и данные не зафиксированы при уровне изоляции repeatable read.
```text
 id |  owner  | amount 
----+---------+--------
  1 | Alice   |    100
  2 | Bob     |    200
  3 | Charlie |    300
  4 | David   |    400
(4 rows)
```
18. Завершаем транзакцию в первом окне
```sql
-- 1 session
commit;
```
19. Делаем запрос на выбор всех записей во второй сессии
```sql
-- 2 session
SELECT * FROM wallet;
```
20. Не видим новую запись, потому что хоть транзакция в первой сессии завершена и данные зафиксированы,
вторая транзакция на уровне repeatable read не видит новую запись, потому что в PostgreSQL фантомное чтение при уровне изоляции repeatable read не допускается.
```text
 id |  owner  | amount 
----+---------+--------
  1 | Alice   |    100
  2 | Bob     |    200
  3 | Charlie |    300
  4 | David   |    400
(4 rows)
```
