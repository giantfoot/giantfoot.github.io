#### ElasticSearch 安装使用

ElasticSearch https://www.elastic.co/cn/downloads/elasticsearch

IK https://github.com/medcl/elasticsearch-analysis-ik

kibana https://www.elastic.co/cn/downloads/kibana

可视化 https://github.com/mobz/elasticsearch-head

先配置跨域访问，config目录下 elasticsearch.yml

```
# 添加跨域访问
http.cors.enabled: true
http.cors.allow-origin: "*"
```
