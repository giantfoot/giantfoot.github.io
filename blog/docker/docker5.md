#### 常用命令

后台启动容器
```
[root@hadoop100 ~]# docker run -d centos
0eec738f77c63b33ac0147681dd160b559853630fe4208af4216bf95eb12dc34
[root@hadoop100 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
使用docker run -d 镜像名 运行容器，使用docker ps发现容器没有正常工作，原因是因为，docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，会自动停止。比如只部署了一个nginx，容器启动后，发现自己没有提供任何服务，就会立刻停止。

查看日志
```
docker run -d centos /bin/sh -c "while true;do echo test;sleep 1;done"
1da455e411e3966f761feaa5fe801f05f85839622c4438d30c5a6d7545d1cb3c
[root@hadoop100 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1da455e411e3        centos              "/bin/sh -c 'while t…"   1 second ago        Up 1 second                             zen_mayer

#查看日志，实时打印10条
[root@hadoop100 ~]# docker logs -tf --tail 10 1da455e411e3
2020-06-22T13:33:23.933165051Z test

```
查看容器内部进程信息
```
[root@hadoop100 ~]# docker top 1da455e411e3
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                8603                8588                0                   21:31               ?                   00:00:00            /bin/sh -c while true;do echo test;sleep 1;done
root                8995                8603                0                   21:36               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
```
查看容器镜像的元数据
```
docker inspect 1da455e411e3
```
进入正在运行的容器
```
#通常容器都是通过后台方式运行的，需要将进入容器
docker exec -it 容器ID bashshell #进入容器后开启一个新的终端，常用
或者
docker attach 容器ID  #进入容器正在执行的终端，不会启动新的
```
从容气馁拷贝文件道宿主机
```
#哪怕容器已经关闭，只要docker ps -a能查到，就可以拷贝文件
docker cp 容器ID:目录/文件名 宿主机目录
```
