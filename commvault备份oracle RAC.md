## 一、环境描述

CV版本:V11 SP18   客户端版本：V11 SP10

备份服务器和介质服务器是一台

RAC： Oracle 11g 204   有2个节点，部署了2套数据库，分别是sdzy和oeedb，归档日志分别存放在本地

```bash
# crs_stat -t
Name           Type           Target    State     Host        
------------------------------------------------------------
ora.DATA.dg    ora....up.type ONLINE    ONLINE    rac1        
ora....ER.lsnr ora....er.type ONLINE    ONLINE    rac1        
ora....N1.lsnr ora....er.type ONLINE    ONLINE    rac1        
ora....VOTE.dg ora....up.type ONLINE    ONLINE    rac1        
ora.asm        ora.asm.type   ONLINE    ONLINE    rac1        
ora.cvu        ora.cvu.type   ONLINE    ONLINE    rac1        
ora.gsd        ora.gsd.type   OFFLINE   OFFLINE               
ora....network ora....rk.type ONLINE    ONLINE    rac1        
ora.oc4j       ora.oc4j.type  ONLINE    ONLINE    rac1        
ora.oeedb.db   ora....se.type ONLINE    ONLINE    rac1        
ora.ons        ora.ons.type   ONLINE    ONLINE    rac1        
ora....SM1.asm application    ONLINE    ONLINE    rac1        
ora....C1.lsnr application    ONLINE    ONLINE    rac1        
ora.rac1.gsd   application    OFFLINE   OFFLINE               
ora.rac1.ons   application    ONLINE    ONLINE    rac1        
ora.rac1.vip   ora....t1.type ONLINE    ONLINE    rac1        
ora....SM2.asm application    ONLINE    ONLINE    rac2        
ora....C2.lsnr application    ONLINE    ONLINE    rac2        
ora.rac2.gsd   application    OFFLINE   OFFLINE               
ora.rac2.ons   application    ONLINE    ONLINE    rac2        
ora.rac2.vip   ora....t1.type ONLINE    ONLINE    rac2        
ora.scan1.vip  ora....ip.type ONLINE    ONLINE    rac1        
ora.sdzy.db    ora....se.type ONLINE    ONLINE    rac1     

# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.188.201   rac1.xl.com  rac1
192.168.188.202   rac2.xl.com rac2
192.168.188.203  rac1-vip.xl.com rac1-vip
192.168.188.204  rac2-vip.xl.com rac2-vip
192.168.200.201 rac1-priv.xl.com rac1-priv
192.168.200.202 rac2-priv.xl.com rac2-priv
192.168.188.205 rac-scan.xl.com rac-scan
192.168.188.100 cs01
```



## 二、准备条件

2.1、配置hosts文件

```bash
192.168.188.201   rac1.xl.com  rac1
192.168.188.202   rac2.xl.com rac2
192.168.188.100 cs01
```



2.2、备份服务器、介质服务器（如有）、客户端开通8400-8420端口互通

1521端口不需要开通



## 三、客户端安装

2个节点同样安装，这里以节点1为例

执行命令

```bash
# ./cvpkgadd 
```

![2020-12-23 00-48-48屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx47mmqr0j30qn07a0tg.jpg)

![2020-12-23 00-49-35屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx48jrpn6j30qm0hjmyr.jpg)![2020-12-23 00-50-27屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx49drdjij30ut0cbjsw.jpg)





![2020-12-23 00-51-13屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4a3kqwoj30sj0bpt9j.jpg)

![2020-12-23 00-51-41屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4alzvwpj30rp08ejrg.jpg)

![2020-12-23 00-52-11屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4b2kcvej30sl07r748.jpg)

![2020-12-23 00-53-45屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4cqkky2j30td09j3yu.jpg)



服务端刷新列表，如图所示：

![2020-12-23 01-01-31屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4ktealwj30bx08xmxe.jpg)



## 四、备份服务器配置

4.1、创建RAC客户端

![2020-12-23 01-02-55屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4mcfke8j30la0e4t9b.jpg)



4.2、先对oeedb库进行配置

![2020-12-23 01-04-38屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4o11wtoj30ez0aut8m.jpg)

数据库名填对应的数据库名称



4.3、配置实例信息

![2020-12-23 01-06-16屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4prlgodj30ev0cjdfs.jpg)





使用以下命令确定连接字符信息

```bash
$ sqlplus "sys/oracle@rac1.xl.com:1521/oeedb as sysdba" 

SQL*Plus: Release 11.2.0.4.0 Production on Wed Dec 23 01:09:00 2020

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,
Data Mining and Real Application Testing options

SQL> 
```

![2020-12-23 01-09-41屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4ta0bt1j30du089t8m.jpg)

![2020-12-23 01-10-33屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4u6gs1ej30e208it8m.jpg)



右键属性

![2020-12-23 01-13-11屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4x9unrdj306s05kq2w.jpg)

查看实例详细信息，是打开状态

![2020-12-23 01-14-00屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4xsh1gqj30fl0f8aa2.jpg)

采用同等方法对另一套库进行配置





## 五、备份配置

5.1、数据库备份

![2020-12-23 01-16-09屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx4zzzixgj30h10eqdfx.jpg)

![2020-12-23 01-17-00屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx50w4vh1j30g60fg0su.jpg)



5.2、日志备份

![2020-12-23 01-18-44屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx52ojanej30go0ik74l.jpg)

![2020-12-23 01-19-19屏幕截图.png](http://ww1.sinaimg.cn/large/007Xg1efgy1glx53bc7nfj30ga0hv0t3.jpg)