#### NameNode和SecondaryNameNode工作机制

NameNode的源数据存在哪里？

如果全部存储在磁盘中，因为NameNode要经常进行随机访问，还要快速响应客户请求，所以效率过低，无法满足要求。如果全部放在内存中，如果一旦断电，元数据就会全部丢失，整个集群就全部无法工作。所以NameNode会把源数据存放在内存并备份在磁盘上的镜像文件中FsImage。

但这样也会有新的问题，如果没次更新内存中的元数据时都同时更新FsImage，就会导致效率过低，若长时间不更新，一旦NameNode节点断电，就会产生数据丢失。因此又引入编辑日志文件Edits（只做追加操作，效率很高）。每当源数据有更新或者添加元数据时，修改内存中的元数据并追加到磁盘上的Edits，这样断电后也可以从FsImage和Edits中恢复数据。

但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且断电后恢复数据时间过长。因此，需要定期将FsImage和Edits合并，但如果这个节点由NameNode完成，会使它的响应效率降低，因此引入新的节点，专门做FsImage和Edits的合并。

NameNode工作机制

![NameNode工作机制](../pic/hadoop/NameNode.png)


查看FsImage和Edits文件

/dfs/tmp/name/目录下

执行hdfs可看到相关提示，oiv和oev参数

把序列化日志转为xml文件
```
镜像文件
hdfs oiv -p XML -i fsimage_0000000 -o fsiamge.xml
编辑文件
hdfs oev -p XML -i fsimage_0000000 -o fsiamge.xml
```

![NameNode工作机制](../pic/hadoop/NameNode2.png)

NameNode如何确定下次开机启动的时候合并哪些Edits？

查看seen_teid文件，以前合并过的就不再合并
