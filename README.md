# Домашнее задание к занятию 3. «MySQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

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

#### Ответ на задание 1.

- Выдача статуса БД из консоли mysql:
```SQL
# команда status или \s
Server version:		8.0.36 MySQL Community Server - GPL
```

- Запуск контейнера через docker-compose с выносом ключевых файлов в родительскую файловую систему.
Запрос к восстановленной БД:
```SQL
MySQL [test_db]> select count(*) from orders where price>300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.001 sec)
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

#### Ответ на задание 2.

- Ниже приведены команды создания и их вывод:
```SQL
MySQL [test_db]> create user 'test'@'localhost' identified by 'test-pass' WITH MAX_QUERIES_PER_HOUR 100 password expire interval 180 day failed_login_attempts 3 PASSWORD_LOCK_TIME 1 ATTRIBUTE '{"fname": "James", "lname": "Pretty"}';
Query OK, 0 rows affected (0.020 sec)

MySQL [test_db]> 
MySQL [test_db]> 
MySQL [test_db]> 
MySQL [test_db]> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user='test'
    -> ;
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "James", "lname": "Pretty"} |
+------+-----------+---------------------------------------+
1 row in set (0.002 sec)

MySQL [test_db]> GRANT SELECT ON test_db.* TO 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.010 sec)
```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

#### Ответ на задание 3.

<details>

<summary>Простыня команд:</summary>

```SQL
MySQL [test_db]> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.001 sec)

MySQL [test_db]> select engine, table_schema from information_schema.tables where table_schema='test_db';
+--------+--------------+
| ENGINE | TABLE_SCHEMA |
+--------+--------------+
| InnoDB | test_db      |
+--------+--------------+
1 row in set (0.002 sec)

MySQL [test_db]> show tables
    -> ;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.002 sec)

MySQL [test_db]> alter table orders ENGINE='MyISAM';
Query OK, 5 rows affected (0.053 sec)
Records: 5  Duplicates: 0  Warnings: 0

MySQL [test_db]> select engine, table_schema from information_schema.tables where table_schema='test_db';
+--------+--------------+
| ENGINE | TABLE_SCHEMA |
+--------+--------------+
| MyISAM | test_db      |
+--------+--------------+
1 row in set (0.002 sec)

MySQL [test_db]> alter table orders ENGINE='InnoDB';
Query OK, 5 rows affected (0.064 sec)
Records: 5  Duplicates: 0  Warnings: 0

MySQL [test_db]> select engine, table_schema from information_schema.tables where table_schema='test_db';
+--------+--------------+
| ENGINE | TABLE_SCHEMA |
+--------+--------------+
| InnoDB | test_db      |
+--------+--------------+
1 row in set (0.002 sec)

MySQL [test_db]> show profiles;
+----------+------------+-----------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                   |
+----------+------------+-----------------------------------------------------------------------------------------+
|        1 | 0.00141400 | select engine, table_schema from information_schema.tables where table_schema='test_db' |
|        2 | 0.00193125 | show tables                                                                             |
|        3 | 0.05259250 | alter table orders ENGINE='MyISAM'                                                      |
|        4 | 0.00194975 | select engine, table_schema from information_schema.tables where table_schema='test_db' |
|        5 | 0.06547300 | alter table orders ENGINE='InnoDB'                                                      |
|        6 | 0.00156425 | select engine, table_schema from information_schema.tables where table_schema='test_db' |
+----------+------------+-----------------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.001 sec)

MySQL [test_db]> show profile for query 3;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000069 |
| Executing hook on transaction  | 0.000005 |
| starting                       | 0.000091 |
| checking permissions           | 0.000008 |
| checking permissions           | 0.000004 |
| init                           | 0.000010 |
| Opening tables                 | 0.000289 |
| setup                          | 0.000083 |
| creating table                 | 0.000892 |
| waiting for handler commit     | 0.000016 |
| waiting for handler commit     | 0.003855 |
| After create                   | 0.000381 |
| System lock                    | 0.000011 |
| copy to tmp table              | 0.000082 |
| waiting for handler commit     | 0.000008 |
| waiting for handler commit     | 0.000009 |
| waiting for handler commit     | 0.000024 |
| rename result table            | 0.000054 |
| waiting for handler commit     | 0.021868 |
| waiting for handler commit     | 0.000018 |
| waiting for handler commit     | 0.002989 |
| waiting for handler commit     | 0.000013 |
| waiting for handler commit     | 0.007461 |
| waiting for handler commit     | 0.000020 |
| waiting for handler commit     | 0.001101 |
| end                            | 0.009592 |
| query end                      | 0.003095 |
| closing tables                 | 0.000019 |
| waiting for handler commit     | 0.000023 |
| freeing items                  | 0.000478 |
| cleaning up                    | 0.000028 |
+--------------------------------+----------+
31 rows in set, 1 warning (0.001 sec)

MySQL [test_db]> show profile for query 5;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000066 |
| Executing hook on transaction  | 0.000094 |
| starting                       | 0.000022 |
| checking permissions           | 0.000006 |
| checking permissions           | 0.000005 |
| init                           | 0.000011 |
| Opening tables                 | 0.000214 |
| setup                          | 0.000046 |
| creating table                 | 0.000073 |
| After create                   | 0.028503 |
| System lock                    | 0.000018 |
| copy to tmp table              | 0.000141 |
| rename result table            | 0.000991 |
| waiting for handler commit     | 0.000040 |
| waiting for handler commit     | 0.004666 |
| waiting for handler commit     | 0.000012 |
| waiting for handler commit     | 0.017665 |
| waiting for handler commit     | 0.000018 |
| waiting for handler commit     | 0.003949 |
| waiting for handler commit     | 0.000016 |
| waiting for handler commit     | 0.003579 |
| end                            | 0.000500 |
| query end                      | 0.002851 |
| closing tables                 | 0.000025 |
| waiting for handler commit     | 0.000044 |
| freeing items                  | 0.001690 |
| cleaning up                    | 0.000235 |
+--------------------------------+----------+
27 rows in set, 1 warning (0.001 sec)
```
</details>

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

#### Ответ на задание 4.

```BASH
# grep -v ^# my.cnf/my.cnf 

[mysqld]

innodb_file_per_table = 1		// в этом параметре и следующим за ним 
innodb_file_format = Barracuda		// задаются параметры сжатия таблиц типа InnoDB
innodb_flush_log_at_trx_commit = 0	// эта настройка позволяет кратно повысить производительность чтения/записи в ущерб сохранности данных
innodb_buffer_pool_size = 1G		// под кэш задается 30% от доступной на сервере оперативной памяти
innodb_log_file_size = 100M		// размер файла логов операций

skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
```
---

### Как оформить ДЗ

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---


