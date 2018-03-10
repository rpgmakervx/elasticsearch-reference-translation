>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 索引API
搜索API可以添加或修改一个 JSON 格式的文档到指定索引中，让它能被搜索到。一下例子插入了一个 ID 为1的 JSON 文档到 "twitter" 索引下的 tweet 类型中。

```json
PUT /twitter/tweet/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

索引操作结果如下：

```json
{
    "_shards" : {
        "total" : 10,
        "failed" : 0,
        "successful" : 10
    },
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 1,
    "created" : true
}
```

`_shard`头提供了索引操作过程的复制信息
- `total` - 表明索引操作在几个分片（主分片和副本分片）上执行了
- `successful` - 表明索引操作在几个分片上操作成功
- `failures` - 索引操作失败的分片数量

`successful` 至少为1索引才算插入成功。

（**译者批注：所有的增删改操作都必须要在主分片上执行，每个主分片维护了一组元数据，能够让我们找到某个分片在哪个具体的节点，当一个文档在一个主分片上操作完成后，请求会被同步到这个分片在其他节点上的副本**）

>注意：副本可能还没有执行完就会返回 successful （默认半数以上执行完认为成功）。这种情况下，`total` 将会返回所有的主分片和副本，successful 将会返回所有执行成功的分片（主分片加副本）。如果没有错误发生，`failed` 等于0。

## 索引自动创建
如果插入操作的索引不存在则会自动创建（手动创建索引查看[创建索引 API](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/indices-create-index.html "Create Index")），并会自动创建 type 的映射到指定 type 下（手动创建映射，查看 [创建映射API](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/indices-put-mapping.html "Put Mapping") ）。

映射本身十分灵活而且是无模式的。新的字段和对象将会加入指定 type 下的映射的定义中。映射定义详见  [mapping](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/mapping.html "Mapping")章节。

动态索引创建可以通过在配置文件中配置  `action.auto_create_index` 为 `false` 来禁用。动态映射创建可以通过配置 `index.mapper.dynamic` 为 `false` 来禁用（或者设置指定的索引的配置）。

动态创建可以基于一个黑白名单指定模式，例如，配置  `action.auto_create_index` 的值为 `+aaa*`,`-bbb*`,`+ccc*`,`-*`等（`+`表示允许，`-`表示不允许）。

## 版本
每个文档都有一个版本号。版本号作为 index API 响应的一部分返回。当指定 `version` 参数的时候，index API 使用乐观锁并发控制，这能控制操作文档的版本。一个不错的使用版本的方法是使用读后再更新的事务策略。指定最初读取文档的版本，能够确保在此期间不会发生改变。（当更新前读取的时候，建议指定 `preference` 值为 `_primary`）。例如:

```json
PUT /twitter/tweet/1?version=2
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

**注意**：版本是完全实时的，并不受搜索操作的准实时方面的影响。如果没有提供版本，则不进行版本检查。

默认的，内部版本号从1开始，每次修改删除操作加1。版本可以由外部提供（例如在数据库保持一份）。为了支持这个功能，`version_type` 参数要指定为 `external`，版本值必须是大于等于0的数字或long类型。而且要小于 `9.2*10^18`。当使用外部版本类型时，不是匹配版本号是否一致，而是系统检查索引请求的版本号是否大于当前存储文档的版本，如果大于当前版本，则文档入索引并使用新版本；如果小于当前版本，则会触发版本冲突异常，并且当前请求失败。

（**译者批注：外部版本可以使用数据库记录的数据中的某些值，比如自增的id，或者数据库再记录一个插入时间戳。数据库的每个操作类似一个
 elasticsearch 的操作日志，当出现同一个记录并发操作可能导致冲突的时候，更新带上版本号，这样就算新的操作再旧的操作后面，也不会被就的操作覆盖**）

> 警告：外部版本号支持以0开始。这样就支持使用从0开始的外部版本系统同步到索引中。当然副作用就是只要是版本号为0的文档，既不可以通过 [Update-By-Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html "Update By Query API")更新，也不可以通过 [Delete By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html "Delete By Query API")删除。

好的作用是只要外部版本数据源使用数据库，则异步的索引操作对于数据库数据源的变化，不用保持严格的排序。如果使用外部版本，即使是来自数据库的更新，索引更新也很简单,因为如果索引操作因为任何原因出问题，它将使用最新版本。

（**译者批注：此处翻译比较拗口，大概意思就是通过外部版本控制，不会导致数据库同步过程中出现并发冲突，及时出现版本不对的情况下，你的数据还是es里面最新的那个版本。**）

## 版本控制类型

