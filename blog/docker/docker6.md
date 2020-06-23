#### Docker实例

安装Nginx
```
docker search nginx
docker pull nginx
docker images
docker run -d --name nginx01 -p 3344:80 nginx (宿主机端口:容器内部端口)
curl localhost:3344
```
安装tomcat
```
#官方使用
docker run -it --rm tomcat:9.0
#之前启动容器都是后台启动，停止了容器之后，容器还是能够查到，docker run -it --rm 用完就删,一般只用来测试
#下载再启动
docker pull tomcat
docker run -d -p 3355:8080 --name tomcat01 tomcat
#发现问题：linux很多命令缺少，而且webapps下是空的，因为拉取的镜像是最小版本的，只包含最基本的内容
```
安装ES
```
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2
docker stats #查看CPU状态
curl localhost:9200
#启动命令设置内存（name不要重复）
docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -e ES_JAVA_OPTS="-Xms128m -Xmx1024m"   elasticsearch:7.6.2
```
