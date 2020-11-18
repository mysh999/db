进入mongo命令行

```
# mongo
```



切换到admin数据库

```bash
> use admin
```



使用root用户登录

```bash
> db.auth('root','ubx')
1
```



查看数据库

```bash
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```



创建新数据库（之前不存在该库）

```bash
> use testdb
switched to db testdb
```



插入数据（新库要插入数据，才能持久化）

```bash
> db.testdb.insert({"name":"你好"})
WriteResult({ "nInserted" : 1 })
```



查询

```bash
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
```



创建用户

```bash
> db.createUser({ user: "user01",pwd: "password01",customData:{name:"read-write-user"},roles:[{ role: "readWrite",db: "testdb" }]})



Successfully added user: {
        "user" : "user01",
        "customData" : {
                "name" : "read-write-user"
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "testdb"
                }
        ]
}



#说明
user文档字段介绍：
    user字段，为新用户的名字；
    pwd字段，用户的密码；
    cusomData字段，为任意内容，例如可以为用户全名介绍；
    roles字段，指定用户的角色，可以用一个空数组给新用户设定空角色；
    在roles字段,可以指定内置角色和用户定义的角色。
    
#角色说明   
 Built-In Roles（内置角色）：
    1. 数据库用户角色：read、readWrite;
    2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
    3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
    4. 备份恢复角色：backup、restore；
    5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
    6. 超级用户角色：root  
    // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
    7. 内部角色：__system

```





验证新用户

```bash

> db.auth('user01','password01')
1

> show dbs
testdb  0.000GB
```



命令行登录mongo

```bash
# mongo -u user01 -p password01 --authenticationDatabase testdb
MongoDB shell version v4.0.9
connecting to: mongodb://127.0.0.1:27017/?authSource=testdb&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("77e5e9fc-399a-44cb-8af7-a7664f1fe516") }
MongoDB server version: 4.0.9
> 
```





集合概念

数据库里面是集合（collection），集合里面是文档

```bash
# 查询数据库下的集合
> show collections
dfs_file
robot_status_Power_Atris
setting_Atris
```



删除集合

```bash
> db.robot_status_Power_Atris.drop()
true
```



