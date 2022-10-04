# 一、Elasticsearch概述

官方介绍：Elasticsearch 是一个分布式、RESTful风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

Elasticsearch 的底层是开源库 Lucene。但是，你没法直接用 Lucene，必须自己写代码去调用它的接口。Elastic 是 Lucene 的封装，提供了 REST API 的操作接口，开箱即用。

ELK：Elasticsearch、Logstash、Kibana

# 二、核心概念

## 物理设计

在后台把每个索引划分成多个分片

## 逻辑设计

- 索引

相当于数据库中的表

- 类型

mapping

- 文档

相当于数据库的一条记录

- 倒排索引

## IK分词器

分词：即把一段中文划分为词

Elasticsearch的一个插件

提供了两个分词算法：ik_smart和ik_max-word，其中ik_smart是最小划分，ik_max-word是最细粒度划分

```
GET _analyze
{
	"analyzer":"ik_smart",
	"text":"超级喜欢狂神说Java"
}
```

**(注）**ik分词器增加自己的配置字典



## Rest风格操作

method	url地址	描述
PUT	/索引名称/类型文档/文档id	创建文档（指定文档id）



# 三、索引的基本操作

## 创建索引

```  
PUT /test1/type1/1
{
	"name": "狂神说",
	"age": 3
}

PUT test_index/
```

## 指定类型

```
PUT /test2
{
	"mapping": {
		"properties": {
			"name": {
				"type": "text"
			},
		}
	}
}
```

如何文档字段没有指定类型，es会给我们默认配置字段类型

## 获得集群信息

```
GET _cat/
```

## 修改文档信息

```
POST /test3/_doc/1/_update
{
	"doc": {
		"name": "法外狂徒张三"
	}
}
```

**注**：加_update的好处是不传的参数不会被覆盖

## 删除索引

```
DELETE test1
```



# 四、文档的基本操作

## 简单的条件查询

```
GET /kuangshen/user/_search?q=name:狂神说
```



## 查看所有数据

```
GET resolution/_search
```



## 复杂查询

```
GET kuangshen/user/_search
{
	"query": {
		"match": {
			"name": "狂神"
		}
	}
}
```



## 指定字段，结果过滤

```
GET kuangshen/user/_search
{
	"query": {
		"match": {
			"name": "狂神"
		}
	},
	"_source": ["name", "desc"]
}
```

## 排序

```
GET kuangshen/user/_search
{
	"query": {
		"match": {
			"name": "狂神"
		}
	},
	"sort": [
		{
			"age": {
				"order": "asc"
			}
		}
	]
}
```

## 分页查询

from：从第几个数据开始
size：返回多少条数据（单页面的数据）

```
GET kuangshen/user/_search
{
	"query": {
		"match": {
			"name": "狂神"
		}
	},
	"sort": [
		{
			"age": {
				"order": "asc"
			}
		}
	],
	"from": 0,
	"size": 1
}
```



## 多条件精确查询

must：所有的条件都要符合，对应于数据库中的and
should：只要有一个条件符合
must_not：对应not

```
GET kuangshen/user/_search
{
	"query": {
		"bool": {
			"must": [
				{
					"match": {
						"name": "狂神说"
					},
					"match": {
						"age": 23
					}
				}
			]
		}
	},
	"_source": ["name", "desc"]
}
```



## 过滤操作

```
GET kuangshen/user/_search
{
	"query": {
		"bool": {
			"must": [
				{
					"match": {
						"name": "狂神说"
					}
				}
			],
			"filter": {
				"range": {
					"age": {
						"gte": 10,
						"lt": 25
					}
				}
			}
		}
	},
}
```



## 匹配条件查询

```
GET kuangshen/user/_search
{
	"query": {
		"match": {
			"tags": "男 技术"
		}
	},
}
```



## 精确查询

term：直接精确查询（倒排索引）

match：会使用分词器解析（先分析文档，然后通过分析的文档进行查询）

keyword类型不会被分词器解析，text类型会被分词器解析

```
GET testdb/_search
{
	"query": {
		"term": {
			"name": "狂"
		}
	}
}
```



## 高亮查询

```
GET kuangshan/user/_search
{
	"query": {
		"match": {
			"name": "狂神"
		}
	},
	"highlight": {
		"pre_tags": "<p class='key' style='color:red'>",
		"post_tags": "</p>",
		"fields": {
			"name": {}
		}
	}
}
```





# 五、集成SpringBoot

- 导入依赖(注意版本)

  ```
          <elasticsearch.version>7.12.1</elasticsearch.version>
          
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
          </dependency>
  ```

  ​


- 导入配置类

  ```
  @Configuration
  public class ElasticSearchClientConfig {

      @Bean
      public RestHighLevelClient restHighLevelClient() {
          RestHighLevelClient client = new RestHighLevelClient(
                  RestClient.builder(
                          new HttpHost("localhost", 9200, "http")
                  )
          );
          return client;
      }

  }
  ```

- 注入配置类

  ```
      @Autowired
      @Qualifier("restHighLevelClient")
      private RestHighLevelClient client;
  ```

- 创建索引

  ```
      //创建索引
      @Test
      void testCreateIndex() throws IOException {
          //创建索引请求
          CreateIndexRequest request = new CreateIndexRequest("jd_goods");
          //执行创建请求,请求后获得响应
          CreateIndexResponse createIndexResponse = client.indices().
                                                   create(request, RequestOptions.DEFAULT);
          System.out.println(createIndexResponse);
      }
  ```

- 获取索引

  ```
      //获取索引，只能判断存不存在
      @Test
      void testExistIndex() throws IOException {
          GetIndexRequest request = new GetIndexRequest("jd_goods");
          boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
          System.out.println(exists);
      }
  ```

- 删除索引

  ```
      //删除索引
      @Test
      void testDeleteIndex() throws IOException {
          DeleteIndexRequest request = new DeleteIndexRequest("xiebaoming");
          //删除
          AcknowledgedResponse delete = client.indices().
                                               delete(request, RequestOptions.DEFAULT);
          System.out.println(delete.isAcknowledged());
      }
  ```

- 添加文档

  ```
   //添加文档
      @Test
      void testAddDocument() throws IOException {
          // 创建对象
          User user = new User("xiebaoming", 3);
          //创建请求
          IndexRequest request = new IndexRequest("xieindex");

          //规则  put /xieindex/_doc/1
          request.id("1");
          request.timeout(TimeValue.timeValueSeconds(1));
          request.timeout("1s");

          //将我们的数据放入请求，json
          request.source(JSON.toJSONString(user), XContentType.JSON);

          //客户端发送请求,获取响应的结果
          IndexResponse index = client.index(request, RequestOptions.DEFAULT);

          System.out.println(index.toString());
          System.out.println(index.status());  //对应命令返回的状态
      }
  ```

- 判断文档是否存在

  ```
      //获取文档，判断是否存在
      @Test
      void testIsExsist() throws IOException {
          GetRequest getRequest = new GetRequest("xieindex", "1");
          // 不获取返回的_source的上下文
          getRequest.fetchSourceContext(new FetchSourceContext(false));
          getRequest.storedFields("_none_");

          boolean exists = client.exists(getRequest, RequestOptions.DEFAULT);
          System.out.println(exists);
      }
  ```

- 获得文档的信息

  ```
      //获得文档的信息
      @Test
      void testGetDocument() throws IOException {

          GetRequest getRequest = new GetRequest("xieindex", "1");
          GetResponse documentFields = client.get(getRequest, RequestOptions.DEFAULT);
          System.out.println(documentFields.getSourceAsString());
          System.out.println(documentFields);
          //返回的全部内容和命令是一样的
      }
  ```

- 更新文档的信息

  ```
      //更新文档的信息
      @Test
      void testUpdateDocument() throws IOException {
          UpdateRequest updateRequest = new UpdateRequest("xieindex", "1");
          updateRequest.timeout("1s");

          User user = new User("xiebaoming666", 18);
          updateRequest.doc(JSON.toJSONString(user), XContentType.JSON);
          UpdateResponse update = client.update(updateRequest, RequestOptions.DEFAULT);
          System.out.println(update.status());
          System.out.println(update);
      }
  ```

- 删除文档记录

  ```
      //删除文档记录
      @Test
      void testDeleteRequest() throws IOException {
          DeleteRequest request = new DeleteRequest("xieindex", "1");
          request.timeout("1s");

          DeleteResponse delete = client.delete(request, RequestOptions.DEFAULT);
          System.out.println(delete);
          System.out.println(delete.status());
      }
  ```

- 批量插入数据

  ```
  //批量插入数据
      @Test
      void testBulkRequest() throws IOException {
          BulkRequest bulkRequest = new BulkRequest();
          bulkRequest.timeout("10s");

          ArrayList<User> userList = new ArrayList<>();
          userList.add(new User("xiebaoming1", 3));
          userList.add(new User("xiebaoming2", 3));
          userList.add(new User("xiebaoming3", 3));
          userList.add(new User("xiebaoming4", 3));
          userList.add(new User("xiebaoming5", 3));
          userList.add(new User("xiebaoming6", 3));
          userList.add(new User("xiebaoming7", 3));

          //批处理请求
          for (int i = 0; i < userList.size(); i++) {
              //批量更新和批量删除，就在这里修改对应请求就行
              bulkRequest.add(new IndexRequest("xieindex")
                      .id("" + (i + 1))
                      .source(JSON.toJSONString(userList.get(i)), XContentType.JSON)
              );
          }

          BulkResponse bulk = client.bulk(bulkRequest, RequestOptions.DEFAULT);
          System.out.println(bulk.hasFailures());  //是否失败
      }
  ```

- 复杂查询

  SearchRequest：搜索请求

  SearchSourceBuilder：条件构造
  HighlightBuilder: 构建高亮
  TermQueryBuilder： 构建精确查询
  MatchAllQueryBuilder: 匹配全部

  ```
      //复杂查询
      @Test
      void testSearch() throws IOException {
          SearchRequest searchRequest = new SearchRequest("xieindex");

          //构建搜索的条件
          SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

          //查询条件，我们可以使用QueryBuliders工具来实现
          //termQuery表示精确匹配
          TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "xiebaoming2");
          //匹配所有
          //MatchAllQueryBuilder matchAllQueryBuilder = QueryBuilders.matchAllQuery();
          searchSourceBuilder.query(termQueryBuilder);
          searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

          searchRequest.source(searchSourceBuilder);

          SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
          System.out.println(JSON.toJSONString(search.getHits()));
          System.out.println("===================");
          //对象然后获取每一个的值
          for (SearchHit hit : search.getHits().getHits()) {
              System.out.println(hit.getSourceAsMap());
          }
      }
  ```

- 高亮显示

  ```
      // 高亮
              HighlightBuilder highlightBuilder = new HighlightBuilder();
              highlightBuilder.field("title"); // 高亮的字段
              highlightBuilder.requireFieldMatch(false); // 只高亮一次(有多个关键字,只高亮第一个)
              highlightBuilder.preTags("<b style='color:red'>");
              highlightBuilder.postTags("</b>");
              searchSourceBuilder.highlighter(highlightBuilder);
  ```

  ​