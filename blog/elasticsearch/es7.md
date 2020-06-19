#### ES实战

批量插入数据
```
public Boolean parseContent(String Keywords) throws Exception {
        /**
         * 如果需要查询中文
         * 手动用PostMan建立了一下中文分词ES索引，http://127.0.0.1:9200/jd_goods   PUT
         * {
         * 	"settings":{
         * 		"number_of_shards":"5",
         * 		"number_of_replicas":"1"
         *        },
         * 	"mappings":{
         * 			"properties":{
         * 				"img":{
         * 					"type":"text",
         * 					"analyzer": "ik_max_word"
         *                },
         * 				"title":{
         * 					"type":"text",
         * 					"analyzer": "ik_max_word"
         *                },
         * 				"price":{
         * 					"type":"text",
         * 					"analyzer": "ik_max_word"
         *                },
         * 				"shop":{
         * 					"type":"text",
         * 					"analyzer": "ik_max_word"
         *                }
         *
         *            }
         *
         *    }
         * }
         *
         *
         *
         */


        List<Content> contents = new HtmlParseUtil().parseJD(Keywords);
        //把查询的数据放入ES中
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout("2m");
        for (int i = 0; i <contents.size() ; i++) {
            bulkRequest.add(new IndexRequest("jd_goods").source(JSON.toJSONString(contents.get(i)), XContentType.JSON));

        }
        BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        return !bulk.hasFailures();
    }
```

获取数据实现高亮功能
```
    public List<Map<String,Object>> searchPageHighlightBuilder(String keyword,int pageNo,int pageSize) throws IOException {
        if (pageNo <= 1){
            pageNo = 1;
        }

        keyword= URLDecoder.decode(keyword, "UTF-8");

        //条件搜索
        SearchRequest searchRequest = new SearchRequest("jd_goods");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        //分页
        searchSourceBuilder.from(pageNo);
        searchSourceBuilder.size(pageSize);

        //精准匹配
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", keyword);
        searchSourceBuilder.query(termQueryBuilder);
        searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

        //高亮
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("title");
        highlightBuilder.requireFieldMatch(true);//多个高亮显示
        highlightBuilder.preTags("<span style='color:red'>");
        highlightBuilder.postTags("</span>");
        searchSourceBuilder.highlighter(highlightBuilder);

        //执行搜索
        searchRequest.source(searchSourceBuilder);
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        //解析结果
        ArrayList<Map<String,Object>> list = new ArrayList<>();
        for (SearchHit documentFields : search.getHits().getHits()) {

            //解析高亮的字段
            Map<String, HighlightField> highlightFields = documentFields.getHighlightFields();
            HighlightField title = highlightFields.get("title");
            Map<String, Object> sourceAsMap = documentFields.getSourceAsMap();
            if (title != null){
                Text[] fragments = title.fragments();
                String n_title = "";
                for (Text text : fragments) {
                    n_title += text;
                }
                sourceAsMap.put("title",n_title);
            }
            list.add(sourceAsMap);
        }
        return list;


    }
```
