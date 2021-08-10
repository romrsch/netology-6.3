## Домашнее задание к занятию "6.3. MySQL"

### Задача 1

Используя docker поднимите инстанс MySQL (версию 8). 

Данные БД сохраните в `volume.`

* Изучите бэкап БД и восстановитесь из него.
* Перейдите в управляющую консоль mysql внутри контейнера.
* Используя команду \h получите список управляющих команд.
* Найдите команду для выдачи статуса БД и приведите в ответе из ее вывода версию сервера БД.
* Подключитесь к восстановленной БД и получите список таблиц из этой БД.
* Приведите в ответе количество записей с `price > 300.`

В следующих заданиях мы будем продолжать работу с данным контейнером.


***Ответ:***
```
docker search mysql
docker pull mysql:8.0
docker volume create vol_mysql
docker volume ls
```
Запуск контейнера  
```
docker run --rm --name mysql-docker --hostname mysql-docker -e MYSQL_ROOT_PASSWORD=12345678 -ti -p 3306:3306 -v vol_mysql:/etc/mysql/ mysql:8.0
```
![альт](https://i.ibb.co/ygrsqMF/4.jpg)


Скопируем бэкап БД `test_dump.sql` в контейнер
```
user@romrsch:~/netology/6.3$ docker cp test_dump.sql mysql-docker:/tmp
```

Запустим bash в контейнере `mysql-docker` и перейдем в него:

![альт](https://i.ibb.co/rm83gnJ/3.jpg)

* Перейдём в управляющую консоль mysql внутри контейнера `mysql-docker`

![альт](https://i.ibb.co/H2d3RQf/5.jpg)

* Восстановим ранее скопированный бэкап БД `test_dump.sql` в созданную пустую БД `test_db`
```
root@mysql-docker:/# mysql -u root -p test_db </tmp/test_dump.sql
```

Используя команду `\h` получим список управляющих команд.
![альт](https://i.ibb.co/X4VWzy4/6.jpg)

Команда для выдачи статуса БД `status`

![альт](https://i.ibb.co/L9DXFG6/7.jpg)

Приведём количество записей с `price > 300`.

![альт](https://i.ibb.co/Ms6QjkJ/8.jpg)

---
### Задача 2

Создайте пользователя `test` в БД c паролем `test-pass`, используя:

* плагин авторизации mysql_native_password
* срок истечения пароля - 180 дней
* количество попыток авторизации - 3
* максимальное количество запросов в час - 100
* аттрибуты пользователя:
* Фамилия "Pretty"
* Имя "James"
* Предоставьте привелегии пользователю test на операции SELECT базы test_db.

Используя таблицу `INFORMATION_SCHEMA.USER_ATTRIBUTES` получите данные по пользователю `test` и приведите в ответе к задаче.

***Ответ:***
```
mysql> CREATE USER 'test'@'localhost' IDENTIFIED BY 'test-pass';
Query OK, 0 rows affected (0.21 sec)

mysql> ALTER USER 'test'@'localhost' ATTRIBUTE '{"fname":"James", "lname":"Pretty"}';
Query OK, 0 rows affected (0.04 sec)

mysql> ALTER USER 'test'@'localhost'
    -> IDENTIFIED BY 'test-pass'
    -> WITH
    -> MAX_QUERIES_PER_HOUR 100
    -> PASSWORD EXPIRE INTERVAL 180 DAY
    -> FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2;
Query OK, 0 rows affected (0.03 sec)
```
![альт](https://i.ibb.co/5RRtft1/9.jpg)

Предоставим привелегии пользователю test на операции SELECT базы `test_db`.

Используя таблицу `INFORMATION_SCHEMA.USER_ATTRIBUTES` получим данные по пользователю test 

```
mysql> GRANT Select ON test_db.orders TO 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "James", "lname": "Pretty"} |
+------+-----------+---------------------------------------+
1 row in set (0.02 sec)

mysql> 
```
![альт](https://i.ibb.co/NWwVXbq/10.jpg)

---
### Задача 3

Установите профилирование `SET profiling = 1`. Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой engine используется в таблице БД test_db и приведите в ответе.

Измените engine и приведите время выполнения и запрос на изменения из профайлера в ответе:
* на MyISAM
* на InnoDB

***Ответ:***

```
mysql> SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and  TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc;
```
В таблице БД `test_db` используется движок (подсистема хранения данных) `InnoDB`

![альт](https://i.ibb.co/ggQr3xb/15.jpg)

Установим профилирование `SET profiling = 1`.

![альт](https://i.ibb.co/N3JYpWB/14.jpg)

```
mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.24 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> 
mysql> ALTER TABLE orders ENGINE = InnoDB;
Query OK, 5 rows affected (0.11 sec)
Records: 5  Duplicates: 0  Warnings: 0

```
Вывод профилирования команд `SHOW PROFILES`;

![альт](https://i.ibb.co/X4jgqc6/13.jpg)


Продолжительность переключения на `MyISAM`: 0,091

Продолжительность переключения на `InnoDB`: 0,093

---
### Задача 4

Изучите файл `my.cnf` в директории /etc/mysql.
Измените его согласно ТЗ (движок InnoDB):

* Скорость IO важнее сохранности данных
* Нужна компрессия таблиц для экономии места на диске
* Размер буффера с незакомиченными транзакциями 1 Мб
* Буффер кеширования 30% от ОЗУ
* Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf.`

***Ответ:***

```
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/

# Позволяет выбрать стратегию сброса данных на диск при работе MySQL
# Значение "0" даст наибольшую производительность.
# В этом случае риск потери данных возрастает.

innodb_flush_log_at_trx_commit = 0 

# Формат файлов Barracuda — поддерживает компрессию
innodb_file_format = Barracuda

# Размер буфера транзакций, которые не были еще закомичены.
innodb_log_buffer_size	= 1M

# Буффер кэширования, 30% от ОЗУ
key_buffer_size = 1200М

# Размер файла логов - 100Mb
max_binlog_size	= 100M

```
---


