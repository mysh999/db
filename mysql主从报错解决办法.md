## 一、问题描述

mysql主从架构，在进行同步的时候经常会报以下错误，导致同步失败

```bash
mysql> show slave status\G;
--- ---
             Slave_IO_Running: No
            Slave_SQL_Running: Yes



                Last_IO_Error: The slave I/O thread stops because a fatal error is encountered  when it tries to SET @master_heartbeat_period on master. Error: 
```





## 二、原因分析

可能由于跨网络质量不佳，导致同步经常终止



```bash
mysql> show status like 'slave%';
+----------------------------+---------------------+
| Variable_name              | Value               |
+----------------------------+---------------------+
| Slave_heartbeat_period     | 5.000               |
| Slave_last_heartbeat       | 2021-07-27 16:20:50 |
| Slave_open_temp_tables     | 0                   |
| Slave_received_heartbeats  | 433                 |
| Slave_retried_transactions | 0                   |
| Slave_running              | OFF                 |
+----------------------------+---------------------+
6 rows in set (0.00 sec)
```

 Slave_last_heartbeat 表示最后一次收到心跳的时间，Slave_received_heartbeats 表示总共收到的心跳次数





##  三、解决办法

```bash
mysql> stop slave;
mysql> change master to master_heartbeat_period = 100;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> set global slave_net_timeout = 250;
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show status like 'slave%';
+----------------------------+---------------------+
| Variable_name              | Value               |
+----------------------------+---------------------+
| Slave_heartbeat_period     | 100.000             |
| Slave_last_heartbeat       | 2021-07-27 16:20:50 |
| Slave_open_temp_tables     | 0                   |
| Slave_received_heartbeats  | 0                   |
| Slave_retried_transactions | 0                   |
| Slave_running              | ON                  |
+----------------------------+---------------------+
6 rows in set (0.00 sec)
```

