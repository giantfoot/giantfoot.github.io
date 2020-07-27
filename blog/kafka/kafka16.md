#### 消费者组到底是什么？

```
Consumer Group ：Kafka提供的可扩展且具有容错性的消息者机制。

1，重要特征：
  A：组内可以有多个消费者实例（Consumer Instance）。
  B：消费者组的唯一标识被称为Group ID，组内的消费者共享这个公共的ID。
  C：消费者组订阅主题，主题的每个分区只能被组内的一个消费者消费
  D：消费者组机制，同时实现了消息队列模型和发布/订阅模型。

2，重要问题：
  A：消费组中的实例与分区的关系：
      消费者组中的实例个数，最好与订阅主题的分区数相同，否则多出的实例只会被闲置。一个分区只能被一个消费者实例订阅。
  B：消费者组的位移管理方式：
      （1）对于Consumer Group而言，位移是一组KV对，Key是分区，V对应Consumer消费该分区的最新位移。
      （2）Kafka的老版本消费者组的位移保存在Zookeeper中，好处是Kafka减少了Kafka Broker端状态保存开销。但ZK是一个分布式的协调框架，不适合进行频繁的写更新，这种大吞吐量的写操作极大的拖慢了Zookeeper集群的性能。
      （3）Kafka的新版本采用了将位移保存在Kafka内部主题的方法。

C：消费者组的重平衡：
  （1）重平衡：本质上是一种协议，规定了消费者组下的每个消费者如何达成一致，来分配订阅topic下的每个分区。
  （2）触发条件：
        a，组成员数发生变更
        b，订阅主题数发生变更
        c，定阅主题分区数发生变更
  （3）影响：
  Rebalance 的设计是要求所有consumer实例共同参与，全部重新分配所有用分区。并且Rebalance的过程比较缓慢，这个过程消息消费会中止。
```
生产者配置
```
spring:
    kafka:
      bootstrap-servers: 10.80.245.23:10086,10.80.244.247:10086,10.66.211.134:10086
      producer:
        retries: 0
        batch-size: 16384
        buffer-memory: 133554432
        key-serializer: org.apache.kafka.common.serialization.StringSerializer
        value-serializer: org.apache.kafka.common.serialization.StringSerializer

@Component
public class KafkaProducer {

    private final KafkaTemplate<String, String> kafkaTemplate;

    public KafkaProducer(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void send(String topic, Message message) {
        kafkaTemplate.send(topic, JSON.toJSONString(message));
    }

}
@Data
public class Message {

    private Long id;
    private LocalDateTime createDate;

    public Message(Long id, LocalDateTime createDate) {
        this.id = id;
        this.createDate = createDate;
    }
}
```
消费者配置
```
    @KafkaListener(groupId = "weiBo-consumer", topicPartitions ={ @TopicPartition(topic =PublicParamConstans.KAFKA_WEIBO_DATA_TOPIC,partitions ="0")})
    public void listenPartition0(ConsumerRecord<String, String> record) throws Exception {
        dealWeiboData(record);
    }

    @KafkaListener(groupId = "weiBo-consumer", topicPartitions ={ @TopicPartition(topic =PublicParamConstans.KAFKA_WEIBO_DATA_TOPIC,partitions = "1") })
    public void listenPartition1(ConsumerRecord<String, String> record) throws Exception {
        dealWeiboData(record);
    }


    @KafkaListener(groupId = "weiBo-consumer", topicPartitions ={ @TopicPartition(topic =PublicParamConstans.KAFKA_WEIBO_DATA_TOPIC,partitions = "2") })
    public void listenPartition2(ConsumerRecord<String, String> record) throws Exception {
        dealWeiboData(record);
    }
```

Consumer Group 是 Kafka 提供的可扩展且具有容错性的消费者机制。

既然是一个组，那么组内必然可以有多个消费者或消费者实例（Consumer Instance），它们共享一个公共的 ID，这个 ID 被称为 Group ID。

组内的所有消费者协调在一起来消费订阅主题（Subscribed Topics）的所有分区（Partition）。当然，每个分区只能由同一个消费者组内的一个 Consumer 实例来消费。

Kafka 仅仅使用 Consumer Group 这一种机制，却同时实现了传统消息引擎系统的两大模型：如果所有实例都属于同一个 Group，那么它实现的就是消息队列模型；如果所有实例分别属于不同的 Group，那么它实现的就是发布 / 订阅模型。

**理想情况下，Consumer 实例的数量应该等于该 Group 订阅主题的分区总数。**

假设一个 Consumer Group 订阅了 3 个主题，分别是 A、B、C，它们的分区数依次是 1、2、3，那么通常情况下，为该 Group 设置 6 个 Consumer 实例是比较理想的情形，因为它能最大限度地实现高伸缩性。

你可能会问，我能设置小于或大于 6 的实例吗？

当然可以！如果你有 3 个实例，那么平均下来每个实例大约消费 2 个分区（6 / 3 = 2）；

如果你设置了 8 个实例，那么很遗憾，有 2 个实例（8 – 6 = 2）将不会被分配任何分区，它们永远处于空闲状态。因此，在实际使用过程中一般不推荐设置大于总分区数的 Consumer 实例。设置多余的实例只会浪费资源，而没有任何好处。

> Rebalance 本质上是一种协议，规定了一个 Consumer Group 下的所有 Consumer 如何达成一致，来分配订阅 Topic 的每个分区。

比如某个 Group 下有 20 个 Consumer 实例，它订阅了一个具有 100 个分区的 Topic。正常情况下，Kafka 平均会为每个 Consumer 分配 5 个分区。这个分配的过程就叫 Rebalance。

Rebalance 的触发条件有 3 个。

- 组成员数发生变更。比如有新的 Consumer 实例加入组或者离开组，抑或是有 Consumer 实例崩溃被“踢出”组。

- 订阅主题数发生变更。Consumer Group 可以使用正则表达式的方式订阅主题，比如 consumer.subscribe(Pattern.compile(“t.*c”)) 就表明该 Group 订阅所有以字母 t 开头、字母 c 结尾的主题。在 Consumer Group 的运行过程中，你新创建了一个满足这样条件的主题，那么该 Group 就会发生 Rebalance。
- 订阅主题的分区数发生变更。Kafka 当前只能允许增加一个主题的分区数。当分区数增加时，就会触发订阅该主题的所有 Group 开启 Rebalance。

比如一个 Group 内有 10 个 Consumer 实例，要消费 100 个分区，理想的分配策略自然是每个实例平均得到 10 个分区。这就叫公平的分配策略。如果出现了严重的分配倾斜，势必会出现这种情况：有的实例会“闲死”，而有的实例则会“忙死”。

**Rebalance 过程对 Consumer Group 消费过程有极大的影响。**

如果你了解 JVM 的垃圾回收机制，你一定听过万物静止的收集方式，即著名的 stop the world，简称 STW。在 STW 期间，所有应用线程都会停止工作，表现为整个应用程序僵在那边一动不动。Rebalance 过程也和这个类似，在 Rebalance 过程中，所有 Consumer 实例都会停止消费，等待 Rebalance 完成。这是 Rebalance 为人诟病的一个方面。
