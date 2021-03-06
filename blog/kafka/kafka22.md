#### Java 消费者是如何管理TCP连接的?

kafka 的世界中，无论是 ServerSocket，还是 SocketChannel，它们实现的都是 TCP 协议。
在某些情况下，TCP 连接或 Socket 资源时，指的是同一个东西。

> 何时创建 TCP 连接？

消费者端主要的程序入口是 KafkaConsumer 类。和生产者不同的是，构建 KafkaConsumer 实例时是不会创建任何 TCP 连接的，**TCP 连接是在调用 KafkaConsumer.poll 方法时被创建的。**

再细粒度地说，在 poll 方法内部有 3 个时机可以创建 TCP 连接：

- 发起 FindCoordinator 请求时。

  还记得消费者端有个组件叫协调者（Coordinator）吗？它驻留在 Broker 端的内存中，负责消费者组的组成员管理和各个消费者的位移提交管理。当消费者程序首次启动调用 poll 方法时，它需要向 Kafka 集群发送一个名为 FindCoordinator 的请求，希望 Kafka 集群告诉它哪个 Broker 是管理它的协调者。

  不过，消费者应该向哪个 Broker 发送这类请求呢？理论上任何一个 Broker 都能回答这个问题，也就是说消费者可以发送 FindCoordinator 请求给集群中的任意服务器。在这个问题上，社区做了一点点优化：**消费者程序会向集群中当前负载最小的那台 Broker 发送请求。**负载是如何评估的呢？其实很简单，就是看消费者连接的所有 Broker 中，谁的待发送请求最少。当然了，这种评估显然是消费者端的单向评估，并非是站在全局角度，因此有的时候也不一定是最优解。不过这不并影响我们的讨论。总之，在这一步，消费者会创建一个 Socket 连接。

- 连接协调者时。

  Broker 处理完上一步发送的 FindCoordinator 请求之后，会返还对应的响应结果（Response），显式地告诉消费者哪个 Broker 是真正的协调者，因此在这一步，消费者知晓了真正的协调者后，会创建连向该 Broker 的 Socket 连接。只有成功连入协调者，协调者才能开启正常的组协调操作，比如加入组、等待组分配方案、心跳请求处理、位移获取、位移提交等。

- 消费数据时。

  消费者会为每个要消费的分区创建与该分区领导者副本所在 Broker 连接的 TCP。举个例子，假设消费者要消费 5 个分区的数据，这 5 个分区各自的领导者副本分布在 4 台 Broker 上，那么该消费者在消费时会创建与这 4 台 Broker 的 Socket 连接。

> 何时关闭 TCP 连接？

消费者关闭 Socket 也分为主动关闭和 Kafka 自动关闭。

主动关闭是指你显式地调用消费者 API 的方法去关闭消费者，具体方式就是手动调用 KafkaConsumer.close() 方法，或者是执行 Kill 命令，不论是 Kill -2 还是 Kill -9；而 Kafka 自动关闭是由消费者端参数 connection.max.idle.ms 控制的，该参数现在的默认值是 9 分钟，即如果某个 Socket 连接上连续 9 分钟都没有任何请求“过境”的话，那么消费者会强行“杀掉”这个 Socket 连接。

不过，和生产者有些不同的是，如果在编写消费者程序时，你使用了循环的方式来调用 poll 方法消费消息，那么上面提到的所有请求都会被定期发送到 Broker，因此这些 Socket 连接上总是能保证有请求在发送，从而也就实现了“长连接”的效果。

小结：

整个生命周期里会建立4个连接，进入稳定的消费过程后，同时保持3个连接。

第一类连接：确定协调者和获取集群元数据。
 一个，初期的时候建立，当第三类连接建立起来之后，这个连接会被关闭。

第二类连接：连接协调者，令其执行组成员管理操作。
 一个

第三类连接：执行实际的消息获取。
两个分别会跟两台broker机器建立一个连接，总共两个TCP连接，**同一个broker机器的不同分区可以复用一个socket。**

整个生命周期也有可能是建立3个连接。
当领导者副本都位于同一个broker上时，第三类连接只会建立一个。
