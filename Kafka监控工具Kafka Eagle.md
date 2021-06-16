# Kafka强大监控工具Kafka Eagle

kafka投入生产使用后，需要借助一些管理（监控）工具。目前这类工具有很多种，具体如下表：

| **监控工具**         | **特点**                                                     | **备注**                                                     |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Kafka Web Console    | 实现brokers、topic列表等监控，展示生产和消费流量图。         | 存在bug，会与生产者、消费者和zookeeper建立大量的连接，导致网络阻塞。 |
| Kafka Manager        | 实现broker级常见的jmx监控，可以对consumer消费进度进行监控，可以web对多个集群进行管理。 | 编译安装耗时，不能设置访问控制，不能配置告警，非常耗内存。   |
| Kafka Eagle          | 实现broker级常见的jmx监控，可以对consumer消费进度进行监控，可以web对多个集群进行管理。 | 安装简单（二进制包解压即用）， 可以配置告警（钉钉、微信、email均可），需要数据库（mysql或sqlite）。 |
| Kafka Offset Monitor | 如果场景是偏重集群管理，则不要选择                           | 该项目已经近2年未维护。                                      |
| JmxTool              | 结合Influxdb和Grafana使用                                    | 比较繁琐                                                     |



我们这里选择Kafka Eagle



#### 1、基础环境准备

```bash
时区调整，时间校准：
# date -R
# timedatectl set-timezone Asia/Shanghai
# yum -y install ntp
# ntpdate ntp1.aliyun.com

安装wget：
# yum install -y wget

关闭selinux：
# vi /etc/sysconfig/selinux
SELINUX=disabled

重启服务器并验证：
# getenforce
Disabled

设置防火墙
[root@node1 ~]# firewall-cmd --zone=public --add-port=8048/tcp --permanent
success
[root@node1 ~]# firewall-cmd --reload
success

设置句柄数等
```



#### 2、安装配置

##### 2.1、安装JDK

```bash
[root@node1 opt]# yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel

[root@node1 opt]# java -version
openjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)

[root@node1 opt]# vim /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

[root@node1 opt]# source /etc/profile
```

##### 2.2、Mysql安装

备注：eagle默认使用sqlite存储，我们这里改成mysql存储

```bash
[root@node1 ~]# yum -y install wget lrzsz net-tools
[root@node1 ~]# wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
[root@node1 ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm
[root@node1 ~]# yum -y install mysql-community-server
[root@node1 ~]# systemctl start  mysqld.service
[root@node1 ~]# systemctl status mysqld.service
[root@node1 ~]# systemctl enable mysqld.service
[root@node1 ~]# netstat -anpt | grep 3306

查询root临时密码
[root@node1 ~]# grep "password" /var/log/mysqld.log
2019-02-14T22:34:02.620689Z 1 [Note] A temporary password is generated for root@localhost: &5E%TRUAzrDL
[root@node1 ~]# mysql -u root -p

#root临时密码是&5E%TRUAzrDL
修改root密码：
mysql> alter user 'root'@'localhost' identified by 'root新密码';
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

创建数据库eagle：
mysql> create database eagle character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.01 sec)
mysql> use eagle;
Database changed

创建普通用户zabbix，并授权数据库eagle给eagle用户、并允许远程访问：
mysql> create user 'eagle'@'%' identified by 'Mysql@123';
mysql> grant all privileges on eagle.* TO 'eagle'@'%' identified by 'Mysql@123';
mysql> flush privileges;

卸载yum源
[root@node1 ~]# yum -y remove mysql57-community-release-el7-10.noarch
```

##### 2.3、安装Eagle

网络下载路径：https://codeload.github.com/smartloli/kafka-eagle-bin/tar.gz/v2.0.1

本地下载路径：https://gitlab.cndy.org/zhangdong/EFK/-/blob/master/soft/kafka-eagle-bin-2.0.2.tar.gz

```bash
[root@node1 opt]# cd /opt/software/
[root@node1 software]# tar -zxvf kafka-eagle-bin-2.0.2.tar.gz
kafka-eagle-bin-2.0.2/
kafka-eagle-bin-2.0.2/kafka-eagle-web-2.0.2-bin.tar.gz

[root@node1 software]# mv kafka-eagle-bin-2.0.2 /usr/local/kafka-eagle
[root@node1 software]# cd /usr/local/kafka-eagle
[root@node1 kafka-eagle]# tar zxvf kafka-eagle-web-2.0.2bin.tar.gz

[root@node1 opt]# vi /etc/profile
export KE_HOME=/usr/local/kafka-eagle/kafka-eagle-web-2.0.2
export PATH=$PATH:$KE_HOME/bin

[root@node1 opt]# source /etc/profile
```

