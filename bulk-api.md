>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-bulk.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-bulk.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# Bulk API

bulk api 可以将多个 index/delete 操作合并成一个单独的操作调用。这样能够极大地提高索引操作速度。

REST API 的接口叫 `_bulk` ,json数据格式如下：

```
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
```

**注意**：最后的一样要包含`\n`换行符。

可以使用的操作有 `index`, `create`, `delete` 和 `update`。`index` 和 `create`请求的第二行需要一个 source请求体，并且和 index API 一样包括一个 `op_type` (**译者批注：就是需要指明操作类型**)。`delete` 请求下一行不需要source请求体，并且和delete API语法一样。`update` 请求可以包含部分文档，upsert 和脚本等其它选项也可以放到下面一行。

如果你使用文件输入到 `curl` 中，你必须使用 `--data-binary` 类型的请求代替 `plain`。后者不保留换行符。

```
$ cat requests
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
$ curl -s -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
{"took":7,"items":[{"create":{"_index":"test","_type":"type1","_id":"1","_version":1}}]}
```

因为这个格式使用 `\n` 换行符作为分隔符。请确保JSON格式的数据源没有语法格式化。下面是正确的 bulk 命令示例：

```
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "type1", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "index1"} }
{ "doc" : {"field2" : "value2"} }
```

上面的例子中，`update` 操作中的 `doc` 是部分文档内容将会和已存储的文档进行整合。

接口名叫可以用 `/_bulk`, `/{index}/_bulk`, 和`{index}/{type}/_bulk`，当提供了index 或 index/type ，则使用默认的bulk ，不用指明index 和 type。

关于格式的说明。这里的想法是尽可能快地处理请求。因为一些请求将被重定向到其他节点上的其他分片，所以在接收节点侧只解析action_meta_data。

使用这个协议的客户端库应尽量在客户端努力做类似的事情，并尽可能减少缓冲。

批量操作的响应是一个很大的json结构，每个操作的执行结果都是独立的。单独的操作失败了不会影响其余的操作。

批量操作没有一个“完美的”操作次数，你应该尝试不同的设置来找到适合你实际场景的最佳尺寸（**译者批注：就是说批量大小没有一个固定的合理值，需要根据你的应用场景找到合理的批量大小**）。

如果使用 http 协议的 API，确保客户端不会发送 HTTP 块，否则会使响应变慢。

## 版本

每个批量操作的元素都可以使用 `_version/version` 字段来设置版本值。请求操作会自动根据映射的 `_version` 字段执行索引/删除操作。同样也支持 `version_type` 参数。

## 路由

每个批量操作元素都可以使用 `_routing/routing` 字段来设置路由。请求操作会自动根据映射的 `_routing` 字段执行索引删除操作。

## Parent

每个批量操作的元素都可以使用 `_parent/parent` 字段来设置父文档。请求操作会自动根据映射的 `parent` 字段执行索引/删除操作。

## 写一致性

当发起批量调用的时候，你可以通过 `consistency` 参数要求最少多少个分片活跃。支持的参数值有：`one` , `quorum` , `all`。默认是通过节点级参数 `action.write_consistency` 来配置，默认值是`quorum`。

例如，索引有N个分片和2个副本，那么至少有两个分片活跃操作才能成功。如果一个索引有N个分片1个副本，那么需要单个分片活跃（这种情况下 `quorum` 和`one` 是一样的）


## Refresh

批量操作可以设置 refresh 参数为 true ,这样每次批量操作后就能执行一次 refresh，从而数据能够被立马检索到，不用等待刷新周期。该参数设置为 true会增加系统开销，可能会降低索引速度。鉴于这种昂贵的开销，建议将这个参数设置到批量请求级别而不是批量元素级别（**译者批注：源码中可以看到，批量操作实际上是循环索引每个 item ，如果每个元素都refresh，开销会很高**）。

## Update

当批量操作有更新时，可以给更新操作设置 `_retry_on_conflict` 参数，以指定在版本冲突的情况下应该重试更新的次数。

update 操作可以支持 `doc`, `upsert`, `doc_as_upsert`, `script` 还有 `fields`，详细参数参数见 update 篇。curl 例子如下：

```
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
{ "script" : { "inline": "ctx._source.counter += param1", "lang" : "js", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_type" : "type1", "_index" : "index1", "fields" : ["_source"]} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_type" : "type1", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "fields": ["_source"]}
```





