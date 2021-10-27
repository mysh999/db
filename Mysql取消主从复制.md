参考：https://blog.csdn.net/mrbuffoon/article/details/105708418





部署环境有时需要更换取消主从机制或者更换备机，需要将之前的主备关系解除，现梳理其一般性流程：

1、slave流程

```bash
首先停止slave
mysql>stop slave;

清除slave信息
mysql>reset slave all;
```





可以通过以下命令查看当前状态

```bash
mysql> show slave status\G

Emptyset (0,00 sec)
```

之后slave可以直接关闭下线。



2、master流程
清除master上主从信息

```bash
mysql> reset master;
```

修改配置重启mysql



如果想彻底清除主从的机制，可以修改配置文件，删除主从相关的配置项，然后重启mysql即可。





————————————————
版权声明：本文为CSDN博主「Mr_buffoon」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/mrbuffoon/article/details/105708418