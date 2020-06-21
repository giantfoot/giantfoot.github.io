#### docker 容器命令解析

运行和退出
```
docker run [可选参数] image
# 参数说明
--name="myRedis" 设置容器名称来区分
-d  后台方式运行
-it 使用交互方式运行，这种方式可以在启动的同时直接进入到容器
-p  指定端口 -p 8080:8090,
    -p ip:主机端口:容器端口
    -p 主机端口:容器端口 （常用）
    -p 容器端口
-P  随机指定端口

[root@hadoop100 ~]# docker run -it centos /bin/bash #启动容器并进入容器
[root@221b7d2bf8db /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@221b7d2bf8db /]# exit  #停止并退出容器
```
查看
```
docker ps       #查看正在运行的容器
docker ps -a    #查看曾经运行过的容器
docker ps -n=2  #查看曾经运行过的n个容器
docker ps -q    #查看曾经运行过的容器编号

Crtl + p + q    #退出容器，但不停止，如果再想进入，执行下面命令

docker attach 容器ID
```
删除
```
docker rm     #容器ID,不能删除正在运行的容器，如果要强行删除，要加 -f
docker rm -f $(docker ps -aq) #删除全部容器
docker ps -aq | xargs docker rm -f  #删除全部容器
```
重新启动和停止容器
```
docker start    #启动容器ID
docker restart  #重启容器ID
docker stop     #停止容器ID
doker kill      #杀除容器ID
```
