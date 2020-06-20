#### docker 安装使用

官方文档 https://docs.docker.com/engine/install/centos/

卸载旧版本
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
配置相关依赖和阿里仓库
```
sudo yum install -y yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
安装docker (docker-ce社区办)
```
#更新yum软件索引包
yum makecache fast
#安装docker
sudo yum install docker-ce docker-ce-cli containerd.io
```

如果不想安装最新版，可以选择版本
```
yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable

sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
启动测试docker
```
sudo systemctl start docker
docker version
sudo docker run hello-world
```

卸载
```
sudo yum remove docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
```

还可以配置阿里云镜像加速，跟上面的阿里云镜像仓库不一样

![docker](../pic/docker/docker1.png)

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://up48u3tz.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
