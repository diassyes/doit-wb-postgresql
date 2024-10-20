# ДЗ
1. Создать таблицу accounts(id integer, amount numeric);
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

# Выполнение в PostgreSQL 16.4
1. Создаем таблицу accounts(id integer, amount numeric);
```sql
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    amount NUMERIC
);
```

2. Добавляем несколько записей
```sql
INSERT INTO accounts (amount) VALUES (100), (200), (300);
```

3. Подключаемся к БД в 2 терминалах
```bash
# 1 session
psql -U postgres

# 2 session
psql -U postgres
```

4. В первом терминале начинаем транзакцию и блокируем запись
```sql
-- 1 session
BEGIN;
UPDATE accounts SET amount = amount + 100 WHERE id = 1;
```

5. Во втором терминале начинаем транзакцию и блокируем запись
```sql
-- 2 session
BEGIN;
UPDATE accounts SET amount = amount + 100 WHERE id = 2;
```

6. В первом терминале пытаемся обновить запись, которую заблокировал второй терминал
```sql
-- 1 session
UPDATE accounts SET amount = amount + 100 WHERE id = 2;
```

7. Во втором терминале пытаемся обновить запись, которую заблокировал первый терминал
```sql
-- 2 session
UPDATE accounts SET amount = amount + 100 WHERE id = 1;
```

8. Во втором терминале видим ошибку
```text
ERROR:  deadlock detected
DETAIL:  Process 179027 waits for ShareLock on transaction 770; blocked by process 178719.
Process 178719 waits for ShareLock on transaction 771; blocked by process 179027.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "accounts"
```

9. Завершаем транзакции
```sql
-- 1 session
ROLLBACK;

-- 2 session
ROLLBACK;
```

10. Посмотрим логи
```bash
tail -f /var/log/postgresql/postgresql-16-main.log
```
```text
2024-10-20 15:35:54.357 MSK [179027] postgres@postgres ERROR:  deadlock detected
2024-10-20 15:35:54.357 MSK [179027] postgres@postgres DETAIL:  Process 179027 waits for ShareLock on transaction 770; blocked by process 178719.
	Process 178719 waits for ShareLock on transaction 771; blocked by process 179027.
	Process 179027: UPDATE accounts SET amount = amount + 100 WHERE id = 1;
	Process 178719: UPDATE accounts SET amount = amount + 100 WHERE id = 2;
2024-10-20 15:35:54.357 MSK [179027] postgres@postgres HINT:  See server log for query details.
2024-10-20 15:35:54.357 MSK [179027] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "accounts"
2024-10-20 15:35:54.357 MSK [179027] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount + 100 WHERE id = 1;
2024-10-20 15:36:31.531 MSK [68202] LOG:  checkpoint starting: time
2024-10-20 15:36:31.637 MSK [68202] LOG:  checkpoint complete: wrote 2 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.102 s, sync=0.002 s, total=0.107 s; sync files=2, longest=0.001 s, average=0.001 s; distance=0 kB, estimate=399533 kB; lsn=1/76996E20, redo lsn=1/76996DE8
```
