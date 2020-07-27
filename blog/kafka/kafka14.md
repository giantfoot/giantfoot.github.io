#### Java生产者是如何管理TCP连接的？

```
Apache Kafka的所有通信都是基于TCP的，而不是于HTTP或其他协议的

1 为什采用TCP?
（1）TCP拥有一些高级功能，如多路复用请求和同时轮询多个连接的能力。
（2）很多编程语言的HTTP库功能相对的比较简陋。
名词解释：
多路复用请求：multiplexing request，是将两个或多个数据合并到底层—物理连接中的过程。TCP的多路复用请求会在一条物理连接上创建若干个虚拟连接，每个虚拟连接负责流转各自对应的数据流。严格讲：TCP并不能多路复用，只是提供可靠的消息交付语义保证，如自动重传丢失的报文。

2 何时创建TCP连接？
（1）在创建KafkaProducer实例时，
    A：生产者应用会在后台创建并启动一个名为Sender的线程，该Sender线程开始运行时，首先会创建与Broker的连接。
    B：此时不知道要连接哪个Broker，kafka会通过METADATA请求获取集群的元数据，连接所有的Broker。
（2）还可能在更新元数据后，或在消息发送时
3 何时关闭TCP连接
  Producer端关闭TCP连接的方式有两种：用户主动关闭，或kafka自动关闭。
    A：用户主动关闭，通过调用producer.close()方关闭，也包括kill -9暴力关闭。
    B：Kafka自动关闭，这与Producer端参数connection.max.idles.ms的值有关，默认为9分钟，9分钟内没有任何请求流过，就会被自动关闭。这个参数可以调整。
    C：第二种方式中，TCP连接是在Broker端被关闭的，但这个连接请求是客户端发起的，对TCP而言这是被动的关闭，被动关闭会产生大量的CLOSE_WAIT连接。
```

Kafka 生产者程序概览Kafka 的 Java 生产者 API 主要的对象就是 KafkaProducer。

通常我们开发一个生产者的步骤有 4 步。

第 1 步：构造生产者对象所需的参数对象。

第 2 步：利用第 1 步的参数对象，创建 KafkaProducer 对象实例。

第 3 步：使用 KafkaProducer 的 send 方法发送消息。

第 4 步：调用 KafkaProducer 的 close 方法关闭生产者并释放各种系统资源。上面这 4 步写成 Java 代码的话大概是这个样子：
```

Properties props = new Properties ();
props.put(“参数1”, “参数1的值”)；
props.put(“参数2”, “参数2的值”)；
……
try (Producer<String, String> producer = new KafkaProducer<>(props)) {
            producer.send(new ProducerRecord<String, String>(……), callback);
  ……
}
```

在创建 KafkaProducer 实例时，生产者应用会在后台创建并启动一个名为 Sender 的线程，该 Sender 线程开始运行时首先会创建与 Broker 的连接。

你也许会问：怎么可能是这样？如果不调用 send 方法，这个 Producer 都不知道给哪个主题发消息，它又怎么能知道连接哪个 Broker 呢？

难不成它会连接 bootstrap.servers 参数指定的所有 Broker 吗？嗯，是的，Java Producer 目前还真是这样设计的。

**bootstrap.servers 参数**。

它是 Producer 的核心参数之一，**指定了这个 Producer 启动时要连接的 Broker 地址**。

请注意，这里的“启动时”，代表的是 Producer 启动时会发起与这些 Broker 的连接。因此，如果你为这个参数指定了 1000 个 Broker 连接信息，那么很遗憾，你的 Producer 启动时会首先创建与这 1000 个 Broker 的 TCP 连接。


在实际使用过程中，我并不建议把集群中所有的 Broker 信息都配置到 bootstrap.servers 中，通常你指定 3～4 台就足以了。

**因为 Producer 一旦连接到集群中的任一台 Broker，就能拿到整个集群的 Broker 信息**，故没必要为 bootstrap.servers 指定所有的 Broker。

连接完成后，Producer 向某一台 Broker 发送了 METADATA 请求，尝试获取集群的元数据信息——这就是前面提到的 Producer 能够获取集群所有信息的方法。

TCP 连接还可能在两个地方被创建：一个是在更新元数据后，另一个是在消息发送时。

为什么说是可能？

因为这两个地方并非总是创建 TCP 连接。

当 Producer 更新了集群的元数据信息之后，如果发现与某些 Broker 当前没有连接，那么它就会创建一个 TCP 连接。

同样地，当要发送消息时，Producer 发现尚不存在与目标 Broker 的连接，也会创建一个。

**Producer 更新集群元数据信息的两个场景。**

场景一：当 Producer 尝试给一个不存在的主题发送消息时，Broker 会告诉 Producer 说这个主题不存在。此时 Producer 会发送 METADATA 请求给 Kafka 集群，去尝试获取最新的元数据信息。

场景二：Producer 通过 metadata.max.age.ms 参数定期地去更新元数据信息。该参数的默认值是 300000，即 5 分钟，也就是说不管集群那边是否有变化，Producer 每 5 分钟都会强制刷新一次元数据以保证它是最及时的数据。
