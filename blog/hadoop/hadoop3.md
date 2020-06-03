### Hadoop学习笔记三

**安装Hadoop**

- 安装JDK
```
tar -zxvf jdk-8u211-linux-x64.tar.gz -C /opt/module/
```
- 配置环境变量
```
vim /etc/profile
添加
##JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_211
export PATH=$PATH:$JAVA_HOME/bin
执行
source /etc/profile
```

- 安装Hadoop
```
tar -zxvf hadoop-2.7.7.tar.gz -C /opt/module/
配置环境变量
##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.7
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
执行 hadoop 验证是否成功
```

**配置Hadoop**

Hadoop官网
https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html


- 配置和启动HDFS

左下角，core-default.xml -》默认配置信息

hadoop解压目录 etc/hadoop/core-site.xml:
```
<!-- 指定HDFS中的NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop100:9000</value>
    </property>
<!-- 指定Hadoop运行时产生文件的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-2.7.7/data/tmp</value>
    </property>
```
设置hadoop-env.sh的java环境变量

```
cd /opt/module/hadoop-2.7.7/etc/hadoop
vim hadoop-env.sh
```

配置：etc/hadoop/hdfs-site.xml
```
<!-- 指定hdfs副本数量 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
```

**启动Hadoop（HDFS）**

格式化NameNode(只在第一次启动的时候格式化，以后不需要)
> 格式化之前先执行jps，把存在的进程全部关掉，然后把之前生成的目录删除，比如data和logs目录

hadoop解压目录下
```
格式化
bin/hdfs namenode -format
启动namenode
sbin/hadoop-daemon.sh start namenode
启动datanode
sbin/hadoop-daemon.sh start datanode
```

访问 http://192.168.80.100:50070 （访问HDFS）

**hadoop启动后，服务器上就存在了两个文件系统，一个是linux本地，一个是HDFS。本地文件系统命令不变，HDFS文件系统命令格式为：bin/hdfs dfs -后面跟linux命令，例如**

```
bin/hdfs dfs -mkdir /usr/test
bin/hdfs dfs -ls
```

把本地文件系统的文件上传到HDFS文件系统命令：
```
bin/hdfs dfs -put wcinput/wc.input /usr/test//input
```

不要经常格式化namenode:

/opt/module/hadoop-2.7.7/data/tmp/dfs/data/current/version是datanode的版本号。

它和
/opt/module/hadoop-2.7.7/data/tmp/dfs/name/current/version，即namenode的版本号是一样的。
格式化NameNode，会产生新的集群id，即版本号，导致NameNode和DataNode的集群id不一致，集群找不到以往数据。

所以，格式NameNode时，一定要先删除data数据和log日志，然后在格式化NameNode。

> 一定要先关掉NameNode和DataNode的进程后，删除相关目录再格式化

![格式化NameNode的问题](../pic/hadoop/格式化NameNode的问题.PNG)

**启动Hadoop（YARN和MapReduce）**

- 配置yarn-env.sh (etc/hadoop/yarn-env.sh)
```
export JAVA_HOME=/opt/module/jdk1.8.0_211
```

- 配置yarn-site.xml
```
  <!-- Reducer获取数据的方式 -->
  <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
  </property>
  <!-- 指定YARN的ResourceManager的地址 -->
  <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>hadoop100</value>
  </property>
```

- 配置mapred-env.sh (etc/hadoop/yarn-env.sh)
```
export JAVA_HOME=/opt/module/jdk1.8.0_211
```

