#### Домашнее задание Выборка данных, виды join'ов. Применение и оптимизация.   

* 1) Реализовать прямое соединение двух или более таблиц  
Создаем 2 таблицы с даными:
 > CREATE TABLE table1 (  
    id SERIAL PRIMARY KEY,  
    data1 VARCHAR(50)  
);  
> CREATE TABLE table2 (  
    id SERIAL PRIMARY KEY,  
    table1_id INT REFERENCES table1(id),  
    data2 VARCHAR(50)  
);  

* Прямое соединение таблиц по полю table1_id::  
> SELECT   
    t1.id, t1.data1,  
    t2.id, t2.data2  
FROM   
    table1 t1  
JOIN   
    table2 t2 ON t1.id = t2.table1_id;  

* 2) Реализовать левостороннее (или правостороннее) соединение двух или более таблиц    
   Получаем все записи из table1 и соответствующие — из table2  
   Левостороннее:  
> SELECT   
    t1.id, t1.data1,  
    t2.id AS t2_id, t2.data2  
FROM   
    table1 t1  
LEFT JOIN   
    table2 t2 ON t1.id = t2.table1_id;    
	Правосторонее:  
	Получаем все записи из table2 и соответствующие — из table1  
> SELECT   
    t1.id, t1.data1,  
    t2.id AS t2_id, t2.data2  
FROM   
    table1 t1  
RIGHT JOIN   
    table2 t2 ON t1.id = t2.table1_id;  	

* 3) Реализовать кросс соединение двух или более таблиц    
> SELECT   
    table1.*,   
    table2.*  
FROM   
    table1  
CROSS JOIN   
    table2;  
	(Каждая строка из table1 сочетается со всеми строками из table2)  

* 4) Реализовать полное соединение двух или более таблиц  
> SELECT  
    t1.*,  
    t2.*   
FROM  
    table1 t1  
FULL OUTER JOIN  
    table2 t2  
ON  
    t1.id = t2.id;  
	(возвращает все строки из обеих таблиц, заполняя NULL там, где соответствующих записей нет.)  
	
* 5) Реализовать запрос, в котором будут использованы разные типы соединений    
 Создать таблиц и заполнение данных:  
 >
CREATE TABLE table1 (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE table2 (
    id INT PRIMARY KEY,
    description VARCHAR(100)
);

INSERT INTO table1 (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie');

INSERT INTO table2 (id, description) VALUES
(2, 'Engineer'),
(3, 'Doctor'),
(4, 'Artist');

 * Запросы:

* a. только совпадающие по id
>SELECT   
    'INNER JOIN' AS join_type,  
    t1.id AS id,  
    t1.name,  
    t2.description  
FROM   
    table1 t1  
INNER JOIN   
    table2 t2 ON t1.id = t2.id  

UNION ALL  

* b. все из table1 + соответствующие из table2 (или NULL)  
>SELECT 
    'LEFT JOIN' AS join_type,  
    t1.id,  
    t1.name,  
    t2.description  
FROM   
    table1 t1  
LEFT JOIN   
    table2 t2 ON t1.id = t2.id  
WHERE   
    t2.id IS NULL  -- только те из table1 без соответствия в table2  

UNION ALL  

* c.все из table2 + соответствующие из table1 (или NULL)    
>SELECT   
    'RIGHT JOIN' AS join_type,  
    t1.id,  
    t1.name,  
    t2.description  
FROM   
    table1 t1  
RIGHT JOIN   
    table2 t2 ON t1.id = t2.id  
WHERE   
    t1.id IS NULL  -- только те из table2 без соответствия в table1  

UNION ALL  

* d.все из обеих таблиц  
>SELECT       
    'FULL OUTER JOIN' AS join_type,  
    COALESCE(t1.id, t2.id) AS id,  
    t1.name,  
    t2.description  
FROM   
    table1 t1  
FULL OUTER JOIN   
    table2 t2 ON t1.id = t2.id;  
 
 -- join_type - тип соединения для каждой строки  
 -- Для строк без совпадений будет NULL