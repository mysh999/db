## 一、什么是clickhouse

ClickHouse 是一个开源的面向列的 DBMS（由 Yandex 开发）。ClickHouse 的工作速度比传统方法快 100-1000 倍。它适用于大数据、业务分析和时间序列数据。ClickHouse 是第一个与 Sybase IQ、Vertica 和 Snowflake 等专有数据库的性能、成熟度和可扩展性相匹配的开源 SQL 数据仓库。





## 二、部署过程

2.1、docker-compose文件

```bash
# vi docker-compose.yml 
version: '3'
 
services:
  ch_server:
    image: yandex/clickhouse-server:latest
    restart: always
    ports:
      - "8123:8123"
      - "9990:9000"
      - "9009:9009"
    volumes:
      - ./db:/var/lib/clickhouse
      - ./logs:/var/log/clickhouse-server
    networks:
      - traefik
networks:
  traefik:
    external: true
```



2.2、启动

```bash
# docker-compose up -d
```



2.3、拷贝容器配置文件

```bash
# docker cp 容器名:/etc/clickhouse-server/config.xml .
    
# docker cp 容器名:/etc/clickhouse-server/uses.xml .
```



2.4. 修改配置文件

```bash
# vi config.xml 
<!-- <listen_host>::</listen_host> -->
改成
<listen_host>::</listen_host> 
```



2.5、生成密码（容器里执行）

```bash
$ PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
#记录密码和密文

```



2.6、修改user文件

```bash
# vi users.xml 
---
    <!-- Users and ACL. -->
    <users>


        <halobug>
            <password_sha256_hex>ba5c4f17968e1c21b44ebe7415b733430104192759c0168224401a99e7a8dfd2</password_sha256_hex>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
            <allow_databases>
                // 自定义对应数据库 tutorial 为测试库
               <database>test</database>
            </allow_databases>
        </halobug>
```



2.7、关闭容器，添加配置文件

```bash
# docker-compose down

# cat docker-compose.yml 
version: '3'
 
services:
  ch_server:
    image: yandex/clickhouse-server:latest
    restart: always
    ports:
      - "8123:8123"
      - "9990:9000"
      - "9009:9009"
    volumes:
      - ./db:/var/lib/clickhouse
      - ./config.xml:/etc/clickhouse-server/config.d/config.xml:rw
      - ./users.xml:/etc/clickhouse-server/users.xml:rw
      - ./logs:/var/log/clickhouse-server
    networks:
      - traefik
networks:
  traefik:
    external: true
    
 
 # docker-compose up -d

```



2.8、创建数据库和表

```bash
# docker exec -it c430b /bin/bash
root@c430b61a03db:/# clickhouse-client 
ClickHouse client version 21.12.3.32 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 21.12.3 revision 54452.

c430b61a03db :) create database test;

c430b61a03db :) 
CREATE TABLE test.test_son ( `id` Int64,`name` String, `title` String ) ENGINE = MergeTree ORDER BY id SETTINGS index_granularity = 8192
```



### 三、连接



```bash
# 容器内
# clickhouse-client -h 127.0.0.1 -u halobug --password JwuLbVSc
ClickHouse client version 21.12.3.32 (official build).
Connecting to 127.0.0.1:9000 as user halobug.
Connected to ClickHouse server version 21.12.3 revision 54452.

c430b61a03db :) show databases;

SHOW DATABASES

Query id: 158b07c9-6ea8-4501-bf31-16ee48014bd0

┌─name─┐
│ test │
└──────┘

1 rows in set. Elapsed: 0.003 sec. 

c430b61a03db :) 

# 容器外
或者
ecognized option '-p'. (UNRECOGNIZED_ARGUMENTS)
[root@bigdata-304 clickhouse]#  clickhouse-client -h 127.0.0.1 -u halobug --password JwuLbVSc --port 9990
ClickHouse client version 21.12.3.32 (official build).
Connecting to 127.0.0.1:9990 as user halobug.
Connected to ClickHouse server version 21.12.3 revision 54452.

c430b61a03db :) show databases;

SHOW DATABASES

Query id: 7856022d-c115-4e28-98c9-a72f0d23a1a6

┌─name─┐
│ test │
└──────┘

1 rows in set. Elapsed: 0.003 sec. 

```

