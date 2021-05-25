## 一、需求说明

某个项目由于更新需要，开发给了新的数据库sql结构语句，需要更新生产上的数据库结构，但是数据记录需要保持



## 二、操作步骤

2.1、导出生产上的sql记录（含数据）

```bash
mysqldump -u root --password=xxxxxx adis>adis_old.sql
```



2.2、在新的测试库上创建2个库，分别导入生产库sql和更新的sql

```bash
# 创建和导入生产库
sql>create database adis_old;
sql>use adis_old;
sql>source adis_old.sql;


# 创建和导入新库
sql>create database adis;
sql>use adis;
sql>source adis.sql;
```



2.3、使用客户端工具对比2套库的差异

点击工具-结构同步

![企业微信截图_20210525194831.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gquwyspahjj61h60sy3z602.jpg)



复制出对比的sql语句

![企业微信截图_20210525195312.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gqux3ivrplj61gu0qngmf02.jpg)

可以先在测试的生产库环境中执行以上语句查看执行结果





2.4、确认无误后，在生产库上执行以上sql语句，同步结果