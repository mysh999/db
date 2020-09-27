1、master上执行命令

```bash
mysql> show master status\G;
*************************** 1. row ***************************
           File: sz-bin.000748  #主上查询的bin log id
         Position: 755070523
     Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql,information_schema,performance_schema,test
Executed_Gtid_Set: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```



在主上查询配置的从库主机信息，其中repl是复制账号，该命令等同  SHOW PROCESSLIST \G;

![企业微信截图_20200927110631.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gj515urqtmj30xm05fq35.jpg)





2、从库查询信息

从库执行以下命令，查询同步状态

![微信截图_20200927110911.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gj518hi308j30is0lydgi.jpg)

如像以上所述，说明同步正常