## 一、问题描述

mysql主从，由于网络或者从库关机，导致同步异常，从库报错信息如下：

```bash
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 61.117.178.8
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000024
          Read_Master_Log_Pos: 973278928
               Relay_Log_File: sz-relay-bin.000626
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000024
             Slave_IO_Running: No
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
          Exec_Master_Log_Pos: 973278928
              Relay_Log_Space: 413255548
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 1236
                Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'
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
      Last_IO_Error_Timestamp: 220516 10:30:54
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





## 二、原因

Slave_IO_Running:  No 一方面原因是因为网络通信的问题也有可能是日志读取错误的问题



## 三、解决办法

3.1、从库停止slave

```bash
mysql> stop slave;
```



3.2、主库查看信息

```bash
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000028
         Position: 320432384
     Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql
Executed_Gtid_Set: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```



3.3、主库执行刷新日志

```bash
mysql> flush logs;
Query OK, 0 rows affected (0.01 sec)

mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000029
         Position: 644213
     Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql
Executed_Gtid_Set: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```



3.4、马上到从库执行

```bash
mysql> CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000029',MASTER_LOG_POS=644213;
Query OK, 0 rows affected (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)


mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 61.117.178.8
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000029
          Read_Master_Log_Pos: 4554407
               Relay_Log_File: sz-relay-bin.000006
                Relay_Log_Pos: 1317290
        Relay_Master_Log_File: mysql-bin.000029
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
          Exec_Master_Log_Pos: 4554407
              Relay_Log_Space: 1523929
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

ERROR: 
No query specified
```

