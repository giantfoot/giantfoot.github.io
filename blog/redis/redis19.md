#### 发布订阅

Redis是一种消息通信模式。

Redis客户端可以订阅任意数量的频道。

```
subscribe mychannel #创建并订阅一个频道
publish mychannel "hello" #往指定频道发布一个消息


unsubscribe mychannel #取消订阅
```

redis是使用c实现的。

原理：

通过subscribe定于某频道后，redis-server里面维护了一个字典，字典的键就是一个个频道，字典的值就是一个链表，链表中保存了所有订阅这个channel的客户端，subscribe的关键，就是把客户端添加到给定channel的订阅链表中。

通过publish向订阅者发送消息，redis-server会使用给定的频道作为键，在它所维护的channel字典中查找记录了订阅这个频道的所有客户端列表，遍历这个链表，将消息发布给所有订阅者。

在redis中，你可以设定对某一key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有他的订阅者都会收到相应的消息，这一功能最明显的做法就是用作实时消息系统。
