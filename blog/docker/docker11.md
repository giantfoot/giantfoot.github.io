#### dockerfile实战

Docker Hub中99%的镜像都是通过这个基础镜像生成的FROM scratch，然后再来自定义应用

编写dockerfile文件

```
FROM centos

MAINTAINER zyy<1234@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH  #启动容器后默认进入的目录，不配置默认为根目录

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSR 80

CMD echo $MYPATH
CMD echo --end--
CMD /bin/bash
```
docker build -f mydockerfile -t myCentos:1.0 .

查看镜像构建历史
```
docker history 镜像iD
```
