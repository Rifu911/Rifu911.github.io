---
title: ElasticSearch 快速入门
date: 2018-10-17 15:18:30
categories: 学习笔记
tags: [分词器, 搜索引擎, ElasticSearch]
---

#### 环境搭建

1.安装

docker pull registry.docker-cn.com/library/elasticsearch

<!-- more -->

2.run

由于ElasticSearch 默认使用的堆内存为2G，在这里需要限制其使用内存的大小

docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p9300:9300 --name elasticSearch01  IMAGE-ID

以上步骤完成之后，可在web页面访问对应ip的9200端口，出现返回了json数据为成功

![](D:\Github\source\_posts\ElasticSearch-快速入门\微信截图_20181017152618.png)

#### 使用(https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

> http://ip:9200/yourIndex/docName/key

###### 1）.添加数据

http请求方式为put, 数据类型为源数据 ,也就是row

> {
>     "first_name" : "rifu",
>     "last_name" : "rifu",
>     "age" : 25,
>     "interests" : ["basketball","tableball"]
> }

返回结果：结果码201

> {
>     "_index":"zava",
>     "_type":"user",
>     "_id":"1",
>     "_version":1,
>     "result":"created",
>     "_shards":{
>         "total":2,
>         "successful":1,
>         "failed":0
>     },
>     "created":true
> }

###### 2）.获取数据

http请求方式为get

###### 3）.删除数据

http请求方式为delete

###### 4）.检查数据是否存在

http请求方式为head

数据存在响应码为200，不存在则为404

###### 5）.查询所有的数据

get方式

>  ip:9200/yourIndex/docName/_search

###### 6）.带条件的查询

请求方式为post,数据放在请求体里面

> ip:9200/yourIndex/docName/_search
>
>
>
> {
>
> "query" : {
>
> 	"match"：{
>
> 		"last_name" : "rifu"
>
> ​	}
>
> }
>
> }





#### SpringBoot使用ElasticSearch

一共有两种方式：

1.使用Jest

在配置文件里面：

> spring.elasticsearch.jest.uris=http://120.77.82.58:9200

```java
<!-- https://mvnrepository.com/artifact/io.searchbox/jest -->
<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>5.3.3</version>
</dependency>
```

2.使用SpringData集成的elasticsearch

在配置文件里面：

> spring.data.elasticsearch.cluster-name=elasticsearch
> spring.data.elasticsearch.cluster-nodes=120.77.82.58:9300

使用springdata-elasticsearch需要注意版本适配问题

> | spring data elasticsearch | elasticsearch |
> | ------------------------- | ------------- |
> | 3.1.x                     | 6.2.2         |
> | 3.0.x                     | 5.5.0         |
> | 2.1.x                     | 2.4.0         |
> | 2.0.x                     | 2.2.0         |
> | 1.3.x                     | 1.5.2         |

```java
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```



##### 1.使用JestClient

```java
	/**
     * 测试保存数据到elasticsearch
     */
    @Test
    public void testSaveData() {
        Msg msg = new Msg();
        msg.setId(1);
        msg.setSender("rifu");
        msg.setReceiver("lili");
        msg.setContent(" i love you");

        Index index = new Index.Builder(msg).index("zava").type("msg").build();

        try {
            jestClient.execute(index);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    /**
     * 测试查询数据
     */
    @Test
    public void testSelectData() {
        String json = "{\n" +
                "    \"query\": {\n" +
                "        \"match\": {\n" +
                "            \"content\": \"love\"\n" +
                "        }\n" +
                "    }\n" +
                "}";
        Search search = new Search.Builder(json).addIndex("zava").addType("msg").build();
        try {
            SearchResult result = jestClient.execute(search);
            System.out.println(result.getJsonString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

##### 2.使用Springdata-ElasticSearch

首先要在对应的实体类上标注

> @Document(indexName = "zava",type = "article")

```java
 /**
     * 保存数据到ES
     */
    @Test
    public void testSaveArticle(){
        Article article = new Article();
        article.setId(1);
        article.setAuthor("rifu");
        article.setTitle("天方夜谭");

        articleRepository.index(article);
    }
```

