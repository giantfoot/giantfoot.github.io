#### 应该选择哪种Kafka？

Kafka 的确有好几种，这里我不是指它的版本，而是指存在多个组织或公司发布不同的 Kafka。

你一定听说过 Linux 发行版吧，比如我们熟知的 CentOS、RedHat、Ubuntu 等，它们都是 Linux 系统，但为什么有不同的名字呢？

其实就是因为它们是不同公司发布的 Linux 系统，即不同的发行版。虽说在 Kafka 领域没有发行版的概念，但你姑且可以这样近似地认为市面上的确存在着多个 Kafka“发行版”。下面我就来梳理一下这些所谓的“发行版”以及你应该如何选择它们。

当然了，“发行版”这个词用在 Kafka 框架上并不严谨，但为了便于我们区分这些不同的 Kafka，我还是勉强套用一下吧。不过切记，当你以后和别人聊到这个话题的时候最好不要提及“发行版”这个词 ，因为这种提法在 Kafka 生态圈非常陌生，说出来难免贻笑大方。

1. Apache KafkaApache Kafka 是最“正宗”的 Kafka，也应该是你最熟悉的发行版了。自 Kafka 开源伊始，它便在 Apache 基金会孵化并最终毕业成为顶级项目，它也被称为社区版 Kafka。咱们专栏就是以这个版本的 Kafka 作为模板来学习的。更重要的是，它是后面其他所有发行版的基础。也就是说，后面提到的发行版要么是原封不动地继承了 Apache Kafka，要么是在此之上扩展了新功能，总之 Apache Kafka 是我们学习和使用 Kafka 的基础。

2. Confluent Kafka我先说说 Confluent 公司吧。2014 年，Kafka 的 3 个创始人 Jay Kreps、Naha Narkhede 和饶军离开 LinkedIn 创办了 Confluent 公司，专注于提供基于 Kafka 的企业级流处理解决方案。2019 年 1 月，Confluent 公司成功融资 D 轮 1.25 亿美元，估值也到了 25 亿美元，足见资本市场的青睐。这里说点题外话， 饶军是我们中国人，清华大学毕业的大神级人物。我们已经看到越来越多的 Apache 顶级项目创始人中出现了中国人的身影，另一个例子就是 Apache Pulsar，它是一个以打败 Kafka 为目标的新一代消息引擎系统。至于在开源社区中活跃的国人更是数不胜数，这种现象实在令人振奋。还说回 Confluent 公司，它主要从事商业化 Kafka 工具开发，并在此基础上发布了 Confluent Kafka。Confluent Kafka 提供了一些 Apache Kafka 没有的高级特性，比如跨数据中心备份、Schema 注册中心以及集群监控工具等。

3. Cloudera/Hortonworks KafkaCloudera 提供的 CDH 和 Hortonworks 提供的 HDP 是非常著名的大数据平台，里面集成了目前主流的大数据框架，能够帮助用户实现从分布式存储、集群调度、流处理到机器学习、实时数据库等全方位的数据处理。我知道很多创业公司在搭建数据平台时首选就是这两个产品。不管是 CDH 还是 HDP 里面都集成了 Apache Kafka，因此我把这两款产品中的 Kafka 称为 CDH Kafka 和 HDP Kafka。当然在 2018 年 10 月两
