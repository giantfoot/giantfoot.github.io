#### 网络实战-部署Redis集群

![docker](../pic/docker/docker11.png)

```
创建Redis网络
docker network create redis --subnet 172.38.0.0/16
```
创建6个redis

先创建6个配置文件，执行创建脚本

![docker](../pic/docker/docker12.png)

启动6个Redis容器

![docker](../pic/docker/docker13.png)

启动完成后执行
```
进入容器
docker exec -it redis-1 /bin/sh
```
创建集群
![docker](../pic/docker/docker14.png)

进入集群
```
redis-cli -c
#查看集群
cluster info
cluster nodes
```
测试
```
#设置变量
set a b
关闭某个master，用get测试slave是否顶替master
docker stop redis-3
get a 正常返回 b
```
