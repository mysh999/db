## 一、问题描述

mysql登录显示

```bash
$ mysql -uroot -p
Enter password: 
ERROR 1040 (HY000): Too many connections
```





## 二、解决办法 

2.1、重启mysql服务释放连接

```bash
# systemctl restart mysql
```



2.2、查看连接数配置

```bash
mysql> select @@max_connections;
+-------------------+
| @@max_connections |
+-------------------+
|              1000 |
+-------------------+
1 row in set (0.00 sec)
```



2.3、查看当前连接

```bash
mysql> show processlist;
+-----+--------------+--------------------+---------------------+---------+------+-------+------------------+
| Id  | User         | Host               | db                  | Command | Time | State | Info             |
+-----+--------------+--------------------+---------------------+---------+------+-------+------------------+
|   1 | root         | 10.10.1.49:43268   | edutoplearnlivedb   | Sleep   |   38 |       | NULL             |
|   2 | root         | 10.10.1.12:33678   | jimudb              | Sleep   |    2 |       | NULL             |
|   3 | root         | 10.10.1.12:33690   | operation           | Sleep   |   13 |       | NULL             |
|   4 | root         | 10.10.1.12:33692   | NULL                | Sleep   |   13 |       | NULL             |
|   5 | root         | 10.10.1.12:33694   | mqtt                | Sleep   |   32 |       | NULL             |
|   6 | root         | 10.10.1.12:33696   | mqtt                | Sleep   |   38 |       | NULL             |
|   7 | root         | 10.10.2.253:57344  | mqtt                | Sleep   |    6 |       | NULL             |
|   8 | root         | 10.10.2.253:47945  | mqtt                | Sleep   |    8 |       | NULL             |
|   9 | root         | 10.10.1.64:52347   | mqtt                | Sleep   |   38 |       | NULL             |
|  10 | root         | 10.10.1.64:33266   | mqtt                | Sleep   |   27 |       | NULL             |
|  11 | root         | 10.10.1.64:37101   | mqtt                | Sleep   |   27 |       | NULL             |
|  12 | root         | 10.10.1.64:59620   | mqtt                | Sleep   |   38 |       | NULL             |
|  13 | root         | 10.10.1.64:39545   | mqtt                | Sleep   |   38 |       | NULL             |
|  14 | root         | 10.10.1.64:43778   | mqtt                | Sleep   |   38 |       | NULL             |
|  15 | root         | 10.10.1.64:50750   | mqtt                | Sleep   |   38 |       | NULL             |
|  16 | root         | 10.10.1.64:55932   | mqtt                | Sleep   |   38 |       | NULL             |
|  17 | root         | 10.10.2.13:53647   | mqtt                | Sleep   |   38 |       | NULL             |
|  18 | root         | 10.10.2.13:57703   | mqtt                | Sleep   |   38 |       | NULL             |
|  19 | root         | 10.10.2.14:41172   | mqtt                | Sleep   |   38 |       | NULL             |
```

在对端服务器上查看端口对应的进程

```bash
# netstat -tnp|grep 41268
tcp        0      0 10.10.1.12:3306         10.10.1.12:41268        ESTABLISHED 2186053/mysqld      
tcp6       0      0 10.10.1.12:41268        10.10.1.12:3306         ESTABLISHED 3481023/java       

# 查看对应进程
# ls -lt /proc/3481023/cwd
lrwxrwxrwx 1 root root 0 Aug 13 14:25 /proc/3481023/cwd ->XXXXXXXXXXXXXXXXXXXX
```







2.4、修改maxconnection大小

```bash
# vi /etc/my.cnf
max_connections=1500   #改大

# 重启服务
# systemctl restart mysql
```



再联系开发检查连接多的会话工程是否能加以限制