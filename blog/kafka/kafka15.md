#### 幂等生产者和事务生产者是一回事吗？

Kafka 消息交付可靠性保障以及精确处理一次语义的实现。

```
消息交付可靠性：
（1）可靠性保障，常见有三种：
最多一次（at most once）:消息可能会丢失，但不会被重复发送
至少一次（at least once）:消息不会丢失，但可能会重复发送
精确一次（exactly once）:消息不会对丢失，也不会被重复发送
（2）Kafka默认提供交付可靠性保障是至少一次
（3）Kafka消息交付可靠性保障以及精确处理一次语义通过两种机制来实现的：冥等性（Idempotence）和事务（Transaction）。


冥等性
（1）什么是幂等性（Idempotence）
       A：“幂等”：原是数学概念，指某些操作或函数能够被执行多次，但每次得到的结果都不变。
       B：计算机领域的含义：
                   a，在命令式编程语言（如C）中，若一个子程序是幂等的，那它必然不能修改系统状态。无论这个子程序运行多少次，与该子程序的关联的那部分系统保持不变。
                   b，在函数式编程语言（比如Scala或Haskell）中，很多纯函数（pure function）天然就是幂等的，他们不执行任何的side effect。
       C：冥等性的优点：最大的优势是可以安全地重试任何冥等性操作，因为他们不会破坏系统状态
（2）冥等性Producer
       A：开启：设置props.put（“enable.idempotence”，true）或props.put(ProducerConfig.ENABLE_IDEMPOTENC_CONFIG，true)。
       B：特征：开启后，Kafka自动做消息的重复去重。
       C：实现思路：用空间换取时间，Broker端多保存一些字段，当Producer发送了具有相同字段值的消息后，Broker就可以知道这些消息重复，就将这些消息丢弃。
       D：作用范围：
          （1）只能保证单分区上幂等性，无法实现多个分区的幂等性。
          （2）只能实现单会话上的冥等性，当Producer重启后，这种幂等性保证就失效了。

事务
（1）事务概念：
        A：事务提供的安全性保障是经典的ACID。原子性(Atomicity)，一致性(Consistency)，隔离性(Isolation)，持久性(Durability)。
        B：kafka的事务机制可以保证多条消息原子性地写入到目标分区，同时也能保证Consumer只能看到事务成功提交的消息。

（2）事务型Producer：
        A：开启：
                  【1】设置enable.idempotence = true。
                  【2】设置Producer端参数transactional.id。最好为其设置一个有意义的名字。
                  【3】调整Producer代码，显示调用事务API。
                  【4】设置Consumer端参数 isolation.level 值：
                          read_uncommitted(默认值，能够读到kafka写入的任何消息)
                          read_committed（Consumer只会读取事务型Producer成功事务写入的消息。）
        B：特征：
                 【1】能够保证将消息原子性地写入到多个分区中。一批消息要么全部成功，要么全部失败。
                 【2】不惧进程重启，Producer重启回来后，kafka依然能保证发送的消息的精确一次处理。

关键事项：
       1，幂等性无法实现多个分区以及多会话上的消息无重复，但事务（transaction）或依赖事务型Produce可以做到。
       2，开启事务对性能影响很大，在使用时要充分考虑
```

问：kafka中的事务提交异常，broker端的数据还是会写入日志，相当于只是记录一下失败状态，在消费端通过隔离级别，来过滤掉出这部分消息，不进行消费。为什么事务异常了，还要将数据写入日志呢？直接删除掉不好吗？像DB那样。

答： Kafka broker基本上还是保持append-only的日志型风格，不做删除处理


**Kafka 也可以提供最多一次交付保障，只需要让 Producer 禁止重试即可。**

这样一来，消息要么写入成功，要么写入失败，但绝不会重复发送。我们通常不会希望出现消息丢失的情况，但一些场景里偶发的消息丢失其实是被允许的，相反，消息重复是绝对要避免的。此时，使用最多一次交付保障就是最恰当的。

**幂等性有很多好处，其最大的优势在于我们可以安全地重试任何幂等性操作，反正它们也不会破坏我们的系统状态。** 如果是非幂等性操作，我们还需要担心某些操作执行多次对状态的影响，但对于幂等性操作而言，我们根本无需担心此事。

enable.idempotence 被设置成 true 后，Producer 自动升级成幂等性 Producer，其他所有的代码逻辑都不需要改变。**Kafka 自动帮你做消息的重复去重。**

底层具体的原理很简单，就是经典的用空间去换时间的优化思路，即在 Broker 端多保存一些字段。当 Producer 发送了具有相同字段值的消息后，Broker 能够自动知晓这些消息已经重复了，于是可以在后台默默地把它们“丢弃”掉。当然，实际的实现原理并没有这么简单，但你大致可以这么理解。

**隔离性表明并发执行的事务彼此相互隔离，互不影响。经典的数据库教科书把隔离性称为可串行化 (serializability)，即每个事务都假装它是整个数据库中唯一的事务。**

Kafka 自 0.11 版本开始也提供了对事务的支持，**目前主要是在 read committed 隔离级别上做事情**。它能保证多条消息原子性地写入到目标分区，同时也能保证 Consumer 只能看到事务成功提交的消息。

除了设置，你还需要在 Producer 代码中做一些调整，如这段代码所示：
```

producer.initTransactions();
try {
            producer.beginTransaction();
            producer.send(record1);
            producer.send(record2);
            producer.commitTransaction();
} catch (KafkaException e) {
            producer.abortTransaction();
}
```

这段代码能够保证 Record1 和 Record2 被当作一个事务统一提交到 Kafka，要么它们全部提交成功，要么全部写入失败。

实际上即使写入失败，Kafka 也会把它们写入到底层的日志中，也就是说 Consumer 还是会看到这些消息。

因此在 Consumer 端，读取事务型 Producer 发送的消息也是需要一些变更的。
修改起来也很简单，设置 isolation.level 参数的值即可。

当前这个参数有两个取值：

- read_uncommitted：这是默认值，表明 Consumer 能够读取到 Kafka 写入的任何消息，不论事务型 Producer 提交事务还是终止事务，其写入的消息都可以读取。很显然，如果你用了事务型 Producer，那么对应的 Consumer 就不要使用这个值。
- read_committed：表明 Consumer 只会读取事务型 Producer 成功提交事务写入的消息。当然了，它也能看到非事务型 Producer 写入的所有消息。


**比起幂等性 Producer，事务型 Producer 的性能要更差，在实际使用过程中，我们需要仔细评估引入事务的开销，切不可无脑地启用事务。**

https://www.jianshu.com/p/f77ade3f41fd
