#### zset

有序集合，命令以z开头

```
zadd myset 1 one          #1为score，用来排序
zadd myset 2 two 2 three  #添加多个值

zrangebyscore myset -inf +inf #从无限小-inf到无限大+inf
zrevrangebyscore              #从大到小排序

zrangebyscore myset -inf +inf withscores #附带分值
zrangebyscore myset -inf 2500 withscores #小于两千五的

zrerange myset 0 -1 #从大到小排序

zcount myset 1 100  #获取区间内的元素


```
