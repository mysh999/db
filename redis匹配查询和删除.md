1、匹配查询key是alpha1e_push_already_的记录
```
> keys *alpha1e_push_already_*
1) "\xac\xed\x00\x05t\x00 alpha1e_push_already_2020-11-05_"
```


2、删除对应key的记录
```
> del "\xac\xed\x00\x05t\x00 alpha1e_push_already_2020-11-05_"
(integer) 1
```

3、进入其他ID的库
```
> select 1
OK
> select 2
OK
```