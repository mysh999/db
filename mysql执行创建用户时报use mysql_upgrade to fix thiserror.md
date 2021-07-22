问题描述：

在创建mysql用户时，报以下提示

```bash
ERROR 1558 (HY000): Column count of mysql.user is wrong. Expected 43, found 42. Created with MySQL 50568, now running 50645. Please use mysql_upgrade to fix this error.
```





解决办法：

```bash
# mysql_upgrade -u root -p
```

