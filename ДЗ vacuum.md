#### Домашнее задание MVCC, vacuum и autovacuum.

##### 1) Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
* Вывод:  
>>>scaling factor: 1  
query mode: simple  
number of clients: 8  
number of threads: 1  
maximum number of tries: 1  
duration: 60 s  
number of transactions actually processed: 202136  
number of failed transactions: 0 (0.000%)  
latency average = 2.372 ms  
latency stddev = 1.448 ms  
initial connection time = 43.229 ms  
tps = 3370.897387 (without initial connection time)

____________________________________________________
##### 2) Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла, протестировать заново  
max_connections = 40  
shared_buffers = 1GB  
effective_cache_size = 3GB  
maintenance_work_mem = 512MB  
checkpoint_completion_target = 0.9  
wal_buffers = 16MB  
default_statistics_target = 500  
random_page_cost = 4  
effective_io_concurrency = 2  
work_mem = 6553kB  
min_wal_size = 4GB  
max_wal_size = 16GB
* Вывод:  
>>>duration: 60 s
number of transactions actually processed: 202120
number of failed transactions: 0 (0.000%)
latency average = 2.373 ms
latency stddev = 1.526 ms
initial connection time = 44.129 ms
tps = 3370.518542 (without initial connection time)
* Результат практически не изменился, пробовал 3 раза

*Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
CREATE TABLE test1 (
  id SERIAL,
  name CHAR(100)
);
INSERT INTO test1(name) SELECT 'noname 'FROM generate_series(1,1000000); - добавить строки
*Посмотреть размер файла с таблицей
SELECT pg_size_pretty(pg_total_relation_size('test1'));
--Вывод:
 pg_size_pretty

 135 MB
(1 row)

*5 раз обновить все строчки и добавить к каждой строчке любой символ
update test1 set name = '123'; - обновить 5 раз
UPDATE 1000000х5

*Посмотреть количество мертвых строчек в таблице
SELECT relname, n_dead_tup
FROM pg_stat_user_tables
WHERE relname = 'test1';

*Когда последний раз приходил автовакуум
SELECT * FROM pg_stat_all_tables WHERE relname = 'test1';

*5 раз обновить все строчки и добавить к каждой строчке любой символ
Повторяем -update test1 set name = '123';

*Посмотреть размер файла с таблицей
SELECT pg_size_pretty(pg_total_relation_size('test1'));
--Вывод:
 pg_size_pretty

 806 MB
(1 row)

*Отключить Автовакуум на конкретной таблице
ALTER TABLE test1 SET (autovacuum_enabled = OFF);

*10 раз обновить все строчки и добавить к каждой строчке любой симво
Повторяем

**Посмотреть размер файла с таблицей
--Вывод:
 pg_size_pretty
 1751 MB
 
 *Объясните полученный результат
 При выключеннном автовакуум - данные в таблице не очищаются из за этого она сильно выросла
 
 *Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице:
CREATE OR REPLACE PROCEDURE update_table_in_loop(
    IN test1 VARCHAR,
    IN column_name VARCHAR,
)
AS $$
DECLARE
    i INTEGER := 1;
BEGIN
    LOOP
	UPDATE test1
        SET column_name = new_value
        WHERE id = i;


        IF i = 10 THEN
            EXIT;
        END IF;


        i := i + 1;
    END LOOP;
END;
$$ LANGUAGE plpgsql;