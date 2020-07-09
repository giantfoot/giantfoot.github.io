#### List常用命令


list可以做栈，队列

所有list命令都是用l开头的

想象一个数组，lpush就是把数据放在下标0的位置，原下标为0的数据不断后移

rpush就是把数据放在下标为length-1的位置，原来的最后一个数据不断往前移

```
lpush list one    #向头部添加值，相当于栈
rpush list two    #向尾部添加值，相当于队列（或者相当于往栈底添加数据）
lrange 0 1        #取下标为0到1的数据，类似栈，从栈顶开始取数据
lrange list 0 -1  #取出全部数据

lpop list     #移除列表的第一个最左边元素
rpop list     #移除列表的最后一个最右边元素
lindex  list 1#通过下标获取元素
llen list     #获取列表长度

lrem list 1 one #从list中移除指定个数的指定元素，可能包含重复元素

ltrim list 1 2 #把list截断，list中只剩下坐标为1到2的元素

rpoplpush list newlist  #移除列表最后一个元素，并把它添加到新列表的最左边

lset list 0 hello  #列表必须存在，往list的下标0设置为hello，类似于更新，此下标必须要有元素存在，列表或元素不存在则报错

linsert list before hello test #在hello前面插入test
linsert list after test world  #在test后面插入world

```
- 如果移除了所有元素，代表key不存在
- 如果key不存在，新建
