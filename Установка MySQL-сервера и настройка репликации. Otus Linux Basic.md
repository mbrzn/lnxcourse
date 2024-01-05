---
aliases:
  - "{ Установка MySQL-сервера и настройка репликации. Otus Linux Basic"
type: course
---

[Установка MySQL-сервера и настройка репликации](https://otus.ru/learning/261941/#)

[[Лавлинский Николай]]

Лекция 6 декабря, среда в 20:00


## архитектрура mysql

![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic.jpg]]

Простой клиент - это консоль mysql, собственно к нему и следует обращаться в скриптах bash, чтобы[[bash интерфейс mysql сервера| управлять БД ]]
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-1.jpg]]

*x protocol 33060* не понадобится.
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-2.jpg]]

*protocol 3306* - основной сокет
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-3.jpg]]
многопоточный
## установка mysql 8.0
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-4.jpg]]

Ставить нужно на две машины, чтобы обеспечить master и slave репликацию.

11:23 Локально можно войти без пароля - так устроена  система безопасности mysql, ведь root локальный OC-root и так уже полноправен, дополнительный пароль для входа его в оболочку mysql сервера будет излишним усложнением.
```bash
$ sudo apt install mysql-server-8.0
...
Получено 29,6 MB за 52с (563 kB/s)
Предварительная настройка пакетов …
...
$ sudo mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.35-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.
...
```

```mysql
# Интерактивная оболочка mysql сервера
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

# берем все поля из таблицы User, с условием поле User=root
mysql> SELECT * FROM user WHERE User='root';
# очень широкая таблица... выводим в другом формате:
mysql> SELECT * FROM user WHERE User='root'\G
*************************** 1. row ***************************
                    Host: localhost
                    # пользовтель mysql, а не ОС
                    # причем это пользователь именно localhost-а, т.е. подключился
                    # локально к этой базе данных
                    # его полное имя root@localhost
                    User: root
             Select_priv: Y
...
    max_user_connections: 0
				# способ аутентификации
				# плагин, который разрешает войти в бд локально
                  plugin: auth_socket
                # здесь нет пароля, пускает на основании того,что я - root
                # удаленное подключение так работать не будет - запрещено
   authentication_string:
        password_expired: N
   password_last_changed: 2023-12-07 14:25:57
       password_lifetime: NULL
          account_locked: N
...
1 row in set (0,00 sec)

mysql> exit
Bye
administrator@u22srv:~$
```
Можно, при желании установить пароль для входа в оболочку mysql сервера:
```mysql
# Устанавливаем пароль 'Testpass1$' для пользователя root
ALTER USER 'root'@'localhost' IDENTIFIED WITH 'caching_sha2_password' BY 'Testpass1$';

# 5.7 версия
ALTER USER 'root'@'localhost' IDENTIFIED WITH 'mysql_native_password' BY 'Testpass1$';
```
## файловая структура mysql
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-5.jpg]]
... здесь документация конфигов для разных ОС.

```bash
$ ll
total 32
drwxr-xr-x   4 root root 4096 дек  7 14:25 ./
drwxr-xr-x 109 root root 4096 дек  7 14:26 ../
drwxr-xr-x   2 root root 4096 дек  7 14:25 conf.d/
-rw-------   1 root root  317 дек  7 14:25 debian.cnf
-rwxr-xr-x   1 root root  120 окт 25 17:34 debian-start*
lrwxrwxrwx   1 root root   24 дек  7 14:25 my.cnf -> /etc/alternatives/my.cnf
-rw-r--r--   1 root root  839 окт 20  2020 my.cnf.fallback
-rw-r--r--   1 root root  682 июн 14 19:23 mysql.cnf
drwxr-xr-x   2 root root 4096 дек  7 14:25 mysql.conf.d/
$ cat my.cnf
# это файл - заглушка
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
# сюда пишутся реальные настройки

/etc/mysql/conf.d$ cat mysql.cnf
[mysql]
# это секция конфига 
# это настройки клиента (а не сервера!) на уровне всей системы

# а вот это - сервер, d - демон
/etc/mysql/mysql.conf.d$ nano mysqld.cnf
...
# прослушивает 127..., значит подключиться к этой машине извне нельзя
bind-address            = 127.0.0.1
# меняем это
bind-address            = 0.0.0.0
...
$ ss -ntpl
State      Recv-Q     Send-Q         Local Address:Port          Peer Address:Port     Process
LISTEN     0          151                127.0.0.1:3306               0.0.0.0:*
# ? как поменять, у sql нет reload...
...
$ sudo service mysql restart
# или, более грамотно:
$ sudo systemctl restart mysql
$ sudo systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-12-07 15:09:03 UTC; 23s ago
...

$ ss -ntpl
State      Recv-Q     Send-Q         Local Address:Port          Peer Address:Port     Process
LISTEN     0          151                  0.0.0.0:3306               0.0.0.0:*

```

