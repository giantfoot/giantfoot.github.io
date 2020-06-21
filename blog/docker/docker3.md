#### docker 镜像命令解析

> run命令运行过程

- dcoker会在本机查找是否有该镜像

- 有该镜像->直接运行这个镜像
- 无该镜像->去docker hub（配置过的去阿里云）上下载
- 若是docker hub上找不到。返回错误，找到则下载到本地运行
![docker](../pic/docker/docker2.png)

docker是一个cs（client-server）结构的系统，docker的守护进程运行在主机（比如本地linux系统，通过客户端访问）上，守护进程docker-server接收到docker-client的命令，就会在服务器上运行这个命令，比如开启一个新容器。
![docker](../pic/docker/docker3.png)

> docker为什么比虚拟机快？

- docker有着更少的抽象层
- docker利用的是宿主机的内核，vm需要虚拟出自己的内核
- 也就是说，新建容器的时候，docker不需要像虚拟机一样重新加载一个操作系统内核，而是直接利用宿主机的内核

> docker命令

官方文档  https://docs.docker.com/reference/

帮助命令
```
docker version #docker版本信息
docker info    #docker系统信息，内容比较详细
docker 命令 --help #帮助命令
```
镜像
```
[root@hadoop100 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        5 months ago        13.3kB
```
搜索

搜索镜像，可以直接在docker hub上搜索 https://hub.docker.com/，也可以使用docker search命令
```
docker search mysql
docker search mysql --filter=stars=3000 #搜索星数大鱼3000的，过滤条件
```
下载
```
docker pull mysql #下载最新版

[root@hadoop100 ~]# docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
8559a31e96f4: Pull complete   #分层下载，docker image核心，联合文件系统
d51ce1c2e575: Pull complete
c2344adc4858: Pull complete
fcf3ceff18fc: Pull complete
16da0c38dc5b: Pull complete
b905d1797e97: Pull complete
4b50d1c6b05c: Pull complete
c75914a65ca2: Pull complete
1ae8042bdd09: Pull complete
453ac13c00a3: Pull complete
9e680cd72f08: Pull complete
a6b5dc864b6c: Pull complete
Digest: sha256:8b7b328a7ff6de46ef96bcf83af048cb00a1c86282bfca0cb119c84568b4caf6
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest #真实地址也就是说：

docker pull docker.io/library/mysql:latest
等价于
docker pull mysql

指定版本
docker pull mysql:5.7
[root@hadoop100 ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
8559a31e96f4: Already exists #已经存在的就不再下载，极大地节省空间，联合文件系统
d51ce1c2e575: Already exists
c2344adc4858: Already exists
fcf3ceff18fc: Already exists
16da0c38dc5b: Already exists
b905d1797e97: Already exists
4b50d1c6b05c: Already exists
d85174a87144: Pull complete
a4ad33703fa8: Pull complete
f7a5433ce20d: Pull complete
3dcd2a278b4a: Pull complete
Digest: sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```
删除
```
[root@hadoop100 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               5.7                 9cfcce23593a        12 days ago         448MB
mysql               latest              be0dbf01a0f3        12 days ago         541MB
hello-world         latest              bf756fb1ae65        5 months ago        13.3kB
[root@hadoop100 ~]# docker rmi -f 9cfcce23593a
Untagged: mysql:5.7
Untagged: mysql@sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854
Deleted: sha256:9cfcce23593a93135ca6dbf3ed544d1db9324d4c40b5c0d56958165bfaa2d46a
Deleted: sha256:98de3e212919056def8c639045293658f6e6022794807d4b0126945ddc8324be
Deleted: sha256:17e8b88858e400f8c5e10e7cb3fbab9477f6d8aacba03b8167d34a91dbe4d8c1
Deleted: sha256:c04c087c2af9abd64ba32fe89d65e6d83da514758923de5da154541cc01a3a1e
Deleted: sha256:ab8bf065b402b99aec4f12c648535ef1b8dc954b4e1773bdffa10ae2027d3e00

docker rmi -f $(docker images -aq) 全部删除，$()里面存放的是命令，$取变量

docker rmi -f 9cfcce23593a be0dbf01a0f3 bf756fb1ae65 #空格隔开，删除多个镜像
```
