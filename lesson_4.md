## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
- подключения к БД,
- вывода списка таблиц,
- вывода описания содержимого таблиц,
- выхода из psql.

```
\l
\c
\dp
\dt+
\q
```

## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

```SQL
-- psql -U admin -d test_database -f /backup/test_dump.sql;

-- psql -U admin
-- \connect test_database


SELECT MAX(avg_width) FROM pg_stats WHERE tablename='orders';
 max
-----
  16
(1 row)

-- Смотрим все значения для проверки:
SELECT avg_width,attname FROM pg_stats WHERE tablename='orders';
 avg_width | attname
-----------+---------
         4 | id
        16 | title
         4 | price
(3 rows)
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?


```SQL
test_database=# CREATE TABLE orders1 (CHECK ( price > 499)) INHERITS (orders);
CREATE TABLE
test_database=# CREATE TABLE orders2 (CHECK ( price <= 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders1 SELECT * FROM orders WHERE price > 499;
INSERT 0 3
test_database=# INSERT INTO orders2 SELECT * FROM orders WHERE price <= 499;
INSERT 0 5
select * from orders1;
 id |       title        | price
----+--------------------+-------
  2 | My little database |   500
  6 | WAL never lies     |   900
  8 | Dbiezdmin          |   501
(3 rows)

 select * from orders2;
 id |        title         | price
----+----------------------+-------
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(5 rows)

-- Но в результате у нас значения дублируются с базовой таблицей:
select * from orders;
 id |        title         | price
----+----------------------+-------
  1 | War and peace        |   100
  2 | My little database   |   500
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  6 | WAL never lies       |   900
  7 | Me and my bash-pet   |   499
  8 | Dbiezdmin            |   501
  2 | My little database   |   500
  6 | WAL never lies       |   900
  8 | Dbiezdmin            |   501
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(16 rows)

-- Выполнив удаление данных только из orders мы получим желаемую консистентность:
DELETE FROM ONLY orders WHERE price > 499;
DELETE  FROM ONLY orders WHERE price <= 499;

select * from orders;
 id |        title         | price
----+----------------------+-------
  2 | My little database   |   500
  6 | WAL never lies       |   900
  8 | Dbiezdmin            |   501
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(8 rows)

При изначальном проектировании можно учесть распределение данных, но необходимо внимательно изучить условия распределения данных.
```


## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

```
pg_dump -U admin test_database > /backup/test_dump.bak

Для добавления уникальности необходимо в секции создания БД добавить признак unique для столбца `title`.

CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL,
    price integer DEFAULT 0
);

```
