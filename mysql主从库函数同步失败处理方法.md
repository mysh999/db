## 一、故障描述

mysql从库查询同步信息时，报以下错误;

```sql
mysql> show slave status\G;

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xx.xx.50
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: sz-bin.000768
          Read_Master_Log_Pos: 948619766
               Relay_Log_File: sz-relay-bin.002239
                Relay_Log_Pos: 488907497
        Relay_Master_Log_File: sz-bin.000763
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1418
                   Last_Error: Error 'This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)' on query. Default database: 'campusucslearndb'. Query: 'CREATE DEFINER=`campus_south`@`xx.xx.xx.%` FUNCTION `currval`(`v_seq_name` varchar(50)) RETURNS int(11)
BEGIN
         declare value integer;         
    set value = 0;         
    select current_val into value  from t_config_sequence where seq_name = v_seq_name;   
   return value;  
END'
                 Skip_Counter: 0
```





## 二、问题原因

在主库上创建存储过程时

开启了bin-log, 必须指定我们的函数是否是
1 DETERMINISTIC 不确定的
2 NO SQL 没有SQl语句，当然也不会修改数据
3 READS SQL DATA 只是读取数据，当然也不会修改数据
4 MODIFIES SQL DATA 要修改数据
5 CONTAINS SQL 包含了SQL语句





## 三、解决办法

3.1、主库上查询function参数

```sql
mysql> show variables  like  'log_bin_trust_function_creators';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin_trust_function_creators | ON    |
+---------------------------------+-------+
```



3.2、从库上查询该参数

```sql
mysql> show variables  like  'log_bin_trust_function_creators';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin_trust_function_creators | OFF   |
+---------------------------------+-------+
1 row in set (0.00 sec)
```



3.3、从库修改该参数

```bash
mysql>  set  global log_bin_trust_function_creators = 1 ;
```



3.4、从库再次查询该参数

```sql
mysql> show variables  like  'log_bin_trust_function_creators';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin_trust_function_creators | ON    |
+---------------------------------+-------+
```



3.4、从库执行重新同步

```sql
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

#没有再报错
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xx.xx.50
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: sz-bin.000768
          Read_Master_Log_Pos: 948744078
               Relay_Log_File: sz-relay-bin.002239
                Relay_Log_Pos: 496255083
        Relay_Master_Log_File: sz-bin.000763
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
```

