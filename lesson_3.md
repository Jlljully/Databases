# Домашнее задание к занятию 3. «MySQL»


## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.

### Ответ

![Скрин](https://github.com/Jlljully/Databases/blob/main/files/lesson_3/Screenshot_13.png "1")

```SQL
mysql -u admin -p test_db < /backup/test_dump.sql

mysql> SHOW TABLES;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM orders WHERE price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)


```

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

### Ответ


```SQL
CREATE USER 'test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'test-pass' PASSWORD REUSE INTERVAL 180 DAY;
ALTER USER 'test'@'localhost' WITH MAX_QUERIES_PER_HOUR 100;
ALTER USER 'test'@'localhost' FAILED_LOGIN_ATTEMPTS 3;
ALTER USER 'test'@'localhost' ATTRIBUTE '{"NAME": "James", "SURNAME": "Pretty"}';
GRANT SELECT ON `test_db`.* TO 'test'@'localhost';

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE User='test';
+------+-----------+----------------------------------------+
| USER | HOST      | ATTRIBUTE                              |
+------+-----------+----------------------------------------+
| test | localhost | {"NAME": "James", "SURNAME": "Pretty"} |
+------+-----------+----------------------------------------+
1 row in set (0.00 sec)

```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

### Ответ

```SQL
set profiling=1;
select * from test_db.orders;
show profiles;

mysql> SELECT ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'test_db';
+--------+
| ENGINE |
+--------+
| InnoDB |
+--------+
1 row in set (0.00 sec)

mysql> show profiles;
+----------+------------+------------------------------+
| Query_ID | Duration   | Query                        |
+----------+------------+------------------------------+
|        1 | 0.00062075 | select * from test_db.orders |
+----------+------------+------------------------------+


ALTER TABLE test_db.orders ENGINE=MyISAM;
mysql> SELECT ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'test_db';
+--------+
| ENGINE |
+--------+
| MyISAM |
+--------+
1 row in set (0.00 sec)

select * from test_db.orders;

mysql> show profiles;
+----------+------------+------------------------------------------+
| Query_ID | Duration   | Query                                    |
+----------+------------+------------------------------------------+
|        1 | 0.00062075 | select * from test_db.orders             |
|        2 | 0.01789175 | ALTER TABLE test_db.orders ENGINE=MyISAM |
|        3 | 0.00046125 | select * from test_db.orders             |
+----------+------------+------------------------------------------+
3 rows in set, 1 warning (0.00 sec)


```

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

### Ответ

```SQL
Для выполнения требуемых условий необходимо прописать в конфиг следующие параметры:

innodb_file_per_table=1
innodb_log_buffer_size=1M
innodb_buffer_pool_size=2,4G
innodb_log_file_size=100M
innodb_flush_method = O_DIRECT
tmpdir = /dev/shm
table_cache = 4096
table_definition_cache = 4096
```

