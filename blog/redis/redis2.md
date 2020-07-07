#### Redis 概述和安装

REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。

Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

读取速度：110000次/s，写的速度：81000次/s

Redis能干嘛？

- 内存存储，持久化
- 效率高，适用于高速缓存
- 发布订阅
- 地图信息分析
- 计时器，计数器

> 特性

1. 多样化数据类型
2. 持久化
3. 集群
4. 事务

> 安装

解压redis压缩包
```
tar -zxvf redis-6.0.5.tar.gz
```
安装相关依赖
```
gcc -v 查看是否安装了GCC
yum install gcc-c++ （redis6.0以后要gcc9）
#进入redis解压目录执行make命令
make
make install
#安装完成进入下面目录查看
cd /usr/local/bin/
为了方便配置，拷贝配置文件到新建的kconfig目录下
cp /opt/software/redis-5.0.8/redis.conf /usr/local/bin/kconfig/
```

redis默认不是后台启动的，需要修改redis.conf。
```
daemonize yes
```

指定配置文件启动
```
redis-server kconfig/redis.conf
```

连接redis
```
redis-cli -p 6379
set name test
get name
keys *
```

关闭redis
```
shutdown
```
