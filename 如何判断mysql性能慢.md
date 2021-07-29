## 一、问题描述

业务反馈数据库访问慢



## 二、排查方法

### 2.1、执行top命令

CPU使用繁忙

![企业微信截图_20210729165315.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gsxx6dp6fjj60k20ae46y02.jpg)



### 2.2、执行status命令查看慢查询

![企业微信截图_20210729165619.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gsxx9i5c98j60yf0b5wja02.jpg)

Slow queries这一项的值，如果多次查看的值大于0的话,说明有些查询sql命令执行时间过长



### 2.3、执行以下命令查看sql

```bash
sql> show full processlist;  # 没有full参数的命令，显示sql不全
```

可以查询到具体的sql语句



比如这里

```bash
SELECT c.content_id, c.content_name, c.content_code, c.content_order,
        c.word_type, c.image_url,
        CASE WHEN COUNT( p2.practise_id ) > 0 THEN 1 else 0  END AS 'is_practised' 
        FROM edu_content_info c
        LEFT JOIN edu_user_practise_info p1 ON p1.content_id = c.content_id 
        LEFT JOIN edu_user_practise_info p2 ON p1.content_code = p2.content_code AND p2.user_id = 840673
        where c.lesson_id=133
         
        group by c.content_id
```



### 2.4、确认sql是否有走索引

```bash
explain   #追加这一行
SELECT c.content_id, c.content_name, c.content_code, c.content_order,
        c.word_type, c.image_url,
        CASE WHEN COUNT( p2.practise_id ) > 0 THEN 1 else 0  END AS 'is_practised' 
        FROM edu_content_info c
        LEFT JOIN edu_user_practise_info p1 ON p1.content_id = c.content_id 
        LEFT JOIN edu_user_practise_info p2 ON p1.content_code = p2.content_code AND p2.user_id = 840673
        where c.lesson_id=133
         
        group by c.content_id
```

![企业微信截图_20210729170117.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gsxxenocmbj616005sqb802.jpg)

如果是all，说明走的是全表，没有索引，所以慢



## 三、解决办法

### 3.1、临时解决办法

kill sql

先使用以下命令查出慢sql

```bash
mysql>show full processlit; 
```

再根据sid执行kill 

![企业微信截图_20210729170402.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gsxxhju0yqj60o60m11bm02.jpg)



### 3.2、优化sql