##### 2.4、编辑Eagle配置文件

```bash
[root@node1 ~]# vim /usr/local/kafka-eagle/kafka-eagle-web-2.0.2/conf/system-config.properties
######################################
# 多个Zookeeper和Kafka集群列表
######################################
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=10.11.112.30:2181,10.11.112.55:2181,10.11.112.48:2181

######################################
# broker size online list
######################################
cluster1.kafka.eagle.broker.size=20

######################################
# zk客户端线程限制
######################################
kafka.zk.limit.size=25

######################################
# kafka eagle web 端口
######################################
kafka.eagle.webui.port=8048

######################################
# kafka jmx acl and ssl authenticate
######################################
cluster1.kafka.eagle.jmx.acl=false
cluster1.kafka.eagle.jmx.user=keadmin
cluster1.kafka.eagle.jmx.password=keadmin123
cluster1.kafka.eagle.jmx.ssl=false
cluster1.kafka.eagle.jmx.truststore.location=/Users/dengjie/workspace/ssl/certificates/kafka.truststore
cluster1.kafka.eagle.jmx.truststore.password=ke123456

######################################
# kafka offset存储
######################################
cluster1.kafka.eagle.offset.storage=kafka

######################################
# kafka 指标数据保存时间，默认15天
######################################
kafka.eagle.metrics.charts=true
kafka.eagle.metrics.retain=30

######################################
# kafka sql 主题最大记录
######################################
kafka.eagle.sql.topic.records.max=5000

######################################
# delete kafka topic token
######################################
kafka.eagle.topic.token=keadmin

#####################################
# 邮件服务器设置，用来告警(目前告警这里有点问题)
#####################################
kafka.eagle.mail.enable=true
kafka.eagle.mail.sa=alert_sa
kafka.eagle.mail.username=自己的邮箱地址
kafka.eagle.mail.password=邮箱授权码
kafka.eagle.mail.server.host=smtp.163.com
kafka.eagle.mail.server.port=465（25端口被关闭）

######################################
# 设置告警用户，多个用户以英文逗号分隔
######################################
kafka.eagle.alert.users=自己的邮箱地址

######################################
# kafka sasl authenticate
######################################
cluster1.kafka.eagle.sasl.enable=false
cluster1.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
cluster1.kafka.eagle.sasl.mechanism=SCRAM-SHA-256
cluster1.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="kafka" password="kafka-eagle";
cluster1.kafka.eagle.sasl.client.id=
cluster1.kafka.eagle.blacklist.topics=
cluster1.kafka.eagle.sasl.cgroup.enable=false
cluster1.kafka.eagle.sasl.cgroup.topics=

######################################
# 配置数据库信息
######################################
kafka.eagle.driver=com.mysql.jdbc.Driver
kafka.eagle.url=jdbc:mysql://ip:3306/eagle
kafka.eagle.username=数据库user
kafka.eagle.password=password

```

##### 2.5、启动Eagle

```bash
[root@node1 ~]# cd /usr/local/kafka-eagle/kafka-eagle-web-2.0.2/bin/
[root@node1 bin]# chmod +x ke.sh
[root@node1 bin]# ke.sh start
……
Welcome to
    __ __    ___     ____    __ __    ___            ______    ___    ______    __     ______
   / //_/   /   |   / __/   / //_/   /   |          / ____/   /   |  / ____/   / /    / ____/
  / ,<     / /| |  / /_    / ,<     / /| |         / __/     / /| | / / __    / /    / __/   
/ /| |   / ___ | / __/   / /| |   / ___ |        / /___    / ___ |/ /_/ /   / /___ / /___   
/_/ |_|  /_/  |_|/_/     /_/ |_|  /_/  |_|       /_____/   /_/  |_|\____/   /_____//_____/   
                                                                                             
Version 2.0.1 -- Copyright 2016-2020
*******************************************************************
* Kafka Eagle Service has started success.
* Welcome, Now you can visit 'http://192.168.146.199:8048'
* Account:admin ,Password:123456
*******************************************************************
* <Usage> ke.sh [start|status|stop|restart|stats] </Usage>
* <Usage> https://www.kafka-eagle.org/ </Usage>
*******************************************************************
```

##### 2.6、访问Eagle

http://192.168.146.199:8048

默认用户名admin，密码123456