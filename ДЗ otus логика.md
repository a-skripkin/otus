#### ДЗ Логический уровень

3) создайте новую базу данных testdb   
```CREATE DATABASE testdb```
4) зайдите в созданную базу данных под пользователем postgres  
```psql -U postgres``` -  если получаем ошибку с авторизацией, меняем  в файле pg_hba.conf peer на md5  
5) создайте новую схему testnm  
```CREATE SCHEMA testnm;```
6) создайте новую таблицу t1 с одной колонкой c1 типа integer  
```CREATE TABLE t1(c1 integer);```
7) вставьте строку со значением c1=1  
```INSERT INTO t1 values(1);```
8) создайте новую роль readonly  
```CREATE role readonly;```
9) дайте новой роли право на подключение к базе данных testdb  
```grant connect on DATABASE testdb TO readonly;```
10) дайте новой роли право на использование схемы testnm  
```grant usage on SCHEMA testnm to readonly;```
11) дайте новой роли право на select для всех таблиц схемы testnm  
```grant SELECT on all TABLEs in SCHEMA testnm TO readonly;```
12) создайте пользователя testread с паролем test123  
```CREATE USER testread with password 'test123';```
13) дайте роль readonly пользователю testread  
```grant readonly TO testread;```
14) зайдите под пользователем testread в базу данных testdb  
Получил ошибку - "Peer authentication failed for user "testread" - прописываем  в файле pg_hba.conf параметр md5 для testread, для возможности подключения с паролем.
15) сделайте select * from t1;  
Ошибка - "permission denied for table t1" потому что таблица t1 создана в схеме public, на нее права мы не давали
16)вернитесь в базу данных testdb под пользователем postgres  
```\c testdb postgres```
17)удалите таблицу t1  
```DROP TABLE t1;```
18)создайте ее заново но уже с явным указанием имени схемы testnm  
```CREATE TABLE testnm.t1(c1 integer);```
19)вставьте строку со значением c1=1  
```INSERT INTO testnm.t1 values(1);```
20)зайдите под пользователем testread в базу данных testdb  
```\c testdb testread;```
21)сделайте select * from testnm.t1;  
Ошибка - "ERROR:  relation "t1" does not exist" - потому что доступ для testread есть только для ранее созданных таблиц
Опять подключаемся пользователем postgres - ```\c testdb postgres;```  
В рабочих базах  рекомендуется отключить роль public  
```REVOKE CREATE on SCHEMA public FROM public;```  
```REVOKE ALL on DATABASE testdb FROM public;```  
22)Повторно назначаем права для роли readonly  
```grant SELECT on all TABLEs in SCHEMA testnm TO readonly;```
23)Перезаходим и  и выполняем запрос  
```\c testdb testread ```
```select * from t1;```  - успешно выполнен!
24)теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);  
Ошибка - "permission denied for schema public" - нет прав для работы в таблице public  
Перезахожу под postres и азначаю права:  
```\c testdb postgres ```
```grant All on schema public to readonly;```
25)Перезахожу под testread и создаю таблицу    
```create table t2(c1 integer); insert into t2 values (2);``` - Успех !
26)теперь попробуйте выполнить команду ```create table t3(c1 integer); insert into t2 values (2);```  
Успешно выполнено, т.к. на предыдущем шаге мы назначили права для работы в схеме public для роли readonly