紧接着上面介绍的内部版本和外部版本控制，Elasticsearch 还根据指定的业务场景支持其他的版本控制类型。以下是这些不同类型的版本控制以及他们的含义。
- `internal` - 只有给定版本等于当前版本的时候才索引文档。
- `external` & `external_gt`只有当给定版本严格大于存储的当前版本，或者文档不存在的时候，文档才会用指定版本作为当前版本并存储新的文档，给定版本不能是负数。
- `external_gte` - 和 `external`的不同在于给定版本等于当前版本也能生效。

> 注意：`external_gte` 类型的版本控制用于特殊业务场景，要小心使用。使用不当会丢数据。还有一个选项：`force`，不过已经废弃了，因为会导致住副本分片数据不一致

## 操作类型

索引操作也可以接受一个 `op_type` 参数，可用来强制执行一个 `create` 操作，支持 `put-if-absent` 的行为（CAS）。当使用  `create` 操作时，如果文档存在，则索引操作将失败。
下面是使用 `op_type` 参数的例子：

```json
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

另一个使用 `create` 操作的方法如下：

```json
PUT twitter/_doc/1/_create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```


## ID自动生成

搜索操作可以不指定id，下面的这个例子，id将会自动创建。另外，`op_type` 将默认设置为 `create`。下面是例子（注意使用 POST请求）：

```json
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

一下是索引操作的返回结果：

```json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "6a8ca01c-7896-48e9-81cc-9f70661fcb32",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result": "created"
}
```
 
## 路由

默认情况下，分片的位置 —— 或者说路由 —— 使用文档id的哈希值控制。路由值还可以通过 `routing` 参数来指明。例如：

```json
POST twitter/tweet?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上面的例子中，文档根据  `routing` 参数值 "kimchy" 路由到某个分片。

当设置过映射后，`_routing` 字段用来指定索引操作到那个分片，并且路由值可以从文档中提取，这个方式会在解析文档时带来一点开销（非常少），如果  `_routing` 在映射的时候指定，并且设置了  `required`，当索引操作的文档对应的字段不存在时，操作会失败。

（**译者批注：2.0.0 以后就废弃的属性不做翻译了**）

## 父子关系

一个子文档在插入的时候可以指定它的父文档。例如：

```json
PUT /blogs/blog_tag/1122?parent=1111
{
    "tag" : "something"
}
```

当索引子文档的时候，路由值会自动和父文档保持一致，除非子文档在参数中指定了 `routing` 参数。

##  分布式

索引操作会根据路由信息被分发到主分片并在主分片所在节点执行。当主节点执行完毕，请求会被分发到可用到副本分片上。

## 写一致性

为确保写操作不在网络分区时发生错误，默认情况下爱，索引操作只有当半数以上的分片（> replicas / 2 + 1）操作成功后才会返回成功。这个配置可以通过 `action.write_consistency` 选项覆盖。如果想一步操作修改这个参数，可以使用 `consistency` 请求。

合法的参数值有：`one` , `quorum` , `all`。

注意副本为1的情况（总共只有两个分片），此时默认认为主分片写成功才算成功。

索引操作仅在所有**可用**分片，包括副本写入文档后才返回（使用同步副本方式）。

（**译者批注：使用异步方式，不等副本写完就可以返回，不会不建议这么做，特别是数据一致性高的场景，而且可能会使 ES 过载**）

## 刷新

为了能够让文档能在搜索结果中立刻被查看，要在索引操作后立马刷新分片（不是整个索引），可以设置 `refresh` 为 `true` 实现这点。设置这个需要你仔细思考并确认会不会导致索引和搜索的性能问题。注意，使用 get `API`是完全实时的，不需要刷新。

## 空的更新
当你创建文档的时候，总是会给文档增加一个新版本，哪怕是这个请求没有对文档做出改变。如果使用 `_update` 你不想让它这样的话，可以设置 `detect_noop` 为 `true` 。这个操作在 `index` API 是无效的，因为 `index` 请求并不会先获得旧的文档的source，也就没法比较文档是否相同。

没有一个硬性规则要求使用 noop 这个特性。这个选择包含了很多因素，例如你更新文档出现 noop 的频率，以及你在更新频率高的分片上查询的QPS等等。

## 超时
执行索引操作的那个主分片有可能执行失败，可能的原因比如主分片正在恢复或者后台在进行分片平移。默认情况下索引操作将等待1分钟让分片变为可用，超过则失败并返回一个异常。`timeout` 参数用来指定这个等待时间。下面的例子我们设置它为5分钟：

```json
PUT /twitter/tweet/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

（**译者批注：其实每种请求 API 理论上都是有请求超时限制的，建议给你的所有请求评估一个超时时间，一面部分请求超时阻塞请求队列导致其他请求也一起被阻塞，可以把这个时间做成动态可配置的，达到熔断效果**）