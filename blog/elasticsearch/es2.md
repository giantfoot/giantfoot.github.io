#### ElasticSearch 安装使用

ElasticSearch https://www.elastic.co/cn/downloads/elasticsearch

IK https://github.com/medcl/elasticsearch-analysis-ik

kibana https://www.elastic.co/cn/downloads/kibana

可视化 https://github.com/mobz/elasticsearch-head

安装 cnpm https://blog.csdn.net/wjnf012/article/details/80422313

先配置跨域访问，config目录下 elasticsearch.yml

```
# 添加跨域访问
http.cors.enabled: true
http.cors.allow-origin: "*"
```

安装elasticsearch-head，进入elasticsearch-head根目录，执行
```
cnpm install
执行
npm run start
```

![elk](../pic/es/elk.png)

kibana配置中文汉化
配置 kibana.yml
```
#i18n.locale: "zh-CN"
```
