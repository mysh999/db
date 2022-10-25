## 一、需求说明

 需要查询某张业务表的部分记录，将其导出，再导入到其他库中



## 二、导出记录

执行以下命令导出

```bash
$ mysql  -u root -p  -P 3306 -e "SELECT * FROM  dbname.tablename t where t.serial_number between 'EAA00720000161' and 'EAA00720000292'">/tmp/1.txt

其中dbname表示库名，tablename表示表名
```



## 三、导入到其他库

```bash
$ mysql -uroot -p

mysql> use dbname2;
mysql>  load data local infile "1.txt" into table tablename; 
```

