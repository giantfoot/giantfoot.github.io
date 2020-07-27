#### 客户端都有哪些不常见但是很高级的功能？

> Kafka 拦截器Kafka 拦截器分为生产者拦截器和消费者拦截器。

生产者拦截器允许你在发送消息前以及消息提交成功后植入你的拦截器逻辑；而消费者拦截器支持在消费消息前以及提交位移后编写特定逻辑。

值得一提的是，这两种拦截器都支持链的方式，即你可以将一组拦截器串连成一个大的拦截器，Kafka 会按照添加顺序依次执行拦截器逻辑。

举个例子，假设你想在生产消息前执行两个“前置动作”：第一个是为消息增加一个头信息，封装发送该消息的时间，第二个是更新发送消息数字段，那么当你将这两个拦截器串联在一起统一指定给 Producer 后，Producer 会按顺序执行上面的动作，然后再发送消息。当前 Kafka 拦截器的设置方法是通过参数配置完成的。

生产者和消费者两端有一个相同的参数，名字叫 interceptor.classes，它指定的是一组类的列表，每个类就是特定逻辑的拦截器实现类。拿上面的例子来说，假设第一个拦截器的完整类路径是 com.yourcompany.kafkaproject.interceptors.AddTimeStampInterceptor，第二个类是 com.yourcompany.kafkaproject.interceptors.UpdateCounterInterceptor，那么你需要按照以下方法在 Producer 端指定拦截器：
```

Properties props = new Properties();
List<String> interceptors = new ArrayList<>();
interceptors.add("com.yourcompany.kafkaproject.interceptors.AddTimestampInterceptor"); // 拦截器1
interceptors.add("com.yourcompany.kafkaproject.interceptors.UpdateCounterInterceptor"); // 拦截器2
props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);

```

现在问题来了，我们应该怎么编写 AddTimeStampInterceptor 和 UpdateCounterInterceptor 类呢？

其实很简单，这两个类以及你自己编写的所有 Producer 端拦截器实现类都要继承 org.apache.kafka.clients.producer.ProducerInterceptor 接口。

该接口是 Kafka 提供的，里面有两个核心的方法。

**onSend：该方法会在消息发送之前被调用。**

如果你想在发送之前对消息“美美容”，这个方法是你唯一的机会。

**onAcknowledgement：该方法会在消息成功提交或发送失败之后被调用。**

还记得我在上一期中提到的发送回调通知 callback 吗？

onAcknowledgement 的调用要早于 callback 的调用。值得注意的是，这个方法和 onSend 不是在同一个线程中被调用的，因此如果你在这两个方法中调用了某个共享可变对象，一定要保证线程安全哦。还有一点很重要，这个方法处在 Producer 发送的主路径中，所以最好别放一些太重的逻辑进去，否则你会发现你的 Producer TPS 直线下降。

同理，指定消费者拦截器也是同样的方法，只是具体的实现类要实现 org.apache.kafka.clients.consumer.ConsumerInterceptor 接口，这里面也有两个核心方法。

**onConsume：该方法在消息返回给 Consumer 程序之前调用。也就是说在开始正式处理消息之前，拦截器会先拦一道，搞一些事情，之后再返回给你。**

**onCommit：Consumer 在提交位移之后调用该方法。通常你可以在该方法中做一些记账类的动作，比如打日志等。**

一定要注意的是，指定拦截器类时要指定它们的全限定名，即 full qualified name。通俗点说就是要把完整包名也加上，不要只有一个类名在那里，并且还要保证你的 Producer 程序能够正确加载你的拦截器类。


案例：业务消息从被生产出来到最后被消费的平均总时长是多少？
```

public class AvgLatencyProducerInterceptor implements ProducerInterceptor<String, String> {


    private Jedis jedis; // 省略Jedis初始化


    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        jedis.incr("totalSentMessage");
        return record;
    }


    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
    }


    @Override
    public void close() {
    }


    @Override
    public void configure(Map<java.lang.String, ?> configs) {
    }



    public class AvgLatencyConsumerInterceptor implements ConsumerInterceptor<String, String> {


        private Jedis jedis; //省略Jedis初始化


        @Override
        public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
            long lantency = 0L;
            for (ConsumerRecord<String, String> record : records) {
                lantency += (System.currentTimeMillis() - record.timestamp());
            }
            jedis.incrBy("totalLatency", lantency);
            long totalLatency = Long.parseLong(jedis.get("totalLatency"));
            long totalSentMsgs = Long.parseLong(jedis.get("totalSentMessage"));
            jedis.set("avgLatency", String.valueOf(totalLatency / totalSentMsgs));
            return records;
        }


        @Override
        public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
        }


        @Override
        public void close() {
        }


        @Override
        public void configure(Map<String, ?> configs) {
```