26:44 Файлы самой sql:
```bash
$ sudo ls -l /var/lib/mysql
total 91612
-rw-r----- 1 mysql mysql       56 дек  7 14:25  auto.cnf
-rw-r----- 1 mysql mysql      180 дек  7 14:26  binlog.000001
-rw-r----- 1 mysql mysql      404 дек  7 14:26  binlog.000002
-rw-r----- 1 mysql mysql      180 дек  7 15:04  binlog.000003
-rw-r----- 1 mysql mysql      180 дек  7 15:08  binlog.000004
-rw-r----- 1 mysql mysql      157 дек  7 15:09  binlog.000005
-rw-r----- 1 mysql mysql       80 дек  7 15:09  binlog.index
-rw------- 1 mysql mysql     1705 дек  7 14:25  ca-key.pem
-rw-r--r-- 1 mysql mysql     1112 дек  7 14:25  ca.pem
-rw-r--r-- 1 mysql mysql     1112 дек  7 14:25  client-cert.pem
-rw------- 1 mysql mysql     1705 дек  7 14:25  client-key.pem
...

```


```bash
# Процессы

ps ax | grep mysqld

# Потоки
ps -eLf | grep mysqld

# Файлы
ls -l /var/lib/mysql

# Тип бинлога
show variables like '%binlog%';
```

## понятие репликации сервера mysql

![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-6.jpg]]


![[Pasted image 20231208011808.jpg]]

![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-8.jpg]]

Данные приезжают на реплику потом, мастер не ждет ничего.
Binlog - это журнал.

![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-10.jpg]]


![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-11.jpg]]
`commit` - фиксирование завершенности операции.
![[2023-12-08 01_23_27-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]
Более надежная, но более медленная схема.
![[2023-12-08 01_25_10-Мои курсы — Administrator Linux.Basic _ OTUS – Brave.jpg]]

## настройка бинарных логов
46:30
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-14.jpg]]

```mysql
# Тип бинлога
show variables like '%binlog%';
# здесь % означает любой символ в любом количестве:
+------------------------------------------------+----------------------+
| Variable_name                                  | Value                |
+------------------------------------------------+----------------------+
| binlog_cache_size                              | 32768                |
| binlog_checksum                                | CRC32                |
| binlog_direct_non_transactional_updates        | OFF                  |
| binlog_encryption                              | OFF                  |
| binlog_error_action                            | ABORT_SERVER         |
| binlog_expire_logs_auto_purge                  | ON                   |
| binlog_expire_logs_seconds                     | 2592000              |
| binlog_format                                  | ROW                  |
| binlog_group_commit_sync_delay                 | 0                    |
| binlog_group_commit_sync_no_delay_count        | 0                    |
| binlog_gtid_simple_recovery                    | ON                   |
| binlog_max_flush_queue_time                    | 0                    |
| binlog_order_commits                           | ON                   |
| binlog_rotate_encryption_master_key_at_startup | OFF                  |
| binlog_row_event_max_size                      | 8192                 |
| binlog_row_image                               | FULL                 |
| binlog_row_metadata                            | MINIMAL              |
| binlog_row_value_options                       |                      |
| binlog_rows_query_log_events                   | OFF                  |
| binlog_stmt_cache_size                         | 32768                |
| binlog_transaction_compression                 | OFF                  |
| binlog_transaction_compression_level_zstd      | 3                    |
| binlog_transaction_dependency_history_size     | 25000                |
| binlog_transaction_dependency_tracking         | COMMIT_ORDER         |
| innodb_api_enable_binlog                       | OFF                  |
| log_statements_unsafe_for_binlog               | ON                   |
| max_binlog_cache_size                          | 18446744073709547520 |
| max_binlog_size                                | 104857600            |
| max_binlog_stmt_cache_size                     | 18446744073709547520 |
| sync_binlog                                    | 1                    |
+------------------------------------------------+----------------------+
30 rows in set (0,06 sec)


```

