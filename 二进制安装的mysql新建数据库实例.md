## 一、背景

使用yum安装的mysql5.6下，新建一个数据库实例





##   二、准备my.cnf文件

```bash
# cat /etc/my3307.cnf 
[mysqld]
server-id = 3307
port=3307
max_connections=1000
group_concat_max_len = 409600

log-bin = sz-bin
log-bin-index = sz-bin.index
binlog_format = row
expire_logs_days = 1
skip-name-resolve
slave_net_timeout = 10
# slave_skip_errors = 1062
# binlog-ignore-db = mysql
# binlog-ignore-db = information_schema
# binlog-ignore-db = performance_schema
# binlog-ignore-db = test
# replicate-ignore-db = mysql
# replicate-ignore-db = information_schema
# replicate-ignore-db = performance_schema
# replicate-ignore-db = test
# replicate-ignore-db = myopenfire

relay-log = sz-relay-bin
relay-log-index = sz-relay-bin.index
# log-slave-updates = 1
read-only = 1
relay_log_recovery = 1
relay_log_info_repository = TABLE

init-connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

datadir=/data/mysql3307_data

symbolic-links=0

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/data/mysql3307_data/mysqld.log
socket=/data/mysql3307_data/mysql.sock

[client]
socket=/data/mysql3307_data/mysql.sock
```



## 三、初始化数据

```bash
# mysql_install_db --defaults-file=/etc/my3307.cnf  --user=mysql
Installing MySQL system tables...2020-09-07 14:35:58 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
```



## 四、启动

```bash
# nohup mysqld_safe --defaults-file=/etc/my3307.cnf --basedir=/usr --datadir=/data/mysql3307_data --user=mysql &
```





## 五、重置密码

```bash

# mysqladmin -uroot password 'XXXXX' -S /data/mysql3307_data/mysql.sock 
Warning: Using a password on the command line interface can be insecure.
```





##   六、登录

```bash
登录
# mysql -uroot -p -S  /data/mysql3307_data/mysql.sock 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.37-log MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

mysql> 
```

