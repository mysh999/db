1、前提

mysql打开binlog功能



2、查看当前binlog位置

```bash
root@localhost[(none)] 23:24:12>show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000015 |     4985 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```



3、执行DML操作，创建一个测试表，插入2条数据后误操作删除了其中一条

```bash
root@localhost[(none)] 23:26:36>use mysh_utf8; #进入库

#创建表
root@localhost[mysh_utf8] 23:26:47>create table student(name varchar(50) default null);

#插入记录
root@localhost[mysh_utf8] 00:25:19>insert into student values('wlf');
Query OK, 1 row affected (0.01 sec)

root@localhost[mysh_utf8] 00:25:38>insert into student values('tom');
Query OK, 1 row affected (0.00 sec)

root@localhost[mysh_utf8] 00:25:41>insert into student values('jim');
Query OK, 1 row affected (0.00 sec)

root@localhost[mysh_utf8] 00:25:45>insert into student values('harry');
Query OK, 1 row affected (0.00 sec)

root@localhost[mysh_utf8] 00:25:51>insert into student values('salay');

root@localhost[mysh_utf8] 00:25:57>select * from student;
+-------+
| name  |
+-------+
| wlf   |
| tom   |
| jim   |
| harry |
| salay |
+-------+
5 rows in set (0.00 sec)


#删除
root@localhost[mysh_utf8] 00:26:57>delete from student where name='salay';
Query OK, 1 row affected (0.01 sec)

root@localhost[mysh_utf8] 00:27:03>delete from student where name='jim';
Query OK, 1 row affected (0.00 sec)

```



4、查看现在binlog位置 

```bash
root@localhost[mysh_utf8] 00:27:26>show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000015 |     7276 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

如果要恢复那条删除的数据，那么应该知道截至到删除前的那一刻所在的位置是多少，而这个位置必然位于位置1（即4985）和2（即7276）之间：



```bash
root@localhost[mysh_utf8] 00:28:35>root@localhost[mysh_utf8] 00:28:35>show binlog events in 'mysql-bin.000015' from 4985;
+------------------+------+----------------+-----------+-------------+----------------------------------------------------------------------+
| Log_name         | Pos  | Event_type     | Server_id | End_log_pos | Info                                                                 |
+------------------+------+----------------+-----------+-------------+----------------------------------------------------------------------+
| mysql-bin.000015 | 4985 | Anonymous_Gtid |       111 |        5050 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 5050 | Query          |       111 |        5180 | use `mysh_utf8`; DROP TABLE `student` /* generated by server */      |
| mysql-bin.000015 | 5180 | Anonymous_Gtid |       111 |        5245 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 5245 | Query          |       111 |        5380 | use `mysh_utf8`; create table student(name varchar(50) default null) |
| mysql-bin.000015 | 5380 | Anonymous_Gtid |       111 |        5445 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 5445 | Query          |       111 |        5522 | BEGIN                                                                |
| mysql-bin.000015 | 5522 | Table_map      |       111 |        5579 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 5579 | Write_rows     |       111 |        5619 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 5619 | Xid            |       111 |        5650 | COMMIT /* xid=306 */                                                 |
| mysql-bin.000015 | 5650 | Anonymous_Gtid |       111 |        5715 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 5715 | Query          |       111 |        5792 | BEGIN                                                                |
| mysql-bin.000015 | 5792 | Table_map      |       111 |        5849 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 5849 | Write_rows     |       111 |        5889 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 5889 | Xid            |       111 |        5920 | COMMIT /* xid=307 */                                                 |
| mysql-bin.000015 | 5920 | Anonymous_Gtid |       111 |        5985 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 5985 | Query          |       111 |        6062 | BEGIN                                                                |
| mysql-bin.000015 | 6062 | Table_map      |       111 |        6119 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 6119 | Write_rows     |       111 |        6159 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 6159 | Xid            |       111 |        6190 | COMMIT /* xid=308 */                                                 |
| mysql-bin.000015 | 6190 | Anonymous_Gtid |       111 |        6255 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 6255 | Query          |       111 |        6332 | BEGIN                                                                |
| mysql-bin.000015 | 6332 | Table_map      |       111 |        6389 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 6389 | Write_rows     |       111 |        6431 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 6431 | Xid            |       111 |        6462 | COMMIT /* xid=309 */                                                 |
| mysql-bin.000015 | 6462 | Anonymous_Gtid |       111 |        6527 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 6527 | Query          |       111 |        6604 | BEGIN                                                                |
| mysql-bin.000015 | 6604 | Table_map      |       111 |        6661 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 6661 | Write_rows     |       111 |        6703 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 6703 | Xid            |       111 |        6734 | COMMIT /* xid=310 */                                                 |
| mysql-bin.000015 | 6734 | Anonymous_Gtid |       111 |        6799 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 6799 | Query          |       111 |        6876 | BEGIN                                                                |
| mysql-bin.000015 | 6876 | Table_map      |       111 |        6933 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 6933 | Delete_rows    |       111 |        6975 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 6975 | Xid            |       111 |        7006 | COMMIT /* xid=313 */                                                 |
| mysql-bin.000015 | 7006 | Anonymous_Gtid |       111 |        7071 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                 |
| mysql-bin.000015 | 7071 | Query          |       111 |        7148 | BEGIN                                                                |
| mysql-bin.000015 | 7148 | Table_map      |       111 |        7205 | table_id: 225 (mysh_utf8.student)                                    |
| mysql-bin.000015 | 7205 | Delete_rows    |       111 |        7245 | table_id: 225 flags: STMT_END_F                                      |
| mysql-bin.000015 | 7245 | Xid            |       111 |        7276 | COMMIT /* xid=314 */                                                 |
+------------------+------+----------------+-----------+-------------+----------------------------------------------------------------------+
39 rows in set (0.00 sec)
```

可以看出5245是建表，5380-6703是插入记录，6734-7245是删除记录

已经知道增量日志用于恢复删除操作的截至位置了，很明显就是6734



5、导出列表和sql

```bash
#从起始位置开始导
[root@localhost binlog]# mysqlbinlog --no-defaults --base64-output=decode-rows -v -v --start-position=6734 mysql-bin.000015| grep -B 15 -A 15 'DELETE FROM' >delete.txt

#格式化为sql
[root@localhost binlog]# cat delete.txt | sed -n '/###/p' | sed 's/### //g;s/\/\*.*/,/g;s/DELETE FROM/INSERT INTO/g;s/WHERE/SELECT/g;' | sed -r 's/(@4.*),/\1;/g' | sed 's/@[1-9]=//g' > source.sql

#修改sql，将尾号的,改成;
[root@localhost binlog]# cat source.sql 
INSERT INTO `mysh_utf8`.`student`
SELECT       #如果是SET改成SELECT
  'salay' ,  #，改成；
INSERT INTO `mysh_utf8`.`student`
SELECT
  'jim' ,    #，改成；
```



6、导回sql恢复数据

```bash
root@localhost[mysh_utf8] 00:37:41>select * from student;
+-------+
| name  |
+-------+
| wlf   |
| tom   |
| harry |
+-------+
3 rows in set (0.00 sec)

root@localhost[mysh_utf8] 00:37:51>source /usr/local/mysql/binlog/source.sql;
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

root@localhost[mysh_utf8] 00:38:21>select * from student;
+-------+
| name  |
+-------+
| wlf   |
| tom   |
| harry |
| salay |
| jim   |
+-------+
5 rows in set (0.00 sec)
```

