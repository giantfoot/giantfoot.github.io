#### HDFS 的 SHELL操作 （重点）

bin/hadoop fs 具体命令 或者 /bin/hdfs dfs 具体命令 其中dfs是fs的实现类

在解压目录下执行（/opt/module/hadoop-2.7.7）,**配置了环境变量可以在任何目录下运行**

例子：

```
    （1）查看帮助
        hdfs dfs -help

    （2）查看当前目录信息
        hdfs dfs -ls /

    （3）上传文件
        hdfs dfs -put /本地路径 /hdfs路径

    （4）剪切文件
        hdfs dfs -moveFromLocal a.txt /aa.txt
        
    （5）下载文件到本地
        hdfs dfs -get /hdfs路径 /本地路径

    （6）合并下载
        hdfs dfs -getmerge /hdfs路径文件夹 /合并后的文件

    （7）创建文件夹
        hdfs dfs -mkdir /hello

    （8）创建多级文件夹
        hdfs dfs -mkdir -p /hello/world

    （9）移动hdfs文件
        hdfs dfs -mv /hdfs路径 /hdfs路径

    （10）复制hdfs文件
        hdfs dfs -cp /hdfs路径 /hdfs路径

    （11）删除hdfs文件
        hdfs dfs -rm /aa.txt

    （12）删除hdfs文件夹
        hdfs dfs -rm -r /hello

    （13）查看hdfs中的文件
        hdfs dfs -cat /文件
        hdfs dfs -tail -f /文件

    （14）查看文件夹中有多少个文件
        hdfs dfs -count /文件夹

    （15）查看hdfs的总空间
        hdfs dfs -df /
        hdfs dfs -df -h /

    （16）修改副本数    
        hdfs dfs -setrep 1 /a.txt

```

**linux中du与df的区别和联系**
```
1，两者区别
du，disk usage,是通过搜索文件来计算每个文件的大小然后累加，du能看到的文件只是一些当前存在
的，没有被删除的。他计算的大小就是当前他认为存在的所有文件大小的累加和。
df，disk free，通过文件系统来快速获取空间大小的信息，当我们删除一个文件的时候，这个文件不
是马上就在文件系统当中消失了，而是暂时消失了，当所有程序都不用时，才会根据OS的规则释放掉已
经删除的文件， df记录的是通过文件系统获取到的文件的大小，他比du强的地方就是能够看到已经删除
的文件，而且计算大小的时候，把这一部分的空间也加上了，更精确了。
当文件系统也确定删除了该文件后，这时候du与df就一致了。

2,du查看目录大小，df查看磁盘使用情况。
我常使用的命令（必要时，sudo使用root权限），
1).查看某个目录的大小：du -hs /home/master/documents
查看目录下所有目录的大小并按大小降序排列：sudo du -sm /etc/* | sort -nr | less
2).查看磁盘使用情况（文件系统的使用情况）：sudo df -h
df --block-size=GB
-h是使输出结果更易于人类阅读；du -s只展示目录的使用总量（不分别展示各个子目录情况），-m是以
MB为单位展示目录的大小（当然-k/-g就是KB/GB了）。
```
