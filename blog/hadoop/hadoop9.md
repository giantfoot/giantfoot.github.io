#### Hadoop 中的Java操作

- java项目下的resources目录下的配置文件的优先级高于hadoop的/etc下的配置，比如hdfs-site.xml，而代码中的设置又高于resources目录下的配置，比如：
```
Configuration conf = new Configuration();
conf.set("dfs.replication", 3);
```
