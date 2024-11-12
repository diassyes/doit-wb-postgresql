# ДЗ
1. Создать таблицу с продажами.
2. Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд)
   а. через case
   b. * (бонуса в виде зачета дз не будет) используя математическую операцию (лучше 2+ варианта)
   с. предусмотреть NULL на входе
3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)
4. Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё отработало

# Выполнение в PostgreSQL 16.4
1. Создаем таблицу с продажами
```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
);
-- Наполним данными в количестве 1 000 000 записей
INSERT INTO sales (sale_date, amount)
SELECT
    '2024-01-01'::DATE + (random() * 364)::INTEGER,
    (random() * 1000)::DECIMAL(10, 2)
FROM generate_series(1, 1000000);
```

2. (A) Реализуем функцию выбора трети года через case
```sql
CREATE OR REPLACE FUNCTION get_sales_third_of_year_case(sale_date DATE)
RETURNS INTEGER AS $$
BEGIN
    RETURN CASE
        WHEN EXTRACT(MONTH FROM sale_date) IN (1, 2, 3, 4) THEN 1
        WHEN EXTRACT(MONTH FROM sale_date) IN (5, 6, 7, 8) THEN 2
        WHEN EXTRACT(MONTH FROM sale_date) IN (9, 10, 11, 12) THEN 3
        ELSE NULL
    END;
END;
$$ LANGUAGE plpgsql;
```

2. (B) Реализуем функцию выбора трети года через математическую операцию
```sql
CREATE OR REPLACE FUNCTION get_sales_third_of_year_math(sale_date DATE)
RETURNS INTEGER AS $$
BEGIN
    RETURN CEIL((EXTRACT(MONTH FROM sale_date) - 0.01) / 4);
END;
$$ LANGUAGE plpgsql;
```

3. Проверим скорость выполнения сложного запроса
```
postgres=# \timing
``` 
```sql
SELECT get_sales_third_of_year_case(sale_date) AS third_of_year_case
FROM sales;
-- Time: 835.490 ms
```
```sql
SELECT get_sales_third_of_year_math(sale_date) AS third_of_year_math
FROM sales;
-- Time: 959.260 ms
```

4. Вызовем функцию в SELECT из таблицы с продажами
```sql
SELECT
    sale_date,
    amount,
    get_sales_third_of_year_case(sale_date) AS third_of_year_case,
    get_sales_third_of_year_math(sale_date) AS third_of_year_math
FROM sales
LIMIT 10
```

5. Убеждаемся, что результат обеих функции корректный на всех записях
```sql
SELECT count(*)
FROM sales
WHERE get_sales_third_of_year_case(sale_date) != get_sales_third_of_year_math(sale_date);
```
```
 count 
-------
     0
(1 row)
```
