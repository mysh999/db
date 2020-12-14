## 一、现象描述

主从同步。报以下提示：

```bash
        Last_SQL_Error: Could not execute Delete_rows event on table xxx.xxxxxxxxx; Can't find record in 'xxxxxxxxxxxxxxxxx', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's master log aws-bin.000002, end_log_pos 64823421
```



查看同步状态，如下：

```bash
 mysql> show slave status\G; 
            Slave_IO_Running: Yes
            Slave_SQL_Running: No
  Seconds_Behind_Master: NULL
```



## 二、原因

在主上删除一条记录，从上没有对应记录



### 三、解决办法

```bash
stop slave;
set global sql_slave_skip_counter=1;
start slave;
```

