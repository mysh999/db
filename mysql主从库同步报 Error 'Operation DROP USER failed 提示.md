## 一、问题描述

在mysql主从库里，主库执行了创建新库和用户、删除用户等操作，但是从库没有该用户，导致主从复制失败

在从库上查询信息如下：

```bash
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xxxxxxxxxxxxxxx
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: xxxxxx-bin.000727
          Read_Master_Log_Pos: 183165073
               Relay_Log_File: sz-relay-bin.005110
                Relay_Log_Pos: 1064067437
        Relay_Master_Log_File: sz-bin.000726
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1396
                   Last_Error: Error 'Operation DROP USER failed for 'campus_south'@'172.31.0.%'' on query. Default database: ''. Query: 'drop user 'campus_south'@'172.31.0.%''
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1064067277
              Relay_Log_Space: 1256909121
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
               Last_SQL_Error: Error 'Operation DROP USER failed for 'campus_south'@'172.31.0.%'' on query. Default database: ''. Query: 'drop user 'campus_south'@'172.31.0.%''
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 210
                  Master_UUID: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
             Master_Info_File: /data/mysql_data/master.info
```





## 二、解决办法

2.1、在从库上创建对应用户

```bash
mysql> GRANT ALL ON `southaccountdb`.* TO 'campus_south'@'172.31.0.%' identified by 'xxxxxxxxxxxxxxxxxxxxxx';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```



2.2 、从库上执行以下命令重启复制

```bash
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```



2.3、再执行查看命令，状态正常

```bash
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: XXXXXXXXXXXXXXXX
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: sz-bin.000727
          Read_Master_Log_Pos: 184092534
               Relay_Log_File: sz-relay-bin.005110
                Relay_Log_Pos: 1070784173
        Relay_Master_Log_File: sz-bin.000726
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
          Exec_Master_Log_Pos: 1070784013
              Relay_Log_Space: 1257836912
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: Yes
           Master_SSL_CA_File: /etc/mysqlcert/master/ca.pem
           Master_SSL_CA_Path: 
              Master_SSL_Cert: /etc/mysqlcert/master/client-cert.pem
            Master_SSL_Cipher: 
               Master_SSL_Key: /etc/mysqlcert/master/client-key.pem
        Seconds_Behind_Master: 20810
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
```

