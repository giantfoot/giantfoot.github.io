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
