>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-multi-get.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-multi-get.html)
>* 译者：[code4j](https://github.com/rpgmakervx)


# Multi Get API

MULTI GET API 能够基于索引名，没醒(可选)和id获取多个文档。响应结果包括一个包含了所有获取结果的 `docs` 数组，每个元素都和 get api 的返回结果的结构类似，例子如下：

```
curl 'localhost:9200/_mget' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}'
```