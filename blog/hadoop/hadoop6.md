#### 集群时间同步


crontab定时任务重启
```
service crond restart
```
![cron](../pic/hadoop/crontab.png)

![cron](../pic/hadoop/crontab2.png)

![cron](../pic/hadoop/crontab3.png)

crontab -e
```
*/1 * * * * /bin/echo "test" >> /opt/module/hadoop-2.7.7/test
```

**为了让不同机器上的任务能按时执行，需要集群时间同步**

![cron](../pic/hadoop/crontab4.png)

1. 时间服务器配置 （root用户）
  - 检查ntp是否安装
  ```
  rpm -qa | grep ntp
  ```
  - 修改ntp配置文件
  ```
  vim /etc/ntp.conf
  修改1：放开注释
  restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
  修改2：注释使用互联网时间
  #server 0.centos.pool.ntp.org iburst
  #server 1.centos.pool.ntp.org iburst
  #server 2.centos.pool.ntp.org iburst
  #server 3.centos.pool.ntp.org iburst
  修改3：当该节点失去网络连接，依然可以使用本地时间作为服务器时间为集群同步时间
  文件末尾添加
  server 127.127.1.0
  fudge 127.127.1.0 stratum 10 (配置精度，共有15级)
  ```
  ![cron](../pic/hadoop/ntp配置.png)

  - 修改/etc/sysconfig/ntpd文件，添加如下内容，让硬件时间和系统时间一致
  ```
  SYNC_HWCLOCK=yes
  ```
  - 重新启动ntpd
  ```
  service ntpd status
  service ntpd start
  ```
  - 设置ntpd开机启动
  ```
  chkconfig ntpd on
  ```
2. 测试

  ![cron](../pic/hadoop/ntpd.png)
