## 一、环境介绍

mysql配置主从，主是容器环境，从是物理机环境

从上已安装mysql软件

启用bin log日志



## 二、从机新建数据库实例

2.1、准备目录

```bash
# mkdir -p /data/mysql3308_data
# touch /data/mysql3308_data/mysqld.log
# chown -R mysql.mysql /data/mysql3308_data/
```



2.2、准备my.cnf文件

```bash
#vi /etc/my3308.cnf
[mysqld]
server-id = 3308
port=3308
max_connections=1000
group_concat_max_len = 409600

log-bin = sz-bin
log-bin-index = sz-bin.index
binlog_format = row
expire_logs_days = 1
skip-name-resolve
slave_net_timeout = 10
relay-log = sz-relay-bin
relay-log-index = sz-relay-bin.index
read-only = 1
relay_log_recovery = 1
relay_log_info_repository = TABLE
init-connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
datadir=/data/mysql3308_data
symbolic-links=0
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
[mysqld_safe]
log-error=/data/mysql3308_data/mysqld.log
socket=/data/mysql3308_data/mysql.sock

[client]
socket=/data/mysql3308_data/mysql.sock
```



2.3、初始化数据

```bash
#  mysql_install_db --defaults-file=/etc/my3308.cnf  --user=mysql 
Installing MySQL system tables...2020-09-21 14:04:12 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
```



2.4 、启动

```bash
# nohup mysqld_safe --defaults-file=/etc/my3308.cnf --basedir=/usr --datadir=/data/mysql3308_data --user=mysql &  
```



2.5 、重置密码

```bash
# mysqladmin -uroot password 'XXXXXXXXXXXXXXXX' -S /data/mysql3308_data/mysql.sock 
```



2.6、登录

```bash
# mysql -uroot -p -S /data/mysql3308_data/mysql.sock 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
```





## 三、主库数据备份与从库还原数据

数据量小可以使用mysqldump，大的话使用xtrabackup

过程略



## 四、配置主从

4.1、主服务器创建复制账号

```bash
mysql> grant replication slave on *.* to 'repl'@'%' identified by 'xxxxxxxxxxxxxxxxxx';
mysql> flush privileges;

mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000016
         Position: 1845722
     Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql
Executed_Gtid_Set: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```



4.2、从库配置

```bash
stop slave;
reset slave;
CHANGE MASTER TO MASTER_HOST='XX.XX.XX.XX',MASTER_PORT=3306,MASTER_USER='repl',MASTER_PASSWORD='xxxxxxxxx',MASTER_LOG_FILE='mysql-bin.000016', MASTER_LOG_POS=1845722;
start slave;
```



4.3、从库查看同步情况

```bash
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xxxxxxxxx
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000016
          Read_Master_Log_Pos: 1856280
               Relay_Log_File: sz-relay-bin.000002
                Relay_Log_Pos: 10841
        Relay_Master_Log_File: mysql-bin.000016
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1856280
              Relay_Log_Space: 11011
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 100
                  Master_UUID: c62eedaf-ec3c-11ea-aca7-0242ac120002
             Master_Info_File: /data/mysql3308_data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)
```

