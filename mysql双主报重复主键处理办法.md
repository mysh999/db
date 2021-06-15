## 一、问题描述

mysql双主架构，报以下提示：

```bash
mysql> show slave status\G;
 Last_Error: Could not execute Write_rows event on table community.user_ext; Duplicate entry '2287915' for key 'PRIMARY', Error_code: 1062; handler error HA_ERR_FOUND_DUPP_KEY; the event's master log us-bin.000480, end_log_pos 324802580
```



查询该id存在记录

```bash
mysql> select * from community.user_ext where user_id='2287915';
```



## 二、解决办法

分别在双主上执行删除该id命令

```bash
mysql> delete  from community.user_ext where user_id='2287915';
Query OK, 1 row affected (0.01 sec)

mysql> stop slave ;
Query OK, 0 rows affected (0.05 sec)

mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status\G;  #正常
```

