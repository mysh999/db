<span style='color:red;'>1、mysql主从结构如何判断是否有延迟</span>

执行命令

```sql
mysql> show slave status\G;
```



a) 先比对

 Master_Log_File: sz-bin.000558

和

 Relay_Master_Log_File: sz-bin.000558

数值是否有差异



b) 如果以上数值一样的话，再比对

Exec_Master_Log_Pos: 729638667

Read_Master_Log_Pos: 729638667

这2个值是否有差异，对比SQL线程比IO线程慢了多少个binlog事件

如果这两个数值不一样，说明延迟较大

需要从MASTER上取得binlog status，判断当前的binlog和MASTER上的差距；



c) 查看Seconds_Behind_Master: 0

值是否是0

衡量主备之间的复制延迟



<span style='color:red;'>2、主从架构</span>

主可读写，从只读

从库配置my.cnf加上

read-only = 1

表示只读

但是对root用户除外，root在从库上也可以读写



<span style='color:red;'>3、关于业务用户复制情况</span>

如果复制排除了mysql用户

在从库上如果业务需要用到用户，就需要重建业务用户