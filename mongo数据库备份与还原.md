1、进入mongo命令行

```bash
# mongo 
如果有自定义端口，指定端口启动

# mongo --port 21777
MongoDB shell version v3.4.2
connecting to: mongodb://127.0.0.1:21777/
MongoDB server version: 3.4.2
> 

```



2、查看库

```bash
> use admin;  #切换到admin下
switched to db admin
> db.auth('root','xxxxxxxxxxx');  #通过认证
1
> show dbs
admin            0.000GB
cbis             0.000GB
local            0.000GB

```



3、备份库

备份所有库

```bash
# mongodump -h 127.0.0.1 --port 21777 -u root -p XXXXXXXXX  -o ./mongodump
```



备份单个库

```bash
# mongodump -h 127.0.0.1 --port 21777 -u cbis -p 'xxxxxxxxxxxx'  -d cbis -o ./test/
```



4、恢复数据库

恢复所有数据库

```bash
# mongorestore /home/zhangy/mongodb/  #这里的路径是所有库的备份路径
```



恢复指定数据库

```bash
# mongorestore -d tank /home/zhangy/mongodb/tank/  #tank这个数据库的备份路径
```

