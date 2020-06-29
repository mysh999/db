## 一、需求

在sysmanage.t_sys_app库表上创建view，列取与ubxdb.ybx_app_config库表相同列，再把ubxdb.ybx_app_config重命名，再将view名称改成ubxdb.ybx_app_config，应用平滑连接ubxdb.ybx_app_config



## 二、环境信息

```my
mysql> select count(*) from sysmanage.t_sys_app;
+----------+
| count(*) |
+----------+
|       63 |
+----------+
1 row in set (0.00 sec)

mysql> select count(*) from ubxdb.ybx_app_config;
+----------+
| count(*) |
+----------+
|      271 |
+----------+
1 row in set (0.00 sec)

mysql> desc sysmanage.t_sys_app;
+-----------------------+--------------+------+-----+-------------------+-------+
| Field                 | Type         | Null | Key | Default           | Extra |
+-----------------------+--------------+------+-----+-------------------+-------+
| app_id                | varchar(16)  | NO   | PRI | NULL              |       |
| secret_key            | varchar(32)  | NO   |     | NULL              |       |
| app_name              | varchar(64)  | NO   |     | NULL              |       |
| product_id            | varchar(16)  | NO   |     | NULL              |       |
| os_type               | varchar(16)  | NO   |     | NULL              |       |
| client_type           | varchar(16)  | NO   |     | NULL              |       |
| sms_platform          | varchar(16)  | NO   |     |                   |       |
| is_multiple_login     | varchar(1)   | NO   |     | Y                 |       |
| is_account_separation | varchar(1)   | NO   |     | Y                 |       |
| activation_type       | varchar(32)  | NO   |     |                   |       |
| filter_deviceid       | varchar(8)   | NO   |     |                   |       |
| remark                | varchar(255) | NO   |     |                   |       |
| is_valid              | varchar(1)   | NO   |     | Y                 |       |
| create_time           | datetime     | NO   |     | CURRENT_TIMESTAMP |       |
| update_time           | datetime     | YES  |     | NULL              |       |
| is_deleted            | tinyint(4)   | NO   |     | 0                 |       |
+-----------------------+--------------+------+-----+-------------------+-------+
16 rows in set (0.00 sec)

mysql> desc ubxdb.ybx_app_config;
+-------------------------+--------------+------+-----+-------------------+-----------------------------+
| Field                   | Type         | Null | Key | Default           | Extra                       |
+-------------------------+--------------+------+-----+-------------------+-----------------------------+
| id                      | int(11)      | NO   | PRI | NULL              | auto_increment              |
| product_name            | varchar(64)  | YES  |     | NULL              |                             |
| app_id                  | int(11)      | NO   | UNI | NULL              |                             |
| secret_key              | varchar(50)  | NO   |     |                   |                             |
| app_name                | varchar(100) | NO   |     |                   |                             |
| device                  | varchar(1)   | YES  |     | NULL              |                             |
| client_type             | int(1)       | YES  |     | NULL              |                             |
| multiple_login          | int(11)      | NO   |     | 0                 |                             |
| create_time             | datetime     | YES  |     | NULL              |                             |
| modify_time             | timestamp    | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| is_deleted              | int(11)      | NO   |     | 0                 |                             |
| remark                  | varchar(255) | YES  |     |                   |                             |
| sms_platform            | tinyint(1)   | YES  |     | 1                 |                             |
| product_no              | int(8)       | YES  |     | NULL              |                             |
| is_account_separation   | tinyint(4)   | YES  |     | 0                 |                             |
| is_update_of_third_info | tinyint(1)   | YES  |     | 0                 |                             |
| is_ignore_old_data      | tinyint(1)   | YES  |     | 0                 |                             |
| custom_expired_time     | bigint(20)   | YES  |     | NULL              |                             |
+-------------------------+--------------+------+-----+-------------------+-----------------------------+
18 rows in set (0.00 sec)
```



## 三、操作步骤

3.1、备份库表



3.2、重命名ubxdb.ybx_app_config

```my
alter table ubxdb.ybx_app_config rename to ubxdb.old_ybx_app_config;
```



3.3、创建视图

```bash
create view ubxdb.ybx_app_config (app_id,secret_key,app_name,client_type,create_time,is_deleted,remark,sms_platform,is_account_separation,modify_time) as select app_id,secret_key,app_name,client_type,create_time,is_deleted,remark,sms_platform,is_account_separation,update_time from sysmanage.t_sys_app;
```



3.4、查询数据

```my
mysql> select count(*) from ybx_app_config;
+----------+
| count(*) |
+----------+
|       63 |
+----------+
```

