参考来源：

https://www.facecto.com/archives/680.html



1、下载包

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-5.6.45-1.el7.x86_64.rpm-bundle.tar
```



2、解压

```bash
tar -xvf MySQL-5.6.45-1.el7.x86_64.rpm-bundle.tar
```



3、安装

```bash
rpm -ivh MySQL-client-5.6.45-1.el7.x86_64.rpm
rpm -ivh MySQL-devel-5.6.45-1.el7.x86_64.rpm
rpm -ivh MySQL-server-5.6.45-1.el7.x86_64.rpm
```



4、启动

```bash
systemctl start  mysql
```



5、修改密码

```bash
# 获得密码
cat /root/.mysql_secret

# 登录
mysql -uroot -p'上一步获得的密码'

# 修改密码
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');
```





6、授权

```bash
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123456' WITH GRANT OPTION;
```

