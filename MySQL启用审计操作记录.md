来源：https://juejin.im/post/5cb6f7e5e51d456e5b66ad53



## 一、说明

server_audit是一款内嵌在mariadb的审计插件，在mysql中同样适用，主要用于记录用户操作



## 二、安装

通过show variables like 'plugin_dir';查看你的插件目录，
我的是：/usr/lib64/mysql/plugin/
把下载好的插件server_audit.so
复制到/usr/lib64/mysql/plugin/ 
注意chmod +x server_audit.so
登录mysql执行插件安装命令：
INSTALL PLUGIN server_audit SONAME 'server_audit.so';
插件安装成功后有这些全局变量：show variables like '%audit%';



## 三、配置

```bash
进入mysql 执行：更改全局变量
set global server_audit_excl_users='root';
set global server_audit_events='QUERY_DDL,QUERY_DML';
set global server_audit_file_path ='/mysqllog/';
set global server_audit_file_rotate_size=1073741824;
set global server_audit_file_rotations=10;
set global server_audit_file_rotate_now=ON;
set global server_audit_logging=on;
 
在my.cnf 增加
#audit
server_audit_events='QUERY_DDL,QUERY_DML'
server_audit_logging=on
server_audit_file_path =/mysqllog/
server_audit_file_rotate_size=1G
server_audit_file_rotations=10
server_audit_file_rotate_now=ON
server_audit_excl_users=root
```



## 四、建议关闭general log

set global general_log=off;
在my.cnf注释
general_log_file = /mysqllog/mysql.log
general_log = 1