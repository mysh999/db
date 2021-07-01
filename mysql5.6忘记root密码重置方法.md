方法如下：

执行以下命令

```bash
# mysqld  --skip-grint-tables
2021-07-01 17:10:42 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2021-07-01 17:10:42 0 [Note] mysqld (mysqld 5.6.45) starting as process 20116 ...
2021-07-01 17:10:42 20116 [ERROR] Fatal error: Please read "Security" section of the manual to find out how to run mysqld as root!

2021-07-01 17:10:42 20116 [ERROR] Aborting

2021-07-01 17:10:42 20116 [Note] Binlog end
2021-07-01 17:10:42 20116 [Note] mysqld: Shutdown complete

```





```bash
# 密码部分直接回车
# mysql -uroot -p   
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.45 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> update mysql.user set password=PASSWORD('password') where user='root';
Query OK, 4 rows affected (0.00 sec)
Rows matched: 4  Changed: 4  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```





再次登陆

```bash
# mysql -uroot -p
Enter password: 
```

