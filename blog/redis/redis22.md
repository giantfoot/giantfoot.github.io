#### 哨兵模式

能够后台监控主机是否故障，如果出现了故障，根据投票数自动将从库转换为主库。

哨兵模式以一种特殊的模式，首先redis提供了哨兵的命令，哨兵是个独立的进程，作为进程，他会独立运行，=。

其原理是哨兵通过发送命令，等待redis服务器响应，从而监控了多个redis实例（包括主机和从机）。

然而一个哨兵进程对redis服务器进行监控，可能会出现问题，为此，可以使用多个哨兵进行监控，各个哨兵之间还会进行监控，形成多哨兵模式。

![redis](../pic/redis/redis4.png)

> 配置哨兵

vim sentinel.conf
```
配置redis.conf
sentinel monitor myredis（哨兵名称，随便取） 127.0.0.1 6379 1（主机挂掉后，有多少个哨兵认为挂了，才算真的挂了）
```
执行
```
resid-sentinel sentinel.conf  #启动哨兵进程
```

缺点：

- redis 不好在线扩容，集群一旦达到上限，扩容十分麻烦
- 哨兵配置相当麻烦，里面有很多选项


Sentinel.conf配置文件主要参数解析：
```
# 端口
port 26379

# 是否后台启动
daemonize yes

# pid文件路径
pidfile /var/run/redis-sentinel.pid

# 日志文件路径
logfile "/var/log/sentinel.log"

# 定义工作目录
dir /tmp

# 定义Redis主的别名, IP, 端口，这里的2指的是需要至少2个Sentinel认为主Redis挂了才最终会采取下一步行为
sentinel monitor mymaster 127.0.0.1 6379 2

# 如果mymaster 30秒内没有响应，则认为其主观失效
sentinel down-after-milliseconds mymaster 30000

# 如果master重新选出来后，其它slave节点能同时并行从新master同步数据的台数有多少个，显然该值越大，所有slave节点完成同步切换的整体速度越快，但如果此时正好有人在访问这些slave，可能造成读取失败，影响面会更广。最保守的设置为1，同一时间，只能有一台干这件事，这样其它slave还能继续服务，但是所有slave全部完成缓存更新同步的进程将变慢。
sentinel parallel-syncs mymaster 1

# 该参数指定一个时间段，在该时间段内没有实现故障转移成功，则会再一次发起故障转移的操作，单位毫秒
sentinel failover-timeout mymaster 180000

# 不允许使用SENTINEL SET设置notification-script和client-reconfig-script。
sentinel deny-scripts-reconfig yes

```

修改三台Sentinel的配置文件，如下
```
[root@redis-master ~]# grep -Ev "^$|#" /usr/local/redis/sentinel.conf
port 26379
daemonize yes
pidfile "/var/run/redis-sentinel.pid"
logfile "/var/log/sentinel.log"
dir "/tmp"
sentinel monitor mymaster 192.168.56.11 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes

[root@redis-slave01 ~]# grep -Ev "^$|#" /usr/local/redis/sentinel.conf
port 26379
daemonize yes
pidfile "/var/run/redis-sentinel.pid"
logfile "/var/log/sentinel.log"
dir "/tmp"
sentinel monitor mymaster 192.168.56.11 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes

[root@redis-slave02 ~]# grep -Ev "^$|#" /usr/local/redis/sentinel.conf
port 26379
daemonize yes
pidfile "/var/run/redis-sentinel.pid"
logfile "/var/log/sentinel.log"
dir "/tmp"
sentinel monitor mymaster 192.168.56.11 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```

https://www.cnblogs.com/linuxk/p/10718153.html
