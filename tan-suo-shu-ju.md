>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/_exploring_your_data.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/_exploring_your_data.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 探索数据

## 样本数据

现在我们已经知道一些基础的东西了，接下来我们用更真实的数据来试试。我已经准备了一份虚假银行客户账户的样本数据，每个文档结构如下：
```json
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```
出于好奇，数据使用这个网站生成的 [`www.json-generator.com`](http://www.json-generator.com/)，所以请忽略这些数据的实际值和语义，因为他们都是随机生成的。

## 加载样本数据

你可以在[这里](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)下载样本数据 (accounts.json)，把它放到你的目录中，用如下语句把它加载到你的集群中：
>curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/account/_bulk?pretty&refresh" --data-binary "@accounts.json"
curl "localhost:9200/_cat/indices?v"

响应如下：
>health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb        128.6kb

（**译者批注：创建索引时有短暂的状态为yellow或red是正常现象，因为创建索引的时候需要分配分片，如果集群繁忙创建会有一定延迟。**）

响应内容说明我们成功的将1000条文档索引到了银行索引中（在 `_doc` 这个 type 下）

***

# 搜索API

现在我们尝试一些简单的搜索，有两个基础的方法执行搜索：一种是将通过 [REST request URI](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/search-uri-request.html "URI Search")发送搜索参数，另一种是通过[REST request body](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/search-request-body.html "Request Body Search")发送搜索参数。请求体这种方法能让你的请求更具有表现力并且也需要你使用更可读的JSON格式。这里我们用一个示例演示URI 的请求方式，但是本教程剩余的部分，我们将只使用请求体这种方式
（**译者批注：请求体灵活性安全性更强，更适合生产环境；测试数据或简单的排查数据可以使用URI的方式，更快捷**）
 
Rest API 的搜索方式使用 `_search` 结尾的方式访问，下面的例子将返回银行索引下的全部文档：
>curl 'localhost:9200/bank/_search?q=*&pretty'

我们来进行第一次分析这个搜索请求。我们搜索（`_search结尾`）银行索引，并且参数 `q=*` 告诉 Elasticsearch 匹配全部文档，之前出现过的这个 `pretty` 参数，仅仅是告诉 Elasticsearch 返回格式化过的JSON。
响应如下：
```json
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0 
},
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "6",
      "_score" : 1.0, "_source" : {"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
    }, {
      "_index" : "bank",
      "_type" : "account",
```

至于响应，我们看到以下部分：
- `took` - Elasticsearch 搜索花销的时间，毫秒单位
- `timed_out ` - 请求是否超时
- `_shards` - 告诉我们搜索了多少个分片，以及搜索成功/失败的分片个数
- `hit` - 搜索结果（默认返回10条）
- `hits.total` - 搜索匹配到的文档总数
- `hit.hits` - 搜索结果实际的数组
- `_score` 和 `max _score` - 先无视这两个参数

下面使用请求体的方式和上面的搜索一样。
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} }
}'

不同的是，我们的 `_search` API使用 POST 请求和 JSON 风格的请求体替代 URI中的`q=*`。
我们将会在下一章节讨论JSON风格的查询。

响应如下：

```json
{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "6",
      "_score" : 1.0, "_source" : {"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "13",
```

一定要明白，一旦你获得了搜索结果，这个请求在 Elasticsearch 中将完全执行完毕并且不会保存任何形式的服务端资源，并且你的搜索结果中也不会有游标等。这一点和其他使用SQL的存储引擎形成了鲜明的对比，在这些引擎中，可能你先获取查询结果的子集，然后如果你使用了带状态的服务端游标，需要接着返回服务器获取剩下结果。


***

# 查询语句介绍

Elasticsearch 提供了基于JSON风格的特定查询语法，供你执行查询操作。在 [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl.html "Query DSL")
中有参考。查询语法十分详细可能第一次看会被吓到，不过最好的学习方法是从几个基本的例子开始：

回到上一个例子，我们执行了如下查询：
```json
{
  "query": { "match_all": {} }
}
```

