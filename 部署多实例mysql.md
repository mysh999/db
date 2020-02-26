[TOC]



## 一、目的

在CentOS7平台上部署多个mysql5.7实例

分别使用3306、3307、3308端口



## 二、准备条件

2.1、安装依赖包

```bash
# yum install –y make cmake  bison gcc gcc-c++     ncurses  ncurses-devel
```

2.2、创建mysql用户和组，默认密码mysql

```bash
# groupadd mysql
# useradd -g mysql -s /bin/false -M mysql  //建立属于mysql组的mysql用户，且不允许登录
```

2.3、安装boost
从MySQL 5.7.5开始Boost库是必需的

```bash
# tar -zxvf boost_1_59_0.tar.gz 
# cp -r boost_1_59_0 /usr/local/
# cd /usr/local/boost_1_59_0
# ./bootstrap.sh
# ./b2 install                       --大概10分钟

```



## 三、采用编译方式安装mysql

3.1、解压缩

```bash
# tar -zxvf mysql-5.7.18.tar.gz 
```

3.2、创建软件目录

```bash
# mkdir -p /app/mysql/data
```



3.3、编译安装

```bash
#cmake . -DCMAKE_INSTALL_PREFIX=/app/mysql  -DMYSQL_DATADIR=/app/mysql/data  -DDOWNLOAD_BOOST=1  -DWITH_BOOST=/usr/local/boost_1_59_0  -SYSCONFDIR=/etc  -DWITH_MYISAM_STORAGE_ENGINE=1  -DWITH_INNOBASE_STORAGE_ENGINE=1  -DWITH_MEMORY_STORAGE_ENGINE=1  -DWITH_READLINE=1  -DMYSQL_UNIX_ADDR=/app/mysql/mysqld.sock    -DENABLED_LOCAL_INFILE=1  -DWITH_PARTITION_STORAGE_ENGINE=1  -DEXTRA_CHARSET=all  -DDEFAULT_CHARSET=utf8  -DDEFAULT_COLLATION=utf8_general_ci
```

```bash
#make
#make install
```

3.4、修改权限

```bash
# chown -R mysql:mysql /app/mysql
# chmod u+w /app/mysql
```



3.5、创建多实例的数据文件目录

```bash
# mkdir -p /data/{3306,3307,3308}/data
# mkdir -p /data/{3306,3307,3308}/binlog
# chown -R mysql.mysql /data
```



3.6、改名系统自带的my.cnf

```bash
# mv /etc/my.cnf /etc/my.cnf.bak
```



3.7、创建多实例的配置文件

```bash
# vi /data/3306/my.cnf
[client]
port        = 3306
socket      = /data/3306/mysql3306.sock
 
[mysqld]
user        = mysql
port        = 3306
socket      = /data/3306/mysql3306.sock
basedir = /app/mysql
datadir = /data/3306/data
log-bin = /data/3306/binlog/bin_log
  
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 1024
sort_buffer_size = 4M
net_buffer_length = 8K
read_buffer_size = 4M
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 64M
thread_cache_size = 128
query_cache_size = 128M
tmp_table_size = 128M
explicit_defaults_for_timestamp = true
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535
binlog_format=mixed
server-id   = 3306
expire_logs_days = 7
early-plugin-load = ""
default_storage_engine = InnoDB

[mysqld_safe]
log_error=/data/3306/mysql3306_error.log
pid-file=/data/3306/mysqld.pid




# vi /data/3307/my.cnf 
[client]
port        = 3307
socket      = /data/3307/mysql3307.sock
 
[mysqld]
user        = mysql
port        = 3307
socket      = /data/3307/mysql3307.sock
basedir = /app/mysql
datadir = /data/3307/data
log-bin = /data/3307/binlog/bin_log
  
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 1024
sort_buffer_size = 4M
net_buffer_length = 8K
read_buffer_size = 4M
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 64M
thread_cache_size = 128
query_cache_size = 128M
tmp_table_size = 128M
explicit_defaults_for_timestamp = true
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535
binlog_format=mixed
server-id   = 3307
expire_logs_days = 7
early-plugin-load = ""
default_storage_engine = InnoDB

[mysqld_safe]
log_error=/data/3307/mysql3307_error.log
pid-file=/data/3307/mysqld.pid



# vi /data/3308/my.cnf 
[client]
port        = 3308
socket      = /data/3308/mysql3308.sock
 
[mysqld]
user        = mysql
port        = 3308
socket      = /data/3308/mysql3308.sock
basedir = /app/mysql
datadir = /data/3308/data
log-bin = /data/3308/binlog/bin_log
  
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 1024
sort_buffer_size = 4M
net_buffer_length = 8K
read_buffer_size = 4M
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 64M
thread_cache_size = 128
query_cache_size = 128M
tmp_table_size = 128M
explicit_defaults_for_timestamp = true
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535
binlog_format=mixed
server-id   = 3308
expire_logs_days = 7
early-plugin-load = ""
default_storage_engine = InnoDB

[mysqld_safe]
log_error=/data/3308/mysql3308_error.log
pid-file=/data/3308/mysqld.pid

```



