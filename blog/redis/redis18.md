#### AOF持久化

将我们所有命令都记录下来，恢复的时候再把命令全部执行一遍（写操作），默认关闭

报存文件：
```
appendonly yes #默认为no

appendonly.aof


appendfsync always    #每次写操作都会修改
appendfsync everysec  #每秒钟都会修改一次，最多只会丢失一秒钟数据
appendfsync no        #不修改

rewrite 重写（让文件不至于过大），如果aof文件，如果大于配置的64m，就会fork一个新的进程来讲我们的文件进行重写
```


如果aof文件出错，redis无法启动，需要修复,redis提供了工具
```
redis-check-aof --fix appendonly.aof
```

优点：

1. 可以设置每次修改都同步，保证及时性以及完整性
2. 每秒同步一次，默认
3. 从不同步，效率最高


缺点：

1. 持久化文件很大，修复速度慢
2. aof运行效率也比RDB慢

若是同时开启了两种持久化方式，会优先采用AOF文件来恢复数据，因为一般AOF要相对完整。


RDB文件一般只用做后备使用，建议只在slave上持久化，只要15分钟备份一次就够了，只保留
save 900 1这一条规则即可