- mapred-site.xml.template重命名为mapred-site.xml并编辑
```
<!-- 指定MapReduce运行在YARN上 -->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

- 启动集群
  1. 确定NameNode和DataNode已经启动

  2. sbin/yarn-daemon.sh start resourcemanager
  3. sbin/yarn-daemon.sh start nodemanager
  4. 访问 http://hadoop100:8088 (访问MapReduce)，调用Wordcount测试用例，就可以在页面上看到计算过程

**配置历史服务器**
> 为了查看程序的历史运行情况，需要配置历史服务器

1. 配置mapred-site.xml
```
<!-- 指定历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop100:10020</value>
</property>
<!-- 指定历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop100:19888</value>
</property>
```

2. 启动历史服务器
```
sbin/mr-jobhistory-daemon.sh start historyserver
```
**配置日志聚集**

日志聚集： 应用运行完成以后，将程序运行日志信息上传到HDFS系统上
好处：可以方便的查看到程序运行详情，方便开发调试

> 注意：开启日志聚集功能需要重新启动NodeManager，ResourceManager，HistoryManager。



步骤：

1. 配置yarn-site.xml并编辑
```
<!-- 开启日志聚集功能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<!-- 设置日志保留时间7天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>
```

2. 关闭NodeManager，ResourceManager，HistoryManager。最好按顺序关闭
```
sbin/mr-jobhistory-daemon.sh stop historyserver
sbin/yarn-daemon.sh stop nodemanager
sbin/yarn-daemon.sh stop resourcemanager
或者直接 kill -9 pid
```
3. 重启NodeManager，ResourceManager，HistoryManager。最好按顺序启动
```
sbin/yarn-daemon.sh start resourcemanager
sbin/yarn-daemon.sh start nodemanager
sbin/mr-jobhistory-daemon.sh start historyserver
```

4. 删除HDFS上生成的历史数据，output目录
```
bin/hsfs dfs -rm -r /usr/atguigu/output
```

5. 重新运行Wordcount样例程序，生成新的历史数据
```
hadoop jar share/hadoop.mapreduce/hadoop-mapreduce-examples-2.7.7.jar wordcount user/atguigu/inout /usr/atguigu/output
```


**完全分布式**

批量同步脚本
```

#!/usr/bin/expect

set password 5091125901
set user root

set pdir [lindex $argv 0]

for { set host 102 } { $host<105} {incr host} {
        spawn rsync -rvl $pdir $user@hadoop$host:$pdir
        expect {
                "*assword:*" { send "$password\r"; exp_continue }
                "*es/no)?*" { send "yes\r" }
        }
        set timeout 500
}
```

![完全分布式集群搭建](../pic/hadoop/完全分布式集群搭建.PNG)

- NameNode和SecondaryNameNode要避免放在一台服务器上
- ResourManager要避免和NameNode和SecondaryNameNode放在同一台服务器。

配置core-site.xml
```
<!-- 指定HDFS中的NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:9000</value>
    </property>
```
配置hadoop-env.sh
```
<!-- 指定HDFS中的NameNode的地址 -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:50090</value>
    </property>
```
配置yarn-site.xml
```
<!-- 指定YARN的ResourceManager的地址 -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop103</value>
</property>
```

利用同步脚本同步所有配置

开始启动：

1. 删除之前所有节点的生成目录，比如data和logs目录

2. 格式化NameNode
```
bin/hdfs namenode -format
```
3. 启动NameNode，DataNode （NameNode有一个，DataNode有多个）

**节点太多的话不可能一个个节点启动，需要配置ssh无密码登录**

命令：ssh hadoop102 可以输密码登录

如果想不输密码：

![ssh免密登录](../pic/hadoop/ssh免密登录.PNG)

```
hadoop100：
回到home目录
ls -al
找到.ssh目录
ssh-keygen -t rsa (连续三次回车)
ssh-copy-id hadoop102 (拷贝公钥)
hadoop102的.ssh目录就会多一个Authorized_keys文件
现在就可以ssh hadoop102免密登录
ssh-copy-id hadoop101 （目前的节点也要拷贝一份，不然自己访问自己也要输密码）
```

**存在NameNode和ResourceManager的机器都要配置ssh免密登录**

群起集群

1. 配置slaves （存放的都是DataNode节点）
```
/opt/module/hadoop-2.7.7/etc/hadoop/slaves
删除原有内容
添加
hadoop102
hadoop103
hadoop104
```
> 该文件中添加的内容结尾不允许有空格，文件中不允许有空行

2. 同步所有节点
```
xsync slaves
```
3. 关闭所有DataNode和NameNode

4. sbin/start-dfs.sh (启动所有node)

5. 启动manager,必须在ResourceManager所在的节点才可以
```
sbin/start-yarn.sh
```

6. 集群测试
```
上传测试文件
bin/hdfs dfs -put wcinput/wc.input /
bin/hdfs dfs -put /opt/software/hadoop-2.7.7.tar.gz /
访问 http://hadoop102:50070
文件存放地址：
/opt/module/hadoop-2.7.7/data/tmp/dfs/data/current/............/current/finalized/subdir0/subdir0
如果一个文件分成多块，比如hadoop-2.7.7.tar.gz，可以把多块内容还原：
cat 文件名1 >> tmp.text
cat 文件名2 >> tmp.text
tar -zxvf tmp.text
```

7. 各个模块分开启动停止（配置ssh是前提）常用
```
整体启动停止HDFS
start-dfs.sh   stop-dfs.sh
整体启动停止YARN
start-yarn.sh  stop-yarn.sh
```