创建日志文件

```bash
# touch /data/3306/mysql3306_error.log
# chown mysql.mysql /data/3306/mysql3306_error.log

# touch /data/3307/mysql3307_error.log
# chown mysql.mysql /data/3307/mysql3307_error.log

# touch /data/3308/mysql3308_error.log
# chown mysql.mysql /data/3308/mysql3308_error.log
```



3.8、初始化表

```bash
# /app/mysql/bin/mysqld --defaults-file=/data/3306/my.cnf --initialize-insecure --user=mysql　

# /app/mysql/bin/mysqld --defaults-file=/data/3307/my.cnf --initialize-insecure --user=mysql　

# /app/mysql/bin/mysqld --defaults-file=/data/3308/my.cnf --initialize-insecure --user=mysql　
```

注： 之前版本mysql_install_db是在mysql_basedir/script下，5.7放在了mysql_install_db/bin目录下,且已被废弃
“–initialize”会生成一个随机密码(~/.mysql_secret)，而”–initialize-insecure”不会生成密码
–datadir目标目录下不能有数据文件



3.9、配置my.cnf文件集中管理多个实例

```bash
# cat /etc/my.cnf
[mysqld_multi]
mysqld=/app/mysql/bin/mysqld_safe
mysqladmin=/app/mysql/bin/mysqladmin

[mysqld1]
port=3306
socket=/data/3306/mysql3306.sock
datadir=/data/3306/data
skip-external-locking
log-bin=/data/3306/binlog/bin_log
server-id=3306
user=mysql

[mysqld2]
port=3307
socket=/data/3307/mysql3307.sock
datadir=/data/3307/data
skip-external-locking
log-bin=/data/3307/binlog/bin_log
server-id=3307
user=mysql

[mysqld3]
port=3308
socket=/data/3308/mysql3308.sock
datadir=/data/3308/data
skip-external-locking
log-bin=/data/3308/binlog/bin_log
server-id=3308
user=mysql


[mysql]
no-auto-rehash

#有多个实例就按以上追加mysqld2、mysqld3...

```



3.10、启动实例

```bash
 # cp  /app/mysql/support-files/mysqld_multi.server /etc/rc.d/init.d/mysqld_multi　　
 # chmod +x /etc/init.d/mysqld_multi
 
 # vi /etc/init.d/mysqld_multi
 #在这个多实例启动脚本顶部添加 
export PATH=$PATH:$HOME/bin:/app/mysql/bin
 
 
 #启动
 # /etc/init.d/mysqld_multi start 1     #1表示mysqld1，如果有多个实例，start 1 2 3 ...
 
 #查看状态
# /etc/init.d/mysqld_multi report 
Reporting MySQL servers
MySQL server from group: mysqld1 is running
MySQL server from group: mysqld2 is running
MySQL server from group: mysqld3 is running
```





3.11、登录mysql

将mysql命令路径追加到path中

```bash
# cat .bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin:/app/mysql/bin

export PATH
```



登录

```bash
# mysql -uroot -S /data/3306/mysql3306.sock
mysql> 

# mysql -uroot -S /data/3307/mysql3307.sock

# mysql -uroot -S /data/3308/mysql3308.sock
```



3.11、修改mysql密码

```bash
# mysqladmin -uroot password '123456' -S /data/3306/mysql3306.sock
```

重新登录

```bash
# mysql -uroot -p -S /data/3306/mysql3306.sock
```



3.12、关闭数据库

```bash
#  mysqladmin shutdown -uroot -p123456 -S /data/3306/mysql3306.sock
#  mysqladmin shutdown -uroot -p123456 -S /data/3307/mysql3307.sock
#  mysqladmin shutdown -uroot -p123456 -S /data/3308/mysql3308.sock

或者
# /etc/init.d/mysqld_multi stop 1-3
```





3.13、添加进service服务管理

```bash
# chkconfig --add mysqld_multi
```



