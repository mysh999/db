## 一、问题描述

业务工程在运行的时候，报以下redis错误：

```bash
MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error.
```





## 二、原因分析

过量使用内存设置为0！在低内存环境下，后台保存可能失败





## 三、解决办法

方法1：

修改redis.conf

将

```bash
stop-writes-on-bgsave-error yes
改为stop-writes-on-bgsave-error no
```

再重启redis



方法2：

在redis命令行中执行

```bash
config set stop-writes-on-bgsave-error no
```