![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-15.jpg]]
Бинлог - очень полезное дополнение к резервной копии, желательно хранить неделю.

## настройка gtid репликации
56:00
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-16.jpg]]
если склонировали виртуалки, то auto.cnf будет одинаковый, а он должен быть уникальным для каждой машины, поэтому 
- на одном из серверов этот файл удалить и
- делать restart sql - автоматически сформируется с новым server id

```mysql
# Найти server_id
SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           1 |
+-------------+
1 row in set (0,00 sec)

```

> [!Attention] Нет репликации - нет DB
> До начала работы с приложением нужно обязательно создать реплику на другом сервере. 

![[GTID репликация MySQL сервера#Настройка GTID репликации]]

Файрвол может блокировать работу, поэтому его можно отключить:
```bash
$iptables -nvL
```

```bash
/etc/mysql/mysql.conf.d$ sudo nano mysqld.cnf
...
# на всякий случай явно прописываем id
server-id               = 1
log_bin                 = /var/log/mysql/mysql-bin.log
# время хранения бинлога
binlog_expire_logs_seconds      = 2592000
max_binlog_size   = 100M

gtid-mode=ON
enforce-gtid-consistency
log-replica-updates
...

$ sudo systemctl restart mysql

# Процессы

ps ax | grep mysqld
$ ps ax | grep mysqld
 283476 ?        Ssl    0:05 /usr/sbin/mysqld
 283924 pts/1    S+     0:00 grep --color=auto mysqld
# стартанул

# Потоки
ps -eLf | grep mysqld
$ ps -eLf | grep mysqld
mysql     283476       1  283476  1   37 15:56 ?        00:00:02 /usr/sbin/mysqld
mysql     283476       1  283491  0   37 15:56 ?        00:00:00 /usr/sbin/mysqld
mysql     283476       1  283492  0   37 15:56 ?        00:00:00 /usr/sbin/mysqld
...
# очень много...
```


![[создать пользователя mysql с правами на репликацию]]
Запустим mysql сервер на другом компьютере для создания реплики:
```bash
# master на комьпьютере administrator@u22srv
# slave на комьпьютере administrator@s1
$ sudo apt install mysql-server-8.0

```

```bash
# slave на комьпьютере administrator@s1
/etc/mysql/mysql.conf.d$ sudo nano mysqld.cnf
...
server-id               = 2
binlog_expire_logs_seconds      = 2592000
max_binlog_size   = 100M
log-bin = mysql-bin
relay-log = relay-log-server
read-only = ON
gtid-mode=ON
enforce-gtid-consistency
log-replica-updates

# этому серверу не нужно слышать всю сеть, от слушает локальный ip
$ ss -ntpl
State         Recv-Q        Send-Q                Local Address:Port                  Peer Address:Port        Process
LISTEN        0             70                        127.0.0.1:33060                      0.0.0.0:*
LISTEN        0             151                       127.0.0.1:3306                       0.0.0.0:*
...
$ sudo service mysql restart
$ ps afx
...
  15434 ?        Ssl    0:05 /usr/sbin/mysqld
```

1:10:16
со стороны реплики:
```mysql
mysql> STOP REPLICA;
Query OK, 0 rows affected, 1 warning (0,00 sec)
# ? какой warning здесь?

mysql> show warnings;
+-------+------+-----------------------------------------------------------+
| Level | Code | Message                                                   |
+-------+------+-----------------------------------------------------------+
| Note  | 3084 | Replication thread(s) for channel '' are already stopped. |
+-------+------+-----------------------------------------------------------+
1 row in set (0,00 sec)


# главная магическая команда репликации
# SOURCE_HOST='192.168.1.68' -  ip source-а 
# SOURCE_USER='repl' - пользователь, созданные на source
# SOURCE_PASSWORD='123456' - пароль пользователя, созданные на source
# SOURCE_AUTO_POSITION = 1 - главная фича, он сам поймет с чего начинать
# GET_SOURCE_PUBLIC_KEY = 1 - для установления защищенного соединения



# это значение применялось на семинаре
# здесь кажется лишним либо пароль, либо ключ
#mysql> CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.68', SOURCE_USER='repl', SOURCE_PASSWORD='123456', SOURCE_AUTO_POSITION = 1, GET_SOURCE_PUBLIC_KEY = 1;
Query OK, 0 rows affected, 2 warnings (0,04 sec)

# тоже, но без ключа
mysql> CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.68', SOURCE_USER='repl', SOURCE_PASSWORD='123456', SOURCE_AUTO_POSITION = 1;
Query OK, 0 rows affected, 2 warnings (0,03 sec)



mysql> show warnings;
...
| Note  | 1759 | Sending passwords in plain text without SSL/TLS is extremely insecure. # предупреждает о небезопасности открытых паролей
...
# предлагает хранить пароли в репозитории
| Note  | 1760 | Storing MySQL user name or password information in the connection metadata repository is not secure and is therefore not recommended. Please consider using the USER and PASSWORD connection options for START REPLICA; see the 'START REPLICA Syntax' in the MySQL Manual for more information.
...
2 rows in set (0,00 sec)
# это не ошибки, а лишь предупреждения

# а вот теперь можно запустить репликацию
mysql> START REPLICA;
Query OK, 0 rows affected (0,04 sec)



```

1:14:20   у меня ошибки Gtid, которых нет у лектора, [[настроить репликацию mysql  сервера|исправлены]].

- [x] до того, как начинать упражнения с созданием баз данных, обязательно настроить репликацию!

1:19:40 Остановка репликации нужна для того, чтобы делать бэкап базы - тогда получим точные консистентные данные. В это время мы точно не получаем записи в БД.
```mysql
START REPLICA;
```
## резервирование (бэкап) БД
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-20.jpg]]
Будем использовать логические. 
Пример:
![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-21.jpg]]
Здесь бэкап всей базы данных в одном файле.
Команда `mysql` получает `-e` запрос, запускает скрипта - выводит строчками базу данных. Опции команды `mysqldump` позволяют делать консистентный бэкап. Для потабличного бэкапа здесь хорошо бы тормознуть репликацию. 

