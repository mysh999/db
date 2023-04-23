```bash
create user 'cloudcanal'@'%' identified by 'cloudcanal'; 
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'cloudcanal'@'%';
UPDATE mysql.user SET Grant_priv='Y', Super_priv='Y' WHERE User='cloudcanal';
flush privileges;
```

