#### redis.conf

配置文件对大小写不敏感

```
#加载自定义配置文件
include /path/my.conf
```

```
bind 127.0.0.1 #绑定的IP
protected-mode yes #保护模式
port 6379  #端口

daemonize yes #以守护进程运行
pidfile #后台方式运行，需要指定


loglevel notice  #日志水平
logfile ""        #日志文件路径，""标准输出

databases 16 #默认16个

#持久化规则,redis是内存数据库，如果没有持久化会丢失数据
如果900秒内，若果至少有一个1个key进行了修改，就进行持久化，可配置多个
save 900 1

stop-writes-on-bgsave-error yes #持久化出错是否还需要继续工作

rdbcompression yes #是否压缩rdb文件，会消耗CPU资源

rdbchecksum yes 保存rdb文件的时候，进行错误检查

dir ./ rdb文件保存目录


requirepass 123456  #配置文件设置密码

config set requirepass 123456  # 123456 #命令设置密码
config set requirepass "" #去除密码

auth 123456 # 输入密码进入

maxclients 1000 #最大客户端

maxmemory  #最大内存配置

maxmemory-policy neoviction #达到内存上限后，配置处理策略

appendonly no #默认不开启aof模式，默认使用rdb持久化，绝大部分情况下rdb完全够用

appendfsync everysec #每秒执行一次，

```
