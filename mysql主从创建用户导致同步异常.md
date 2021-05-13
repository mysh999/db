## 一、问题描述

 mysql主从架构

在主服务器上创建了用户，导致从库同步异常



从库查看同步信息

```bash
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xx.xx.210
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: sz-bin.000886
          Read_Master_Log_Pos: 214807995
               Relay_Log_File: sz-relay-bin.005635
                Relay_Log_Pos: 184899373
        Relay_Master_Log_File: sz-bin.000886
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1396
                   Last_Error: Error 'Operation CREATE USER failed for 'xxxx.liao'@'%'' on query. Default database: ''. Query: 'CREATE USER 'xxxx.liao'@'%' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618Bxxxxxxx5ADCA2EC9D1E19''
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 184899213
              Relay_Log_Space: 214808369
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: Yes
           Master_SSL_CA_File: /etc/mysqlcert/master/ca.pem
           Master_SSL_CA_Path: 
              Master_SSL_Cert: /etc/mysqlcert/master/client-cert.pem
            Master_SSL_Cipher: 
               Master_SSL_Key: /etc/mysqlcert/master/client-key.pem
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 1396
               Last_SQL_Error: Error 'Operation CREATE USER failed for 'xxxx.liao'@'%'' on query. Default database: ''. Query: 'CREATE USER 'xxxx.liao'@'%' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618Bxxxxxxx5ADCA2EC9D1E19''
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 210
                  Master_UUID: 3a7defb4-445c-11e8-9b4a-00163e0cf1bc
             Master_Info_File: /data/mysql_data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 210513 13:42:27
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

```





## 二、问题分析

由于之前在主库上有创建同名用户，后来又做了删除，这次再创建，怀疑从库对应信息同步报错





## 三、解决办法

3.1、从库查询用户

```bash
mysql> select user,host from user;
+---------------+----------------+
| user          | host           |
+---------------+----------------+
| xxxx.liao     | %              |

mysql> show grants for 'xxxx.liao'@'%';
+----------------------------------------------------------------------------------------------------------+
| Grants for xxxx.liao@%                                                                                   |
+----------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'xxxx.liao'@'%' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618Bxxxxxxx5ADCA2EC9D1E19' |
| GRANT ALL PRIVILEGES ON `xxxx`.* TO 'xxxx.liao'@'%'                                                  |
| GRANT ALL PRIVILEGES ON `xxxx`.* TO 'xxxx.liao'@'%'                                                  |
+----------------------------------------------------------------------------------------------------------+
```



3.2、从库回收权限

```bash
mysql> revoke  ALL PRIVILEGES ON `xxxx`.*  from 'xxxx.liao'@'%';     
Query OK, 0 rows affected (0.02 sec)

mysql> revoke  ALL PRIVILEGES ON `xxxx`.*  from 'xxxx.liao'@'%'; 
Query OK, 0 rows affected (0.00 sec)

mysql> revoke  ALL PRIVILEGES ON *.*  from 'xxxx.liao'@'%';          
Query OK, 0 rows affected (0.00 sec)

```





3.3、重新同步

用户会自动同步 

```bash
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

 mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xx.xx.210
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: sz-bin.000886
          Read_Master_Log_Pos: 230953228
               Relay_Log_File: sz-relay-bin.005640
                Relay_Log_Pos: 10781887
        Relay_Master_Log_File: sz-bin.000886
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
          Exec_Master_Log_Pos: 230953228
              Relay_Log_Space: 12270034
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: Yes
           Master_SSL_CA_File: /etc/mysqlcert/master/ca.pem
           Master_SSL_CA_Path: 
              Master_SSL_Cert: /etc/mysqlcert/master/client-cert.pem
            Master_SSL_Cipher: 
               Master_SSL_Key: /etc/mysqlcert/master/client-key.pem
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 210
                  Master_UUID: 3a7defb4-445c-11e8-9b4a-00163e0cf1bc
             Master_Info_File: /data/mysql_data/master.info
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

ERROR: 
No query specified
```



