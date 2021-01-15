## 一、升级思路

由于mongodb才有的是容器部署

因此才有对旧版本进行备份，再对mongo的image镜像进行替换，最后还原的方式进行



## 二、准备工作

2.1、数据备份

- 查看容器

```bash
# docker ps|grep -i mongo
037afd02e286        mongo:4.0.9                             "docker-entrypoint.s…"   2 weeks ago         Up 2 weeks             0.0.0.0:27017->27017/tcp                                                                                                                                                                                                                                                                                                                         mongo
```



- 登陆容器

  ```bash
  # docker exec -it mongo /bin/bash
  
  # 记录库信息
  # mongo
  MongoDB shell version v4.0.22
  connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
  Implicit session: session { "id" : UUID("61290687-e2a0-4c5f-a772-d67261c03d4a") }
  MongoDB server version: 4.0.22
  > use admin;
  switched to db admin
  > db.auth('root','password');
  1
  > show dbs;
  
  # 备份所有库
  root@037afd02e286:/#   mongodump -h 127.0.0.1 --port 27017 -u root -p password  -o ./mongodump
  
  # 将备份库拷贝出来
  ```

- 关闭旧版本的mongo容器

  ```bash
  # docker-compose down
  ```

  



2.2、准备新image

```bash
# docker pull mongo:4.0.22
```





## 三、升级mongo

3.1、准备新版本的容器部署

说明：可以参考之前的mongo容器写法，只需修改image版本号

```bash
# cat docker-compose.yml 
version: '2.0'
services:
  mongodb:
    container_name: mongo
    image: mongo:4.0.22
    restart: always
    environment:
      - TZ=Asia/Shanghai
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=password
    ports:
      - 27017:27017
    volumes:
      - ./mongo/data:/data/db
      - ./mongo/init:/docker-entrypoint-initdb.d
    networks:
      - default
    logging:
      driver: "json-file"
      options:
        max-size: "1g"
		
networks:
  default:
    external:
      name: cbis
```



3.2、启动容器

```bash
# docker-compose up -d
```



3.3、将备份的数据拷贝到新容器中



3.4、进入到容器中

```bash
# docker exec -it mongo /bin/bash
root@83af8a127c52:/# 

#进行恢复操作
# mongorestore -h 127.0.0.1:27017 -u "root" -p "password" --authenticationDatabase admin ./mongodump/

# 查看库信息，和旧版本做对比
# mongo
MongoDB shell version v4.0.22
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("61290687-e2a0-4c5f-a772-d67261c03d4a") }
MongoDB server version: 4.0.22
> use admin;
switched to db admin
> db.auth('root','password');
1
> show dbs;
```

