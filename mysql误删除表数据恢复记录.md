## 一、现象描述

用户对某个表做了误删除，需要恢复数据

mysql启用了binlog



表记录如下:

```mysql
mysql> desc student_info;
+-----------------+--------------+------+-----+---------+----------------+
| Field           | Type         | Null | Key | Default | Extra          |
+-----------------+--------------+------+-----+---------+----------------+
| work_student_id | int(11)      | NO   | PRI | NULL    | auto_increment |
| work_class_id   | int(11)      | YES  | MUL | NULL    |                |
| user_id         | int(11)      | YES  | MUL | NULL    |                |
| user_name       | varchar(128) | YES  |     | NULL    |                |
| is_submitted    | tinyint(1)   | YES  |     | 0       |                |
| submit_type     | tinyint(1)   | YES  |     | NULL    |                |
| submit_time     | timestamp    | YES  |     | NULL    |                |
+-----------------+--------------+------+-----+---------+----------------+
7 rows in set (0.00 sec)

# 其中 submit_time  列是时间戳格式
```







## 二、恢复思路

由于知道具体的删除时间，使用mysqlbinlog从binlog数据中根据delete关键字恢复记录





## 三、操作记录

### 3.1、从binlog中恢复sql

```bash
mysqlbinlog --no-defaults --base64-output=decode-rows -v -v --start-datetime="2022-10-02 10:00:00"  sz-bin.001575>/tmp/del.txt
```



### 3.2、过滤delete参数

```bash
 grep -A 9 -i "DELETE FROM \`edupen\`.\`student_info\`" del.txt > 111.txt
 
 过滤的内容如下：
 ### DELETE FROM `edupen`.`student_info`
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=330586 /* INT meta=0 nullable=1 is_null=0 */
###   @3=968630 /* INT meta=0 nullable=1 is_null=0 */
###   @4='123' /* VARSTRING(512) meta=512 nullable=1 is_null=0 */
###   @5=1 /* TINYINT meta=0 nullable=1 is_null=0 */
###   @6=2 /* TINYINT meta=0 nullable=1 is_null=0 */
###   @7=1645177158 /* TIMESTAMP(0) meta=0 nullable=1 is_null=0 */
### DELETE FROM `edupen`.`student_info`
### WHERE
###   @1=129290 /* INT meta=0 nullable=0 is_null=0 */
###   @2=4984 /* INT meta=0 nullable=1 is_null=0 */
###   @3=2629053 /* INT meta=0 nullable=1 is_null=0 */
###   @4='陈曹轩' /* VARSTRING(512) meta=512 nullable=1 is_null=0 */
###   @5=1 /* TINYINT meta=0 nullable=1 is_null=0 */
###   @6=2 /* TINYINT meta=0 nullable=1 is_null=0 */
###   @7=1654680087 /* TIMESTAMP(0) meta=0 nullable=1 is_null=0 */
### DELETE FROM `edupen`.`student_info`

其中@7是时间戳
```



### 3.3、修改字段，将delete语句改成insert

```bash
cat 111.txt | sed -n '/###/p' | sed 's/### //g;s/\/\*.*/,/g;s/DELETE FROM/INSERT INTO/g;s/WHERE/VALUES (/g;' | sed -r 's/(@7.*),/\1);/g' | sed 's/@[1-9]=//g' > 13.sql
```



### 3.4、导入

```bash
开发将submit_time 字段改成长字符
再导入
source 13.sql
```



### 3.5、开发将submit_time 字段再转换成时间戳

