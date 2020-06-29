容器数据卷

容器里面一般只有应用和环境，如果把数据也放到容器，容器一旦删除，数据也就没了。
现在想要数据持久化。

容器之间有一种数据共享的技术！

docker容器中产生的数据，同步到本地。

这就是卷技术，目录的挂载，将我们容器的目录挂载到linux宿主机上。

使用数据卷：

> 方式一 直接使用命令挂在 -v

```
#同步，双向绑定，占用两倍存储，可以挂载多个
docker run -it -v 主机目录:容器内目录 -v 主机目录:容器内目录 镜像名
```

> 方式二 具名挂载和匿名挂载

```
#匿名挂载  -v 容器内路径(只写了容器内路径，没写容器外路径)
docker run -d -P --name nginx01 -v /etc/nginx nginx

#查看所有volume的情况
docker volume ls

#具名挂载,名称为 test-nginx
docker run -d -P --name nginx02 -v test-nginx:/etc/nginx nginx
#查看具名卷具体目录(映射在宿主机的目录)
docker volume inspect test-nginx
#一般在下面的目录下
/var/lib/docker/volumes/test-nginx/_data
```

扩展：

```
通过ro，rw改变读写权限
docker run -d -P --name nginx02 -v test-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v test-nginx:/etc/nginx:rw nginx
#ro：这个路径下的内容只能通过宿主机操作，容器内无法操作，默认读写
```

> 方式三 Dockerfile，经常使用

Dockerfile 就是用来构建docker镜像的文件，脚本命令，类似之前的docker commit,脚本里面的命令就是一层一层的组建我们的自定义镜像

```
# 创建一个脚本文件，名称随机，一般以dockerfile开始，比如dockerfile1
# 文件中格式 ： 指令（大写） 参数

FROM centos #以centos最新版镜像为基础创建新镜像
VOLUME ["volume01","volume02"] #挂载的目录（只指定了容器内路径，属于匿名挂载），会出现在容器内的/目录下，映射的是（宿主机）目录var/lib/docker/volumes/xxxx/_data
CMD echo "--- end ----"
CMD /bin/bash

然后执行
docker build -f dockerfile1 -t 镜像名（自定义） .（目录）
执行完就创建了一个镜像
docker images查看
```

**容器间的数据同步 --volumes-from**
```
# 只会同步docker01挂载的数据卷里的数据（比如通过dockerfile创建并启动），删除一个容器，其他容器共享卷仍然存在，只见类似相互拷贝备份的关系
docker run -t --name docker01 centos:7.0
docker run -t --name docker02 --volumes-from docker01 centos:7.0
docker run -t --name docker03 --volumes-from docker01 centos:7.0
```
