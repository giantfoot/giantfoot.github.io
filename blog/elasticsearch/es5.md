#### ES基本命令使用

![](../pic/es/es10.png)

![](../pic/es/es11.png)

索引增删改查
```
#查看不同分词器的分词效果
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "我的中国心"
}

GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "我的中国心"
}

PUT /index1/type1/1
{
  "name":"test",
  "age":"20"
}


# 创建索引
PUT /index2
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "age":{
        "type": "long"
      },
      "birthday":{
        "type": "date"
      }
    }
  }
}


#获取信息
GET index2


#后面版本废弃type,默认为_doc，字段类型不指定默认设置
PUT index3/_doc/1
{
  "name":"zyy",
  "age":23,
  "birth":"1994-04-04"
}
GET index3


#_cat 查看健康值等基础信息
GET _cat/health

GET _cat/indices?v

#修改可用PUT 直接覆盖修改,不设置的字段会置空，建议使用_update
PUT index3/_doc/1
{
  "name":"zyy2",
  "age":23,
  "birth":"1994-04-04"
}

# 更新
POST /index3/_doc/1 或者POST /index3/_doc/1/_update
{
  "name":"zyyyyyy3"
}

#删除
DELETE /index1
```

文档操作
```
PUT /test/user/3
{
  "name":"张三",
  "age":40,
  "desc":"歹毒的中年男子",
  "tags":["歹毒","冷漠","大胆"]
}


#简单查询，不建议
GET test/user/_search?q=name:三


#查询，全查询
GET test/user/_search
{
  "query": {
    "match": {
      "name": "张"
    }
  }
}

#选择查询字段
GET test/user/_search
{
  "query": {
    "match": {
      "name": "张"
    }
  },
  "_source": ["name","age"]
}

#排序查询
GET test/user/_search
{
  "query": {
    "match": {
      "name": "张"
    }
  },
  "_source": ["name","age"],
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}

#分页查询
GET test/user/_search
{
  "query": {
    "match": {
      "name": "张"
    }
  },
  "_source": ["name","age"],
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 1
}

#多条件查询
#must -> and
GET test/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "张"
          }
        },
        {
          "match": {
            "age": "40"
          }
        }
      ]
    }
  }
}
#should -> or
GET test/user/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "name": "张"
          }
        },
        {
          "match": {
            "age": "40"
          }
        }
      ]
    }
  }
}

#must_not -> not
GET test/user/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "name": "张"
          }
        },
        {
          "match": {
            "age": "40"
          }
        }
      ]
    }
  }
}
#filter -> 过滤，区间等操作
GET test/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "张"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 10,
            "lte": 50
          }
        }
      }
    }
  }
}

#空格隔开 一个字段多条件查询（模糊查询，满足一个条件即可）
GET test/_search
{
 "query": {
   "match": {
     "tags": "心 男"
   }
 }
}

#term查询通过倒排索引精确查询，效率更高
#matc使用分词器解析，先分析文档，然后通过分析的文档查询
#两个字段，text会被分词器解析，keyword不会被分词器解析，默认用keyword分词器，也就是不分词
#term查询直接匹配索引里面初始化分好的词，match需要指定分词器现场分词
#如下例子中，name字段为text类型，虽然是精确匹配，但会被分词，精确匹配到分词中的“张”，所以可以查出张三
GET test/_search
{
 "query": {
   "term": {
     "name": "张"
   }
 }
}

#如下例子中，desc字段为keyword类型，不会被分词，直接匹配整个desc，所以不会查出数据
GET test/_search
{
 "query": {
   "term": {
     "desc": "张"
   }
 }
}

#term也可以在某种情况下代替match，如果字段不是text，它是精确查询
GET test/user/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "name": "张"
          }
        },
        {
          "term": {
            "age": "40"
          }
        }
      ]
    }
  }
}

#高亮

GET test/_search
{
 "query": {
   "term": {
     "name": "张"
   }
 },
 "highlight": {
   "fields": {
     "name": {}
   }
 }
}

#自定义高亮
GET test/_search
{
 "query": {
   "term": {
     "name": "张"
   }
 },
 "highlight": {
   "pre_tags": "<p class = 'key' style='color:red'>",
   "post_tags": "</p>",
   "fields": {
     "name": {}
   }
 }
}
```
![](../pic/es/es12.png)
