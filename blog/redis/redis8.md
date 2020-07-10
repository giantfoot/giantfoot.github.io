#### Map

所有命令都以h开头

```
hset myhash key1 value1

hmset myhash k1 v2 k2 v2 #批量添加
hmget myhash k1 k2
hgetall myhash           #获取所有值

hdel myhash k1           #删除指定key

hlen myhash              #查看元素个数
hexists myhash k1

hkeys
hvalues

hincrby myhash k1 -1  #value的值减一

hsetnx myhash k1 v1


hset user:1 name zyy age 11
```
