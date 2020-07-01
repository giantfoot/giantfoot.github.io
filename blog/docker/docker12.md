#### Docker Hub

> 登录docker hub，发布

```
docker login

docker push 用户名/镜像名:tag #这样有时候会拒绝，此时还没有设置tag，默认是last，需要手动设置tag

docker tag 镜像id 用户名/镜像名:tag
docker push 用户名/镜像名:tag
```

> 发布到阿里云上

![dockerhub](../pic/docker/dockerhub.png)
