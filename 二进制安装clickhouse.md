## 一、clickhouse说明

ClickHouse是近年来备受关注的开源列式数据库，主要用于数据分析（OLAP）领域



## 二、安装

### 2.1、yum源安装

```ba
yum install yum-utils
rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/stable/x86_64
```





### 2.2、安装

```bash
yum install clickhouse-server clickhouse-client
```



### 2.3、配置

默认安装完只能本地进行访问，要其他服务器能访问，修改

```bash
找到下面的语句，将其注释去掉即可：

 <listen_host>::</listen_host>
```



### 2.4、重启服务

```bash
/etc/init.d/clickhouse-server restart
```



### 2.5、查看端口

```bash
# netstat -ntlp|grep -i 8123
tcp6       0      0 :::8123                 :::*                    LISTEN      2837/clickhouse-ser 
```



## 三、使用

### 3.1、登陆

```bash
#  clickhouse-client 
ClickHouse client version 23.3.1.2823 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 23.3.1 revision 54462.

Warnings:
 * Linux transparent hugepages are set to "always". Check /sys/kernel/mm/transparent_hugepage/enabled

ali-sz-dataplatform-14 :) 

ali-sz-dataplatform-14 :)       show databases;

SHOW DATABASES

Query id: e7c2f422-6e3f-45da-98f0-74614ef0613d

┌─name───────────────┐
│ INFORMATION_SCHEMA │
│ default            │
│ information_schema │
│ system             │
└────────────────────┘

4 rows in set. Elapsed: 0.001 sec
```





### 3.2、默认没有密码，配置

```bash
# vi /etc/clickhouse-server/user.xml
修改
<password>123456789</password>


# 重启服务 
# systemctl restart clickhouse-server

# 登陆
# clickhouse-client --password
```

