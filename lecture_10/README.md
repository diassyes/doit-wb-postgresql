# ДЗ
1. Проанализировать данные о зарплатах сотрудников с использованием оконных функций.
   а) На сколько было увеличение с предыдущей зарплатой
   б) если это первая зарплата - вместо NULL вывести 0
   https://www.db-fiddle.com/f/eQ8zNtAFY88i8nB4GRB65V/0

# Выполнение

_*Copy as Markdown on fiddle_

**Schema (PostgreSQL v15)**

    CREATE TYPE employee_gender AS ENUM (
        'M',
        'F'
    );
    
    CREATE TABLE grade (
        id SERIAL PRIMARY KEY,
        value text NOT NULL
    );
    
    insert into grade (value) values ('junior'), ('middle'), ('senoir'), ('lead');
    
    CREATE TABLE employee (
        id SERIAL PRIMARY KEY,
        birth_date date NOT NULL,
        first_name varchar(255) NOT NULL,
        last_name varchar(255) NOT NULL,
        gender employee_gender NOT NULL,
        hire_date date NOT NULL,
        doc_id text NOT NULL default '' -- passport N
    );
    
    insert into employee (birth_date,first_name,last_name,gender,hire_date, doc_id) values 
    ('19800101','Eugene','Aristov','M','20240101','2700123456'),
    ('19790901','Ivan','Ivanov','M','20230101','2400123456'),
    ('19810505','Petr','Petrov','M','20240301','4700123456');
    
    CREATE TABLE salary (
        fk_employee int NOT NULL,
        amount int NOT NULL,
        from_date date NOT NULL,
        to_date date NOT NULL,
        fk_grade int NOT NULL references grade(id),
        CONSTRAINT pk_primary PRIMARY KEY (fk_employee, from_date),
        CONSTRAINT salaries_fk FOREIGN KEY (fk_employee) REFERENCES employee(id) ON UPDATE RESTRICT ON DELETE RESTRICT
    );
    
    insert into salary (fk_employee,amount,from_date,to_date,fk_grade)
    values
    (1,100000,'20240101','20240131',1),
    (1,200000,'20240201','20240229',2),
    (1,300000,'20240301','20991231',3),(2,200000,'20230101','20240131',2),
    (3,200000,'20240301','20240131',2);
    
    CREATE TABLE title_name (
        id SERIAL PRIMARY KEY,
        title varchar(255) NOT NULL,
        amount_from int NOT NULL,
      	amount_to int NOT NULL
    );
    
    insert into title_name(title, amount_from, amount_to) values ('manager',300000,1000000),('teamlead',250000,800000),('python developer',100000,500000),('vice president',300000,5000000);
    
    CREATE TABLE title (
        id SERIAL PRIMARY KEY,
        fk_employee int NOT NULL,
        fk_titlename int NOT NULL references title_name(id),
        from_date date NOT NULL,
        to_date date,
        CONSTRAINT pk_title UNIQUE (fk_employee, fk_titlename, from_date),
        CONSTRAINT titles_fk FOREIGN KEY (fk_employee) REFERENCES employee(id) ON UPDATE RESTRICT ON DELETE CASCADE
    );
    
    insert into title(fk_employee,fk_titlename,from_date)
    values (1,1,'20240101'),(1,4,'20240301'),(2,2,'20230101'),(3,3,'20240301');
    
    create table command(
    	fk_title int references title(id), -- boss
      	fk_title2 int references title(id), -- team
     	CONSTRAINT pk_command PRIMARY KEY (fk_title, fk_title2)
    );
    
    insert into command values (1,2),(1,3);
    
    CREATE OR REPLACE FUNCTION format_number(numeric) RETURNS text AS $$
    BEGIN
        RETURN to_char($1, 'FM999 999 999 999');
    END;
    $$ LANGUAGE plpgsql;


---

**Query #1**

    -- Проанализировать данные о зарплатах сотрудников с использованием оконных функций
    -- 
    
    select e.id emp_id
    , e.first_name || ' ' || e.last_name as employee_name
    , g.value
    , format_number(s.amount) as emp_salary_amount
    , format_number(coalesce(s.amount - lag(s.amount) over salary_window, 0)) as diff
    , s.from_date as grade_from_date
    , s.to_date as grade_to_date
    , tn.title
    , format_number(tn.amount_from) as title_amount_from
    , format_number(tn.amount_to) as title_amount_to
    , t.from_date title_from_date
    , t.from_date title_to_date
    from salary as s
    left join employee as e on e.id = s.fk_employee
    left join grade as g on g.id = s.fk_grade
    left join title as t on t.id = e.id
    left join title_name as tn on tn.id = t.fk_titlename
    window salary_window as (partition by e.id order by e.id, s.from_date)
    order by e.id, s.from_date
    ;

| emp_id | employee_name  | value  | emp_salary_amount | diff      | grade_from_date | grade_to_date | title          | title_amount_from | title_amount_to | title_from_date | title_to_date |
| ------ | -------------- | ------ | ----------------- | --------- | --------------- | ------------- | -------------- | ----------------- | --------------- | --------------- | ------------- |
| 1      | Eugene Aristov | junior |   100 000         |    0      | 2024-01-01      | 2024-01-31    | manager        |   300 000         |  1 000 000      | 2024-01-01      | 2024-01-01    |
| 1      | Eugene Aristov | middle |   200 000         |   100 000 | 2024-02-01      | 2024-02-29    | manager        |   300 000         |  1 000 000      | 2024-01-01      | 2024-01-01    |
| 1      | Eugene Aristov | senoir |   300 000         |   100 000 | 2024-03-01      | 2099-12-31    | manager        |   300 000         |  1 000 000      | 2024-01-01      | 2024-01-01    |
| 2      | Ivan Ivanov    | middle |   200 000         |    0      | 2023-01-01      | 2024-01-31    | vice president |   300 000         |  5 000 000      | 2024-03-01      | 2024-03-01    |
| 3      | Petr Petrov    | middle |   200 000         |    0      | 2024-03-01      | 2024-01-31    | teamlead       |   250 000         |   800 000       | 2023-01-01      | 2023-01-01    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/eQ8zNtAFY88i8nB4GRB65V/0)