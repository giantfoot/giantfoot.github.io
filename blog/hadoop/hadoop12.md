#### NameNode故障处理和安全模式以及多目录

NameNode故障后，可以采用如下两种方法恢复数据：

方法一：将SecondaryNameNode中的数据复制到NameNode存储数据的目录
dfs/tmp/name/,然后单独重新启动NameNode
```
sbin/hadoop-daemon.sh start namenode
```

方法二：使用-importCheckPoint选项启动NameNode守护进程，从而将SecondaryNameNode中的数据复制到NameNode中。【多余繁琐，暂不考虑】


集群安全模式

![安全模式](../pic/hadoop/安全模式.png)

![安全模式](../pic/hadoop/安全模式1.png)

![安全模式](../pic/hadoop/安全模式2.png)

配置NameNode多目录，目的是为了保证NameNode数据的可靠性，两份数据完全一样，由同一个NameNode进程生成，并不保证NameNode的高可用。

![安全模式](../pic/hadoop/namenode多目录.png)
