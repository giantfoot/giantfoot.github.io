#### HyperLogLog

HyperLogLog基数统计算法

基数：不重复元素个数 [1,3,5,7,8,7]的基数为5(7重复)

网页UV：单个客户端访问网站的次数

传统方式用set保存用户id，统计id的个数，如果量很大的话会很麻烦，我们只是为了计数，并不是为了保存id，这时就可以用HyperLogLog。

占用内存小，2的64次方的不同元素只要12KB，只有0.81%的错误率，可以忽略不计

```
pfadd myset a b c d e f g h
pfcount myset 统计基数
pfmerge myset3 myset1 myset2 #合并到myset3
pfcount myset3


```
