## 问题一：安装RAC 时，执行root.sh脚本，ohasd进程无法正常启动，提示如下：

```bash
ohasd failed to start

Failed to start the Clusterware. Last 20 lines of the alert log follow:

2018-04-19 09:54:30.897:

[client(19244)]CRS-2101:The OLR was formatted using version 3.

alert ：

Oracle High Availability Service has timed out waiting for init.ohasd to be started.
```



原因：

因为 Oracle Linux 7( 和 Redhat 7) 使用 systemd 而不是 initd 来启动 / 重新启动进程，并将它们作为服务运行，所以当前的 11.2.0.4 和 12.1.0.1 的软件安装不会成功，因为 ohasd 进程没有正常启动。



- 解决办法：

手动在 systemd 中添加 ohasd 服务

1) 创建一个空服务文件

```bash
# touch /usr/lib/systemd/system/ohasd.service
```



2) 编辑文件 ohasd.service

```bash
# vi /usr/lib/systemd/system/ohasd.service

[Unit]

Description=Oracle High Availability Services

After=syslog.target

[Service]

ExecStart=/etc/init.d/init.ohasd run >/dev/null 2>&1 Type=simple

Restart=always

[Install]

WantedBy=multi-user.target
```



3)  添加和启动服务

```bash
# systemctl daemon-reload

# systemctl enable ohasd.service

# systemctl start ohasd.service

# systemctl status ohasd.service
```



4) 重新执行root.sh脚本

为了避免其余节点遇到这种报错，可以在 root.sh 执行过程中，待 /etc/init.d/ 目录下生成了 init.ohasd 文件后执行 systemctl start ohasd.service 启动 ohasd 服务即可



5) 在节点2上也执行步骤1-4





## 问题二：安装 database 软件时候报错

报错如下：

```bash
Error in invoking target 'agent nmhs' of makefile '/u01/app/oracle/product/11.2.0/db_1/sysman/lib/ins_emagent.mk'
```



解决办法：

```bash
# vi /u01/app/oracle/product/11.2.0/db_1/sysman/lib/ins_emagent.mk
找到 $(MK_EMAGENT_NMECTL) 这一行，在后面添加 -lnnz11 如下：

复制代码

$(MK_EMAGENT_NMECTL) -lnnz11

然后点击 retry 即可
```





## 问题三：图形安装Oracle弹窗只显示竖线

解决办法：

修改显示分辨率





## 问题四：安装显示乱码

执行

```bash
# export LANG=en_US
```