分析上面的语句，`query` 部分告诉我们查询定义了什么，`match_all` 是我们想用的查询类型。`match_all` 就是搜索指定索引下的全部文档。

除了 `query` 参数我们还可以传递其他的参数影响搜索结果。例如下面的`match_all` 查询只返回第一条文档：
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "size": 1
}'

注意 `size` 如果不指定，默认返回10条文档。

下面的例子中，`match_all` 查询返回第11到20条文档：
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}'

`from`（默认是0）参数从第几个文档开始，`size` 参数用来指定从`from` 参数指定的文档号开始返回几条文档。这个特性在实现分页搜索的时候很有用，注意 `from` 不指定的时候默认是0。

下面这个例子执行 `match_all` 查询，并且按照账户余额进行降序排序，并返回10（默认是大小）条文档。
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}'

***

# 执行搜索
现在我们已经见过一些基本的搜索了，下面我们来发掘下更多的查询[DSL](http://blog.csdn.net/u010278882/article/details/50554299)。我们先来看一下搜索结果的字段，默认情况下所有查询都会返回完整的JSON文档，这个（完整的 JSON 文档）被称作 源（搜索命中结果的 `_source` 字段），如果我们不想返回整个文档，我们可以只从其中请求几个字段。

下面的例子告诉我们搜索结果如何只返回 `account_number` 和  `balance` 两个字段。

>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}'

注意上面的例子我们只是减少了_source字段，返回结果仍然有 `_source` 但是里面只包含 `account_number` 和 `balance` 两个字段。

如果你有使用SQL语句的背景，上面的例子有点类似SQL中的 `SELECT FROM` 中的字段列表的概念。

