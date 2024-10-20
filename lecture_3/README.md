# ДЗ
1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
2. Посмотреть размер файла с таблицей
3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
   автовакуум
5. Подождать некоторое время, проверяя, пришел ли автовакуум
6. 5 раз обновить все строчки и добавить к каждой строчке любой символ
7. Посмотреть размер файла с таблицей
8. Отключить Автовакуум на конкретной таблице
9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
10. Посмотреть размер файла с таблицей
11. Объясните полученный результат
12. Не забудьте включить автовакуум)

### Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.

# Выполнение в PostgreSQL 16.4
1. Создаем таблицу с текстовым полем и заполняем случайными данными в размере 1 млн строк
```sql
CREATE TABLE random_text_table (
    id SERIAL PRIMARY KEY,
    random_text TEXT
);

INSERT INTO random_text_table (random_text) SELECT substr(md5(random()::text), 1, 10) FROM generate_series(1, 1000000);
```

2. Посмотрим размер файла с таблицей
```sql
SELECT pg_size_pretty(pg_total_relation_size('random_text_table'));

-- pg_size_pretty
-- ----------------
-- 64 MB
-- (1 row)
```

3. 5 раз обновим все строки и добавим к каждой строке любой символ, используя анонимную процедуру
```sql
DO $$
DECLARE
    i INT := 0;
BEGIN
    WHILE i < 5 LOOP
        i := i + 1;
        UPDATE random_text_table SET random_text = random_text || '@';
    END LOOP;
END $$;
```

4. Посмотрим количество мертвых строк в таблице и когда последний раз приходил автовакуум
```sql
SELECT n_dead_tup, last_autovacuum FROM pg_stat_user_tables WHERE relname = 'random_text_table';

-- n_dead_tup |        last_autovacuum        
-- ------------+-------------------------------
--           0 | 2024-10-20 14:13:42.256006+03
-- (1 row)
```

5. Подождем некоторое время, проверяя, пришел ли автовакуум
```sql
SELECT n_dead_tup, last_autovacuum FROM pg_stat_user_tables WHERE relname = 'random_text_table';

-- n_dead_tup |        last_autovacuum        
-- ------------+-------------------------------
--           0 | 2024-10-20 14:13:42.256006+03
-- (1 row)
```
Без изменений

6. 5 раз обновим все строки и добавим к каждой строке любой символ
```sql
DO $$
DECLARE
    i INT := 0;
BEGIN
    WHILE i < 5 LOOP
        i := i + 1;
        UPDATE random_text_table SET random_text = random_text || '!';
    END LOOP;
END $$;
```

7. Посмотрим размер файла с таблицей
```sql
SELECT pg_size_pretty(pg_total_relation_size('random_text_table'));

-- pg_size_pretty
-- ----------------
-- 392 MB
-- (1 row)
```

8. Отключим автовакуум на конкретной таблице
```sql
ALTER TABLE random_text_table SET (autovacuum_enabled = false);
```

9. 10 раз обновим все строки и добавим к каждой строке любой символ
```sql
DO $$
DECLARE
    i INT := 0;
BEGIN
    WHILE i < 10 LOOP
        i := i + 1;
        UPDATE random_text_table SET random_text = random_text || '?';
    END LOOP;
END $$;
```

10. Посмотрим размер файла с таблицей
```sql
SELECT pg_size_pretty(pg_total_relation_size('random_text_table'));

--  pg_size_pretty 
-- ----------------
--  784 MB
-- (1 row)
```

11. Объясним полученный результат
Выключенный автовакуум не удаляет мертвые строки, что приводит к увеличению размера файла с таблицей.

12. Включим автовакуум на конкретной таблице
```sql
ALTER TABLE random_text_table SET (autovacuum_enabled = true);
```

### Самостоятельная проверка принципа работы VACUUM & VACUUM FULL

13. Сделаем очистку таблицы с помощью VACUUM
```sql
VACUUM random_text_table;
SELECT pg_size_pretty(pg_total_relation_size('random_text_table'));
-- pg_size_pretty
-- ----------------
-- 784 MB
-- (1 row)
```
Дело в том, что VACUUM освобождает место, занимаемое мертвыми строками, но не возвращает это место операционной системе.
Оно остаётся доступным для будущих вставок и обновлений.

14. Сделаем очистку таблицы с помощью VACUUM FULL
```sql
VACUUM random_text_table;
SELECT pg_size_pretty(pg_total_relation_size('random_text_table'));
-- pg_size_pretty
-- ----------------
-- 87 MB
-- (1 row)
```
VACUUM FULL перестраивает таблицу полностью, удаляя все мертвые строки и возвращая освободившееся место обратно операционной системе.
