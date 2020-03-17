## 一、前言

xtrabackup是一个开源免费的对mysql进行备份还原工具，特点就是备份恢复操作速度快，不会打断正在执行的事务（不会锁表），能够基于压缩等功能，自动备份校验，还原速度快



## 二、测试环境描述

- 单机

- Server version: 5.7.18

- innodb引擎



## 三、包准备

```bash
# yum -y install libev
# yum -y install rsync
# yum -y install perl*

# wget -c https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.9/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.9-1.el7.x86_64.rpm

# rpm -ivh percona-xtrabackup-24-2.4.9-1.el7.x86_64.rpm 

# xtrabackup --version
xtrabackup version 2.4.9 based on MySQL server 5.7.13 Linux (x86_64) (revision id: a467167cdd4)
```



## 四、备份

4.1、准备测试数据

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| abiclouddb         |
| cbiatrisdb         |
| mysql              |
| performance_schema |
| sys                |
| ubxdb              |
+--------------------+
7 rows in set (0.00 sec)

mysql> grant select on ubxdb.* to 'reader'@'%' identified by 'reader';

mysql> grant select on abiclouddb.* to 'reader'@'%';

mysql> grant select on cbiatrisdb.* to 'reader'@'%';
```



4.2、全备

```bash
# innobackupex  --defaults-file=/etc/my.cnf --user=root --password='root' --backup /opt/backup
```

备份后的文件

```bash
# tree -d
.
└── 2020-03-17_11-21-24
    ├── abiclouddb
    ├── cbiatrisdb
    ├── mysql
    ├── performance_schema
    ├── sys
    └── ubxdb
```



4.3、模拟删除数据

```vasg

mysql> drop database ubxdb;
Query OK, 1 row affected (0.01 sec)

mysql> drop database cbiatrisdb;
Query OK, 0 rows affected (0.00 sec)

mysql> drop database abiclouddb;
Query OK, 31 rows affected (0.11 sec)
```



4.4、恢复

```bash
# service mysqld stop

# innobackupex --apply-log --use-memory=4G /opt/backup/2020-03-17_11-21-24              # 预备全备,让数据与备份完成时保持一致
把data目录清空，不然会提示目录非空

#innobackupex --defaults-file=/etc/my.cnf --copy-back /opt/backup/2020-03-17_11-21-24  # 指定/etc/my.cnf,使用--copy-back将完备拷贝到datadir


#chown mysql.mysql /data/mysql_data -R

#service mysqld start
```

为了加快还原的速度，可加--use-memory=4G 参数



4.5、增量备份

是在已有全量的基础之上的，不然是没有任何意义的。
当前以/opt/backup/2020-03-17_11-21-24 全量备份为基础 进行增量备份

```bash
# innobackupex  --defaults-file=/etc/my.cnf --incremental /opt/backup/inc --incremental-basedir=/opt/backup/2020-03-17_11-21-24 --user=root --password='root'
```



4.6、增量恢复

```bash
#innobackupex --defaults-file=/etc/my.cnf --apply-log --redo-only /opt/backup/2020-03-17_11-21-24 --use-memory=1G --user=root --password=root

#innobackupex --defaults-file=/etc/my.cnf --apply-log --redo-only /opt/backup/2020-03-17_11-21-24 --incremental-dir=/opt/backup/inc/2020-03-17_13-38-10 --use-memory=1G --user=root --password=root

#停掉mysql
#service mysqld stop


#清空/usr/local/mysql/data/

#innobackupex --defaults-file=/etc/my.cnf --user=root --password=root --copy-back /opt/backup/2020-03-17_11-21-24
#chown -R mysql:mysql /usr/local/mysql/data/
#启动mysql
#service mysqld start
```



##  五、附录备份脚本

```bash
# crontab -l
00 */1 * * * /root/tools/sbin/.backup_all.sh >> /root/tools/logs/backup.log 2>&1




# cat /root/tools/sbin/.backup_all.sh
#!/usr/bin/env bash
DEFAULTSFILE="/etc/my.cnf"
USER="root"
PASS=""
SOCKET="/var/lib/mysql/mysql.sock"
BACKPATH="/data/backup/`hostname`"
REMOTE="10.10.1.12"

function main() {
    echo '-------------------------------------------------------------------------------------------------------------'
    start=`date +"%s"`
    now=`date +"%F-%H"`
    test -d $BACKPATH || mkdir -p $BACKPATH
    if [[ `date +"%H"` == 00 ]]; then
        lastday=`date +"%F" -d "-1days"`
        echo "开始执行清理操作.当前日期 ${now}"
        echo -e "\n\n"
        echo -e '------------------------------------\n\n'
        echo -e "\t需要清理的文件夹如下:"
        cd ${BACKPATH}
        for name in `ls -d ${lastday}-*`; do
            echo $name
            [ $name ] && rm -rf $name
        done
        echo -e '------------------------------------\n\n'
        echo "开始执行完备操作.当前日期 ${now}"
        echo -e "\n\n"
        innobackupex --defaults-file=${DEFAULTSFILE} --backup --user=${USER} --password=${PASS} --socket=${SOCKET} --no-timestamp ${BACKPATH}/${now}
    else
        echo "开始执行增量操作.当前日期 ${now}"
        old=`date +"%F-%H" -d "-1hours"`
        innobackupex --no-timestamp --incremental ${BACKPATH}/${now} --incremental-basedir=${BACKPATH}/${old}
    fi
    echo "完备至${REMOTE}操作.当前日期 ${now}"
    rsync -av --delete $BACKPATH/ $REMOTE:$BACKPATH/
    echo -e "\n\n"
    end=`date +"%s"`
    echo -e "\n\n"
    echo "备份完毕.耗时$(($end-$start)) 秒"
    echo -e "\n\n"
}

main
```

