## 一、environment introduce

​         OS: Centos7

​         Mysql: 8.0.18 

         ```bash
mysql-8.0.18-1.el7.x86_64.rpm-bundle.tar
         ```





## 二、install mysql8 one by one

2.1、query if exist mariadb,if uninstall it

```bash
[root@localhost ~]# rpm -qa|grep -i mariadb
mariadb-libs-5.5.64-1.el7.x86_64
[root@localhost ~]# rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
```





2.2、install depend package

```bash
# yum -y install perl
# yum -y install  net-tools
```



2.3、install mysql8 follow 

```bash
# rpm -ivh mysql-community-common-8.0.18-1.el7.x86_64.rpm 
# rpm -ivh mysql-community-libs-8.0.18-1.el7.x86_64.rpm 
# rpm -ivh mysql-community-client-8.0.18-1.el7.x86_64.rpm 
# rpm -ivh mysql-community-server-8.0.18-1.el7.x86_64.rpm 
```



2.4、query rpm

```bash
# rpm -qa|grep -i mysql
mysql-community-common-8.0.18-1.el7.x86_64
mysql-community-client-8.0.18-1.el7.x86_64
mysql-community-libs-8.0.18-1.el7.x86_64
mysql-community-server-8.0.18-1.el7.x86_64
```



2.5、init db

```bash
# mysqld --initialize
```



2.6、modify owner and enable services

```bash
# chown mysql:mysql /var/lib/mysql -R
# systemctl start mysqld
# systemctl status mysqld
```



2.7、query root's initial password

```bash
# cat /var/log/mysqld.log |grep -i password
2020-11-22T14:48:49.976879Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: s#4Sdig2DaZ;  #password
```



2.8、modify default password

```bash
# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.18

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> alter user "root"@'localhost' identified by "abc123";
Query OK, 0 rows affected (0.02 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```



2.9、enable remote access

```bash
mysql>use mysql;
mysql>update user set host = '%' where user ='root';
mysql>ALTER USER 'root'@'%' IDENTIFIED BY 'abc123' PASSWORD EXPIRE NEVER; # modify encryption type
mysql>ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'new password'; #update password
mysql>flush privileges;
```

