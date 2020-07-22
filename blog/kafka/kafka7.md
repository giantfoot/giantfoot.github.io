#### kafka集群搭建

> 安装JAVA环境

准备安装包 [jdk-8u162-linux-x64.tar](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1CQoBRkLtAn-4HSdJ4OejdQ)

```
mv jdk-8u162-linux-x64.tar.gz /usr/local
tar -zxvf jdk-8u162-linux-x64.tar.gz
mv jdk-8u162-linux-x64 java
vim /etc/profile

在文件末尾添加如下JAVA环境变量
export JAVA_HOME=/usr/local/java
export JRE_HOME=/usr/local/java/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

 # 检验变量值
source /etc/profile
echo $JAVA_HOME    
java -version
java
javac
```

> 搭建zookeeper集群

准备安装包 [zookeeper-3.4.12.tar](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1uvbXJvjqOetpUB8Y7aT05Q)

在每台主机上执行下面步骤：

```
mv zookeeper-3.4.12.tar /usr/local
tar -zxvf zookeeper-3.4.12.tar
mv zookeeper-3.4.12 zookeeper

配置zookeeper环境变量，首先打开profile文件
vim /etc/profile

在文件末尾添加zookeeper环境变量


#set zookeeper environment
export ZK_HOME=/usr/local/zookeeper
export PATH=$ZK_HOME/bin:$PATH

保存文件后，让该环境变量生效
source /etc/profile

打开zookeeper配置文件
cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg
vim /usr/local/zookeeper/conf/zoo.cfg
修改zookeeper配置文件如下

#修改数据文件夹路径
dataDir=/usr/local/zookeeper/data
#在文件末尾添加
server.1=192.168.1.42:2888:3888
server.2=192.168.1.41:2888:3888
server.3=192.168.1.47:2888:3888
#其它不变


创建数据文件夹

mkdir /usr/local/zookeeper/data

创建myid文件，在myid文件中添加本机的 server ID，在本例中对应关系如下

主机名	  IP地址	     zookeeper	myid
kafka-1	192.168.1.42	server.1	1
kafka-2	192.168.1.41	server.2	2
kafka-3	192.168.1.47	server.3	3

所以，在kafka-1中执行下面命令
echo "1" > /usr/local/zookeeper/data/myid  #kafka-1主机myid

在kafka-2中执行下面命令

echo "2" > /usr/local/zookeeper/data/myid  #kafka-2主机myid
在kafka-3中执行下面命令

echo "3" > /usr/local/zookeeper/data/myid  #kafka-3主机myid

在每台电脑上启动zookeeper
/usr/local/zookeeper/bin/zkServer.sh start

全部启动后，查看启动结果
/usr/local/zookeeper/bin/zkServer.sh status
```

> 搭建kafka集群

准备安装包 [kafka_2.11-2.0.0 .tgz](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1Flc6qthv6p1Dqq7mEISyIA)

在每台主机上执行下面步骤：

```
将安装包移到/usr/local目录下
mv kafka_2.11-2.0.0 .tgz /usr/local

解压文件
tar -zxvf kafka_2.11-2.0.0 .tgz

重命名文件夹为zookeeper
mv kafka_2.11-2.0.0 kafka

配置kafka环境变量，首先打开profile文件
vim /etc/profile

按i进入编辑模式，在文件末尾添加kafka环境变量
#set kafka environment
export KAFKA_HOME=/usr/local/kafka
PATH=${KAFKA_HOME}/bin:$PATH

保存文件后，让该环境变量生效
source /etc/profile

在kafka-1主机中修改server.properties配置文件

打开配置文件
vim /usr/local/kafka/config/server.properties

修改配置如下（IP地址应该根据实际情况填写）
broker.id=1
listeners=PLAINTEXT://192.168.1.42:9092
zookeeper.connect=192.168.1.41:2181,192.168.1.42:2181,192.168.1.47:2181

在kafka-2主机中修改server.properties配置文件
打开配置文件

vim /usr/local/kafka/config/server.properties

修改配置如下（IP地址应该根据实际情况填写）
broker.id=2
listeners=PLAINTEXT://192.168.1.41:9092
zookeeper.connect=192.168.1.41:2181,192.168.1.42:2181,192.168.1.47:2181

在kafka-3主机中修改server.properties配置文件
打开配置文件
vim /usr/local/kafka/config/server.properties

修改配置如下（IP地址应该根据实际情况填写）
broker.id=3
listeners=PLAINTEXT://192.168.1.47:9092
zookeeper.connect=192.168.1.41:2181,192.168.1.42:2181,192.168.1.47:2181

启动kafka（要确保zookeeper已启动）

在每台主机上分别启动kafka
/usr/local/kafka/bin/kafka-server-start.sh -daemon config/server.properties

在其中一台虚拟机(192.168.1.47)创建topic
/usr/local/kafka/bin/kafka-topics.sh --create --zookeeper 192.168.1.47:2181 --replication-factor 3 --partitions 1 --topic test-topic

查看创建的topic信息
/usr/local/kafka/bin/kafka-topics.sh --describe --zookeeper 192.168.1.47:2181 --topic test-topic
```

转 https://www.jianshu.com/p/bdd9608df6b3
