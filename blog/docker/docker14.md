#### 容器互联-自定义网络

移除网络
```
docker network ls
docker network rm 网络名
```

> 网络模式

```
bridge:桥接，docker默认，通过宿主机连接网络，自定义也使用桥接
nono：不配置网络
host：和宿主机共享网络
container：容器网络连通
```

自定义网络

```
# 默认命令会自动添加 --net bridge，就是我们的docker0
docker run -d -P --name tomcat01 (--net bridge)
#自定义网络
# subnet 192.168.0.0/16 共有 192.168.0.2-192.168.255.255 65535个ip
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
docker network ls
```
测试
```
docker run -d -P --name tomcat-net-01 --net mynet tomcat
docker run -d -P --name tomcat-net-02 --net mynet tomcat
# 自定义网络mynet下的两个容器可以相互连通
docker exec -it tomcat-net-01 ping tomcat-net-02
```

好处

不同的集群使用不同的网络，保证集群是安全健康的，比如Redis集群使用redis-net网络，MySQL集群使用mysql-net网络。

**网络连通**

连接一个容器到另外一个网络，比如连通默认网络docker0上的容器到自定义网络mynet
```
docker network connect --help
#连通之后就是把容器tomcat01放到了mynet网络下，既是一个容器两个ip，网络和网络无法打通（ 不在同一个网段，但容器和网络能）
docker network connect mynet tomcat01
#查看具体信息
docker network inspect mynet
dcoker exec -it tomcat01 ping mynet
```
![自定义网络](../pic/docker/docker10.png)
