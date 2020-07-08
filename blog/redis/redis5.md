#### 基本数据类型string

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，**它可以用作数据库、缓存和消息中间件。**

它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。

 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

String常用使用场景：**除了字符串，还可以作为数字使用**


 ```
 exist name     #是否存在
 move name 1    #把name移动到 1 数据库
 del  name      #删除
 expire name 10 #过期时间10s
 ttl name       #还剩多长时间过期
 type name      #查看类型

 append name "hello" #追加字符串
 strlen name         #查看字符串长度，如果当前key不存在，相当于set

 set name 0
 incr name        #加一，虽然是字符串，但也可以实现加一操作
 incrby name 10   #设置自增步长
 decr name        #减一
 decrby name 10   #自减步长


 getrange name 0 3  #截取字符串，包前包后，闭区间
 getrange name 0 -1 #获取全部字符串
 setrange name 1 xx #从下标1开始，替换成xx


 setex name 10 "hello" (set with expire) #设置过期时间
 setnx name hello  （set if not exist）  #不存在再设置（分布式锁）,存在创建失败，原子性操作

 mset k1 v1 k2 v2 k3 v3  #批量设置
 msetnx k1 v1 k4 v4      #创建失败，k1已经存在，原子性

 set user:1 {name:zhangsan,age:11}        #存储对象
 get user:1                               #获取对象
 mset user:2:name zhangsan user:2:age 22  #批量存储对象
 mget user:2:name                         #批量获取对象

 getset name test         #如果不存在值，返回nil
 getset name hello        #如果存在值，获取原来值，并设置新值
 ```