下面我们继续查询部分。之前我们已经知道如何使用 `match_all` 获取索引全部文档，那么现在来介绍一种新查询，叫[`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-match-query.html "Match Query")，可以说是最基本的基于字段的搜索（针对特定字段或字段集的搜索）。

下面的例子返回账户号为20的文档：
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "account_number": 20 } }
}'

下面的例子返回地址中包含`mill`的全部文档。
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill" } }
}'

下面的例子返回地址中包含 `mill` 或 `lane` 的全部文档。
>curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill lane" } }
}'

（**译者批注：后面的查询语句都省略开头的 command，大家只需要关注 query 体即可**）

下面的例子是 `match` 查询的变种（`match_phrase`），返回地址字段包含词组 `mill lane` 的全部文档：

```json
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

下面来介绍[`bool`(ean) query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-bool-query.html "Bool Query")（**译者批注：布尔查询**）。布尔查询允许我们通过布尔逻辑将小的查询组合成大的查询。

下面的例子组合了两个 `match` 查询，返回地址字段包含 `mill` 和  `lane` 的全部文档。

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```


上面的例子中，`bool must` 部分指定了所有查询条件都必须返回true，文档才算命中。

相反的，下面的例子组合了两个 `match` 查询，返回地址中包含 `mill` 或 `lane` 的全部文档。
```json
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

上面的例子，`bool should` 部分指定了一组查询任意一个条件是true才能命中文档。

下面的例子组合了两个 `match` 查询，返回文档的地址字段中既不包括`mill` 也不包括 `lane`。

```json
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

上面的例子中，`bool must_not`部分指定了一组查询任意一个条件都不为true才能命中文档（**译者批注：简单说就是全部条件都是false**）。

我们还可以同时组合`must` ，`should`，`must_not`这几个逻辑到一个布尔查询中，进一步说，我们还能组合多个布尔查询到其他任何布尔查询中，从而模拟复杂的多级布尔逻辑

（**译者批注：类似 SQL 语句中 AND OR 多层嵌套，只不过 JSON 表达比较麻烦，建议逻辑不要过于复杂，否则当你排查业务逻辑问题时看到N层 query dsl 会崩溃。。。**）

下面的例子返回年龄是40岁但是不住在 ID 州的账户：

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

***

# 执行过滤

之前的章节我们跳过了一个小细节，叫文档得分（搜索结果中的 `_score` 字段）。得分是一个数值类型的值，他能表示文档同我们指定的搜索条件的匹配度。得分越高，文档相关度越高，繁殖文档相关度越低。

但是查询也并不总是需要产出得分，特别是当他们只用于"过滤"文档集合的时候，Elasticsearch 检测到这些情况并自动优化了查询执行过程避免了无用的评分计算。

前一章我们介绍的布尔查询也支持 `filter` 短语，可以用来限制被其他短语匹配到，并且不改变原有的得分计算（**译者批注：说白了就是不影响评分**）。让我们用一个例子介绍下 [`range` query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-range-query.html "Range Query")，可以让我们按照范围过滤。这个查询通常用在数值或日期类型的过滤。

下面的例子使用布尔查询，过滤 余额 在2000-3000 闭区间内的所有文档，换句话说，我们要找到余额大于等于2000且小于等于3000的账户。

```json
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

分析上述例子，布尔查询包括一个 `match_all` 查询（查询部分）还有一个范围查询（过滤部分）。我们可以用其他任意查询替换查询和过滤部分。例子中范围查询命中的文档最有意义，没有其他方式命中的文档比它更相关
（**译者批注：意思就是过滤命中的相关度是最高的，毕竟属于精准匹配**）

除了 `match_all`，`match`，`bool`，`range` 几种查询方式，还有许多查询类型可用，在这里不一一深究，鉴于我们已经对查询有了基本的理解，所以将这些知识点运用到学习和实验其他类型的查询中应该不会太难。

***

# 执行聚合

聚合提供能从数据中分组和提取统计的功能。最简单的理解方法就是认为他们和SQL中的 `GROUP BY` 和 SQL 中的聚合方法大致相同。在 Elasticsearch 中，你可以在执行一次搜索的结果中返回命中文档并同时在命中结果集外返回聚合结果。这个功能从某种意义上说十分有效，你可以通过简洁的 API 一次性返回查询结果和聚合结果，从而避免了多次网络 IO。

我们以统计账户信息中各个州有多少账户开始，并按照州的字母顺序倒序排序（默认就是这样），返回10条聚合结果。

```json
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      }
    }
  }
}
```

SQL 语句中和下面类似：
>SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC

响应如下：

```json
{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
"hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "buckets" : [ {
        "key" : "al",
        "doc_count" : 21
      }, {
        "key" : "tx",
        "doc_count" : 17
      }, {
        "key" : "id",
        "doc_count" : 15
      }, {
        "key" : "ma",
        "doc_count" : 15
      }, {
        "key" : "md",
        "doc_count" : 15
      }, {
        "key" : "pa",
        "doc_count" : 15
      }, {
        "key" : "dc",
        "doc_count" : 14
      }, {
        "key" : "me",
        "doc_count" : 14
      }, {
        "key" : "mo",
        "doc_count" : 14
      }, {
        "key" : "nd",
        "doc_count" : 14
      } ]
    }
  }
}

```

我们能看到，al州有21个账户，其次是 tx 有17个，其次是id有15个，等等。

注意我们设置 `size` 为0，因为我们只想关注响应中的聚合结果。

基于之前的聚合例子，下面的例子还计算了每个州账户的平均余额。

```json
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

注意我们是如何在 `group_by_state` 聚合体中嵌套 `average_balance` 聚合体的，这是一个公共的聚合模式，你可以嵌套任意聚合体改变聚合结果，从而实现你的需求。

基于上面的聚合例子，我们再根据平均余额进行倒序排序：

```json
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

下面的例子展示了如何根据年龄段（20-29, 30-39, 和 40-49）聚合，其次是性别，最后计算它们的平均余额，分别每个年龄段每个性别一组结果。

```json
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```

还有很多聚合方法在这里不做深究，如果你想进一步研究，[聚合参考指导](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/search-aggregations.html)是个不错的开始。

***

# 总结

Elasticsearch 既简单又复杂，到目前为止我们已经基本了解了它是什么，他的一些内部机制，以及如何使用 REST API 来操作它。希望这个教程能够让你更好地理解 Elasticsearch，更重要的是，能够启发你去实验 ES 中更多地特性。
