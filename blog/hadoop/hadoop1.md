### Hadoop学习笔记
Hadoop主要分为四大块：
- **HDFS**：数据存储，主要分为NameNode，DataNode和Secondary NameNode。
    1. **NameNode**：主要存储元数据，相当于目录，包括文件名，文件目录结构，文件属性（生成时间，副本数，文件权限），以及每个文件的块列表和块所在的DataNode等。
    2. **DataNode**：存储块数据和校验数据。
    3. **Secondary NameNode**：监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。
- **Yarn**：资源调度，主要分为RourceManager，NodeManager，Container，ApplicationMaster。
    1. **RourceManager**：集群资源的调度，客户端的请求分发，监控NodeMananger，启动监控ApplicationMaster。
    2. **NodeManager**：单个节点的资源调度，处理来自ResourceManager的命令和来自ApplicatioMaster的命令。
    3. **Container**：容器，是资源的抽象，封装了某个节点上的资源，内存，CPU，磁盘和网络等。
    4. ApplicationMaster：负责数据切分，为应用程序申请资源并分配给内部任务，任务的监控和容错。

    其中，NodeManager，Container和ApplicationMaster出于一个节点上，出于同一层级。
- **MapReduce**：负责运算，类似分治算法，Map分发任务，Reduce聚合结果。
- **Common**：辅助工具