- [x] В домашнем задании сделать: одна таблица - один файл.
- [x] [[сделать бэкап конкретной базы данных]]
	- [x] просмотреть бэкап в MySQL Wokrbench
- [x] сделать бэкап одной конктретной таблицы из конкретной базы данных
	- [x] просмотреть этот бэкап в MySQL Wokrbench

```bash
##################
# На Мастере
#
$ sudo mysqldump otus_db > ~/bckp/otus_db.dump.sql
...
administrator@s0:~$ ll ~/bckp/otus_db.dump.sql
-rw-rw-r-- 1 administrator administrator 2167 дек  8 17:07 /home/administrator/bckp/otus_db.dump.sql
# здесь аутентификация в mysql производится на основании, того, что пользователь root

# в случае, если пользователь mysql настроен на аутентификацию иным способом - 'User' с паролем 'Password' то:
mysqldump --all-databases -uUser -pPassword otus_db > ~/otus_db.dump.sql

```

Содержание `~/bckp/otus_db.dump.sql` файла. Много SQL-а:
```mysql
-- GTID state at the beginning of the backup
--

SET @@GLOBAL.GTID_PURGED=/*!80000 '+'*/ '82725ac9-950c-11ee-abae-080027543289:1-4';

--
-- Dumping data for table `test_tbl`
--

LOCK TABLES `test_tbl` WRITE;
/*!40000 ALTER TABLE `test_tbl` DISABLE KEYS */;
INSERT INTO `test_tbl` VALUES (2),(3),(4),(848848),(9080803),(525254);
/*!40000 ALTER TABLE `test_tbl` ENABLE KEYS */;
UNLOCK TABLES;
SET @@SESSION.SQL_LOG_BIN = @MYSQLDUMP_TEMP_LOG_BIN;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2023-12-08 17:09:53
```
Логично бэкап делать на реплике - он будет консистентным. 

