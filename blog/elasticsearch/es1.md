#### ElasticSearch 概述

Lucene是一套信息检索工具，jar包，不包含搜索引擎系统，它包含索引结构，读写索引的结构，排序，搜索规则，是一套工具类。

> Lucene和ES的关系

ES是基于Luncne做了一些封装和增强，是一个开源的**高扩展**的**分布式全文检索引擎**，它可以近乎实时的存储检索数据，本身扩展性极强，能轻易扩展到上百台服务器，处理PB级别的数据。

ES也使用Java开发并使用Lucene作为核心来实现所有索引和搜索功能，但它巧妙的通过简单的RESTFUL API来隐藏LUCENCE的复杂性，让全文检索变得更简单。

![solr](../pic/es/lucene.png)

![solr](../pic/es/solr.png)

搜索引擎选型对比：

![solr](../pic/es/es和solr对比1.png)

![solr](../pic/es/es和solr对比2.png)

![solr](../pic/es/es和solr对比3.png)

![solr](../pic/es/es和solr对比4.png)
