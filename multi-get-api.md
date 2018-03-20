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

`mget` 也可以用在一个索引上（这样请求体就无需指定索引名了）：

```
curl 'localhost:9200/test/_mget' -d '{
    "docs" : [
        {
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_type" : "type",
            "_id" : "2"
        }
    ]
}'
```

加上type：

```
curl 'localhost:9200/test/type/_mget' -d '{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}'
```

在这种情况下，直接使用 `ids` 元素可以简化请求体：

```
curl 'localhost:9200/test/type/_mget' -d '{
    "ids" : ["1", "2"]
}'
```

## 可选的Type

mget API 中的参数 `_type` 是可选的，设置 `_all` 或者不设置该参数将返回所有type中第一个匹配的文档。（**译者批注：言外之意就是每个文档只出现一次**）

如果 type 没有指定并且有多个文档id相同（_type不同），那么将只返回第一个命中的文档。

例如，文档1出现在 typeA 和 typeB 中，下面的查询将会把同样的文档返回两次给你：

```
curl 'localhost:9200/test/_mget' -d '{
    "ids" : ["1", "1"]
}'
```
你需要明确指定 `_type` :

```
GET /test/_mget/
{
  "docs" : [
        {
            "_type":"typeA",
            "_id" : "1"
        },
        {
            "_type":"typeB",
            "_id" : "1"
        }
    ]
}
```

## source过滤

默认情况，每个文档都会返回 `_source` 字段（该字段被存储）。和  [get](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-get.html#get-source-filtering "Source filteringedit") API 类似，你可以使用 `_source` 参数来指定 `_source` 特定的字段。你也可以使用 url 参数 `_source,_source_include & _source_exclude` 来根据情况指定。

例如：

```
curl 'localhost:9200/_mget' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}'
```

##  Fields

指定 `store` 字段能够返回文档特定字段，和 GET API 中的 [fields](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-get.html#get-fields "Fieldsedit")字段类似，例如：

```
curl 'localhost:9200/_mget' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "fields" : ["field3", "field4"]
        }
    ]
}'
```

`fields` 字段可以在url中指定，默认返回文档全部内容：

```
curl 'localhost:9200/test/type/_mget?fields=field1,field2' -d '{
    "docs" : [
        {
            "_id" : "1" 
        },
        {
            "_id" : "2",
            "fields" : ["field3", "field4"] 
        }
    ]
}'
```

上面的结果文档1中获得字段 field1 field2; 文档2获得field3 field4。

## Routing

你可以指定路由参数：

```
curl 'localhost:9200/_mget?routing=key1' -d '{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}'
```
上面的例子，文档2将通过 `key1` 寻找对应的分片，但是文档1将通过 `key2` 寻找对应的分片。




