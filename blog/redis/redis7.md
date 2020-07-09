#### SET

不存在重复元素

命令都以s开头

```
sadd myset hello        #添加
smembers myset          #查看所有元素
sismember myset hello   #元素是否存在
scard myset             #查看元素个数
srem myset hello        #移除元素

srandmember myset 2     #随机抽选出指定个数的元素

spop myset              #随即移除一个元素
smove myset myset2 helo #将一个指定值移动到另一个set

sdiff myset1 myset2     #两个set的差集
sinter myset1 myset2    #两个set交集
sunion myset myset2     #并集


```
