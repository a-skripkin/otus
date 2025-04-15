# otus
postgres
скрипиы
-- Создать таблицу
DROP TABLE IF EXISTS persons;
CREATE TABLE persons (
    id SERIAL PRIMARY KEY,
    first_name TEXT,
    second_name TEXT
);

INSERT INTO persons (first_name, second_name) VALUES
('Ivan', 'Ivanov'),
('Petr', 'Petrov'),
('Anna', 'Smirnova');


---------сессия 1
-- Неповторяющееся чтение
----READ COMMITTED
BEGIN;
SELECT second_name FROM persons WHERE first_name = 'Petr';
-- сессия 2
SELECT second_name FROM persons WHERE first_name = 'Petr';
COMMIT;
----REPEATABLE READ
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT second_name FROM persons WHERE first_name = 'Anna';
-- сессия 2
SELECT second_name FROM persons WHERE first_name = 'Anna';
COMMIT;
----SERIALIZABLE
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT second_name FROM persons WHERE first_name = 'Ivan';
-- сессия 2
SELECT second_name FROM persons WHERE first_name = 'Ivan';
COMMIT;

-- Фантомное чтение
----READ COMMITTED
BEGIN;
SELECT * FROM persons WHERE first_name LIKE 'Z%';
-- сессия 2
SELECT * FROM persons WHERE first_name LIKE 'Z%';
COMMIT;
----REPEATABLE READ
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM persons WHERE first_name LIKE 'Z%';
-- сессия 2
SELECT * FROM persons WHERE first_name LIKE 'Z%';
COMMIT;
----SERIALIZABLE
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM persons WHERE first_name LIKE 'Z%';
-- сессия 2
SELECT * FROM persons WHERE first_name LIKE 'Z%';
COMMIT;

-- Аномалия сериализации
----READ COMMITTED
BEGIN;
SELECT 1 FROM persons WHERE first_name = 'Olga' AND second_name = 'Petrova';
-- ничего не найдено → считаем, что можно вставить
INSERT INTO persons(first_name, second_name) VALUES ('Olga', 'Petrova');
-- сессия 2
COMMIT;
----REPEATABLE READ
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT 1 FROM persons WHERE first_name = 'Galina' AND second_name = 'Okuneva';
INSERT INTO persons(first_name, second_name) VALUES ('Galina', 'Okuneva');
-- сессия 2
COMMIT;
----SERIALIZABLE
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT 1 FROM persons WHERE first_name = 'Maxim' AND second_name = 'Maximov';
INSERT INTO persons(first_name, second_name) VALUES ('Maxim', 'Maximov');
-- сессия 2
COMMIT;


---------сессия 2
-- Неповторяющееся чтение
----READ COMMITTED
UPDATE persons SET second_name = 'Updated' WHERE first_name = 'Petr';
----REPEATABLE READ
UPDATE persons SET second_name = 'Changed' WHERE first_name = 'Anna';
----SERIALIZABLE
UPDATE persons SET second_name = 'Modified' WHERE first_name = 'Ivan';

-- Фантомное чтение
----READ COMMITTED
INSERT INTO persons(first_name, second_name) VALUES ('Zina', 'Zinina');
----REPEATABLE READ
INSERT INTO persons(first_name, second_name) VALUES ('Zoya', 'Zorina');
----SERIALIZABLE
INSERT INTO persons(first_name, second_name) VALUES ('Zara', 'Zakharova');

-- Аномалия сериализации
----READ COMMITTED
BEGIN;
SELECT 1 FROM persons WHERE first_name = 'Olga' AND second_name = 'Petrova';
INSERT INTO persons(first_name, second_name) VALUES ('Olga', 'Petrova');
COMMIT;
----REPEATABLE READ
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT 1 FROM persons WHERE first_name = 'Galina' AND second_name = 'Okuneva';
INSERT INTO persons(first_name, second_name) VALUES ('Galina', 'Okuneva');
COMMIT;
----SERIALIZABLE
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT 1 FROM persons WHERE first_name = 'Maxim' AND second_name = 'Maximov';
INSERT INTO persons(first_name, second_name) VALUES ('Maxim', 'Maximov');
COMMIT;