```bash
####### Бекапы
mysqldump --help

# Бекап без создания таблиц
mysqldump --all-databases --no-create-info -u root -p > dump-data.sql

# C сохранением позиции бинлога
# подключение к удаленному хосту
# здесь репликация со второй позиции
mysqldump -h 10.128.15.220 -p --all-databases --events --routines --master-data=2 > dump_file

# Скачивание бинлогов
# стандартный бэкап 1 файла
mysqlbinlog -R -h 10.128.15.201 -p --raw binlog.000001

# бэкапы без остановки начиная с 1 файла
mysqlbinlog -R -h 10.128.15.201 -p --raw --stop-never binlog.000001

# заливаем данные
mysql -u root -p < dump-data.sql
# например заливаем в базу otus_db такой то дамп
#    в дампе нужно закомментарить gtid, чтобы не ругался
mysql otus_db -u root -p < dump-data.sql

# Проигрываем изменение из бинлога
mysqlbinlog --start-position=4596 binlog.000004 | mysql
```

## Домашнее задание

![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-22.jpg]]

>Настроить репликацию MySQL master-slave, настроить бэкап БД на slave (потаблично с указанием позиции бинлога)

- [x] настроить репликацию MySQL master-slave, 
- [x] настроить бэкап БД на slave (потаблично с указанием позиции бинлога)

- [ ] ? Как увидеть в реплике то, на какой момент времени она создана? Как идентифицируется этот момент?

>Цель:
>В результате выполнения ДЗ вы создадите репликацию базы данных master-slave для последующей работы с бекапами.  
>В данном задании тренируются навыки:
>
>- понимание предметной области задания
>- установка ПО на сервер, работа с файлами конфигурации
>- построения репликации master-slave с последющей настройкой бекапа"


>Описание/Пошаговая инструкция выполнения домашнего задания:
>
>Необходимо:
>- установить mysql на двух серверах
>- прверить доступность порта mysql с одного сервера на другой
>- создать пользователя для репликации на сервере master
>- настроить реплику slave
>- написать скрипт бекапа баз с реплики

- [ ] написать скрипт бекапа баз с реплики

>Критерии оценки:
>- настроена репликация master-slave - 3 балла
>- написан скрипт бекапа согласно заданию (потаблично с указанием позиции бинлога) - 4 балла
>- написан скрипт бекапа (без указания позиции бинлога) - 3 балла 
>
\\Статус "Принято" ставится от 6 баллов.

- [x] настроена репликация master-slave - 3 балла
- [x] написан скрипт бекапа согласно заданию (потаблично с указанием позиции бинлога) - 4 балла
- [x] написан скрипт бекапа (без указания позиции бинлога) - 3 балла  

*Если вы хотите получить более сложное задание, обратитесь, пожалуйста, к ментору.

>Компетенции:
>
>- умения работать с сервисами Linux
> - базовые навыки администрирования Mysql 
> 	- архитектура, 
> 	- установка, 
> 	- конфигурация, 
> 	- бэкап и 
> 	- репликация

Рекомендуем сдать до: 12.12.2023
## Литература
- [[sql index]]
- ![[Установка MySQL-сервера и настройка репликации. Otus Linux Basic-23.jpg]] 7.-8. Блог с огромной коллекцией статей для администраторов.
## Упражнения
- [[станции связи, учебная база данных MySQL]]
- [[сделать бэкап конкретной базы данных]]

