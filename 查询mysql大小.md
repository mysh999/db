1、查询mysql 各个库大小
```
SELECT table_schema,SUM(AVG_ROW_LENGTH*TABLE_ROWS+INDEX_LENGTH)/1024/1024 AS total_mb FROM information_schema.TABLES group by table_schema   order by total_mb desc
```

2、查询所有库大小
```
 select round(sum(data_length/1024/1024),2) as size_mb from  information_schema.tables;
 ```
 
 3、查询指定数据库大小
 其中ubtdev是库名
 ```
 select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data_mb from TABLES where table_schema='ubtdev';
 ```
 
 4、查询所有数据库各容量大小
 ```
 select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
group by table_schema
order by sum(data_length) desc, sum(index_length) desc;
 ```
 
 5、查看指定数据库容量大小
 ```
 select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
where table_schema='ubtdev';
```

6、查看指定库各表容量大小
```
select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
where table_schema='ubtdev'
order by data_length desc, index_length desc;
```