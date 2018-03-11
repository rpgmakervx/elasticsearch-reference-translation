>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html)
>* 译者：[code4j](https://github.com/rpgmakervx)


# Delete API

删除 api 可以从指定的索引中通过 id 删除一个 JSON 格式的文档。例子如下：

```
curl -XDELETE 'http://localhost:9200/twitter/tweet/1'
```

删除结果如下：

```json
{
    "_shards" : {
        "total" : 10,
        "failed" : 0,
        "successful" : 10
    },
    "found" : true,
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 2
}
```

## 版本控制

当索引文档时指定路由的时候，删除文档的时候也必须提供路由信息，例如：

> curl -XDELETE 'http://localhost:9200/twitter/tweet/1?routing=kimchy'


上面的例子删除 id 为1的文档，不过是根据用户名路由到的，注意，如果没有指定正确的路由文档将不会被删除。

如果 mapping 中设置了 `_routing` 参数为 `required` 但是删除的时候没有指定路由值，则会抛出一个 `RoutingMissingException` 异常并拒绝请求。

## 父文档

`parent` 参数和 路由参数一样可以配置.

> 注意 删除父文档并不能自动删除子文档。一个删除全部子文档的办法就是使用 `delete-by-query` 插件在子索引上执行删除，前提是子文档都指定了父文档的 id,`_parent` 的格式是 parent_type#parent_id。

当删除子文档时，必须指定父文档 id,否则删除请求会被拒绝并抛出一个 `RoutingMissingException` 异常。

## 自动创建索引
删除操作如果索引不存在的话，会自动创建一个索引（手动创建索引参照[create index API](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/indices-create-index.html)），并且之前没有创建过创建动态类型映射，这里也会动态创建（手动创建映射参照[put mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/indices-put-mapping.html)）

## 分布式

delete 操作会计算一个哈希值指定一个分片 id ，然后 get 请求会被重定向到其中一个等于上述 id 分片集合上。分片集合指的是包括这个 id 的主分片和副本。

## 写一致性
根据该分区中活动分片的数来控制删除操作是否允许执行。阈值可以是 `one`,`quorum`,`all`。参数是 `consistency`,默认是在节点级别的配置 `action.write_consistency`,默认值是 `quorum`。

例如，有一个 N 个分片的索引，包括2个副本，则在这个文档所在所有分片中至少要有2个可用分片在一个分区中操作完。N个分片如果包括1个副本，则需要至少单个可用分片（`one`,`quorum`,`all`同理）

（**译者批注：其实就是保证一个数据所在的所有分片中，要有几个分片成功才能算成功，默认是半数以上**）

## 刷新

可以在 delete 操作设置 `refresh` 参数为 `true` 刷新相关的分片，以确保文档能够搜索到。设置这个参数为 `true` 需要注意验证是否会导致系统负载过高（以及降低索引速度）。
（**译者批注：道理同 get **）

## 超时设置

执行删除操作的那个主分片有可能执行失败，可能的原因比如主分片正在恢复或者后台在进行分片平移。默认情况下删除操作将等待1分钟让分片变为可用，超过则失败并返回一个异常。`timeout` 参数用来指定这个等待时间。下面的例子我们设置它为5分钟：
 
```
curl -XDELETE 'http://localhost:9200/twitter/tweet/1?timeout=5m'
```











