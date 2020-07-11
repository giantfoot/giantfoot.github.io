#### Jedis

Jedis跟redis原生命令几乎一模一样，不再详述，简略过下

```
Jedis jedis = new Jedis("127.0.0.1",6379);

#字符串
jedis.set("name", "test");

#list
jedis.lpush();

#set
jedis.sadd();

#map
Map<String,String> map = new HashMap<>();
map.put("k1",v1);
jedis.hmset("hash",map);
jedis.hset("hash","key5","v5");

...
...
...


```
