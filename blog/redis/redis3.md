#### 性能测试


redis-benchmark

Redis 性能测试是通过同时执行多个命令实现的。
```
redis-benchmark [option] [option value]
```
> 该命令是在 redis 的目录下执行的，而不是 redis 客户端的内部指令。

实例同时执行 10000 个请求来检测性能：
```
$ redis-benchmark -n 10000  -q
```

具体参数
```
1	-h	指定服务器主机名	127.0.0.1
2	-p	指定服务器端口	6379
3	-s	指定服务器 socket
4	-c	指定并发连接数	50
5	-n	指定请求数	10000
6	-d	以字节的形式指定 SET/GET 值的数据大小	2
7	-k	1=keep alive 0=reconnect	1
8	-r	SET/GET/INCR 使用随机 key, SADD 使用随机值
9	-P	通过管道传输 <numreq> 请求	1
10	-q	强制退出 redis。仅显示 query/sec 值
11	--csv	以 CSV 格式输出
12	-l	生成循环，永久执行测试
13	-t	仅运行以逗号分隔的测试命令列表。
14	-I	Idle 模式。仅打开 N 个 idle 连接并等待。
```
