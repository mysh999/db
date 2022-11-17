```
# cat dump.sh 
vdate=$(date '+%Y%m%d%H%M%S' )
s=.sql
n=dumpxxxxxxxxxxx
namea=${n}${vdate}${s}
mysqldump -uroot -pxxxxxxxx -h xxxxxxxxxxxxx --set-gtid-purged=off --single-transaction --databases xxxxxxxxxxxx > /data/mysqldb_backup/backupdb/$namea

mysql -uroot -pxxxxxxxxx -h xxxxxxxxxxx xxxxxxxxxxx < /data/mysqldb_backup/backupdb/$namea
```