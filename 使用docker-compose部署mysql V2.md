

```bash
#cat ./docker-compose.yml 

version: '3'                            #版本3语法
services:
  mysql-5.7.17:                         #定义服务名
   build:
      context: ./
      dockerfile: ./Dockerfile
   container_name: mysql_prod           #定义容器名称
   environment:                         #设置环境变量
    MYSQL_DATABASE: test                #新建库
    MYSQL_ROOT_PASSWORD: 123456         #root初始密码
	MYSQL_DATABASE: proddb
    MYSQL_ROOT_HOST: '%'                #允许来自运行容器的主机连接
    TZ: Asia/Shanghai                   #设置时区
   ports:                               #端口映射
   - "3306:3306"
   volumes:                             #卷映射
   - ./data:/var/lib/mysql
   restart: always                      #当docker重启时容器自动启动
   
# cat ./Dockerfile 
FROM mysql:5.7.17
MAINTAINER harrison
ADD ./my.cnf /etc/mysql/my.cnf   
   
   

# cat ./my.cnf 
[mysqld]
## 设置server_id，一般设置为IP，注意要唯一
server_id=100  
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql  
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=mysql-bin  
## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M  
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed  
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7  

```

