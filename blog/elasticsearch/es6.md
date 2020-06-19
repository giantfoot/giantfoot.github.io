#### SpringBoot集成ES

官方文档 https://www.elastic.co/guide/en/elasticsearch/client/index.html

配置
```
@Configuration
public class ESConfig {

    @Bean
    public RestHighLevelClient restHighLevelClient() {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http")));
        return restHighLevelClient;
    }
}
```

简单操作
```
@SpringBootTest
class EsStudyApiApplicationTests {

	@Autowired
	private RestHighLevelClient restHighLevelClient;

	@Test
	void createIndexTest() throws IOException {
		//创建索引
		CreateIndexRequest request = new CreateIndexRequest("index_create_test");
		CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
	}

	@Test
	void ifExistTest() throws IOException {
		//查询索引
		GetIndexRequest request = new GetIndexRequest("index_create_test");
		boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
	}

	@Test
	void deleteTest() throws IOException {
		DeleteIndexRequest request = new DeleteIndexRequest("index_create_test");
		restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);

	}

//搜索，重点
  @Test
	void searchTest() throws IOException {
		SearchRequest request = new SearchRequest();

		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

		TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "张");

		searchSourceBuilder.query(termQueryBuilder);
		searchSourceBuilder.timeout(TimeValue.MINUS_ONE);
		request.source(searchSourceBuilder);

		SearchResponse search = restHighLevelClient.search(request, RequestOptions.DEFAULT);
	}

}

//插入文档
	@Test
	void insertTest() throws IOException {
		JSONObject save = new JSONObject().fluentPut("name", "test");

		IndexRequest request = new IndexRequest("test");
		request.id(String.valueOf(1));
		request.timeout(TimeValue.MINUS_ONE);
		request.source(save, XContentType.JSON);

		IndexResponse indexResponse = restHighLevelClient.index(request, RequestOptions.DEFAULT);
	}
```
