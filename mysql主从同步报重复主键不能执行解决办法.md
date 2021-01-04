## 一、现象

A、B两套做了互为主从的数据库，同步警告邮件报以下提示：

```bash
异常主机: ali-us-AAA-db

异常情况: MySQL Slave is down

异常时间: 2021-01-04 10:35:01

Last_SQL_Errno: 1062

Last_SQL_Error: Could not execute Write_rows event on table aaa.user_ext_info; Duplicate entry '2194024' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log sz-bin.000818, end_log_pos 712487991




异常主机: ali-BBB-alpha-db

异常情况: MySQL Slave is down

异常时间: 2021-01-04 10:20:01

Last_SQL_Errno: 1062

Last_SQL_Error: Could not execute Write_rows event on table bbb.user_ext; Duplicate entry '2194024' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log us-bin.000397, end_log_pos 807753339




A主机查看同步状态：
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: BBBBB
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: us-bin.000398
          Read_Master_Log_Pos: 965250455
               Relay_Log_File: sz-relay-bin.005042
                Relay_Log_Pos: 442225861
        Relay_Master_Log_File: us-bin.000397
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,information_schema,performance_schema,test,myopenfire
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1062
                   Last_Error: Could not execute Write_rows event on table aaa.user_ext_info; Duplicate entry '2194024' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log us-bin.000397, end_log_pos 807753689
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 807753370
              Relay_Log_Space: 1845122637
              Until_Condition: None
               Until_Log_File: 
               
               
               
               
               
               
B主机查看同步状态：
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: AAAAA
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: sz-bin.000819
          Read_Master_Log_Pos: 597198797
               Relay_Log_File: us-relay-bin.004563
                Relay_Log_Pos: 311391531
        Relay_Master_Log_File: sz-bin.000818
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,information_schema,performance_schema,test,myopenfire
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1062
                   Last_Error: Could not execute Write_rows event on table aaa.user_ext_info; Duplicate entry '2194024' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log sz-bin.000818, end_log_pos 712487991
```



## 二、原因分析

A\B库本身含有这些主键的记录，程序再往这些表插入同ID记录，报错



## 三、解决办法

A、B库采用以下方法执行删除记录

```bash
SQL>stop slave;
SQL>delete from  aaa.user_ext_info where id=2194024;
SQL>delete from  bbb.user_ext where id=2194024;
SQL>start slave;

# 此处是程序自动执行插入，无需手动再插入记录，如果是新sql执行插入，手动执行即可

mysql> show slave status\G;
#正常
```

