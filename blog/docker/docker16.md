#### 集成SpringBoot

创建springboot项目，打包成test.jar

安装docker插件，dockerfile高亮
![docker](../pic/docker/docker15.png)

创建Dockerfile文件，编写脚本
```
FROM java:8

COPY test.jar /app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

把jar包和Dockerfile文件上传到服务器

```
docker build -t test-springboot  .
docker run -d -p 8080:8080 --name test01  test-springboot
curl localhost:8080
```

如果最后有很多镜像，需要compose,swarm
