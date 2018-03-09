>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/_exploring_your_cluster.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/_exploring_your_cluster.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 探索集群

### REST API
现在我们的集群（节点）已经跑起来了，下一步就是如何与它交互。幸运的是 Elasticsearch 提供了方便理解并且强大的 REST API 供你和 ES 集群进行交互，API 能做的事情如下：
- 检查集群，节点以及索引的健康状况，状态，以及统计信息。
- 管理你的集群，节点，以及索引的数据和元数据
- 提供CURD（Create, Read, Update, 和 Delete）以及对索引的搜索操作。
- 进行一些诸如 分页，排序，脚本，聚合等高阶搜索操作，还有更多其他的。
（**译者批注：为了搜索性能，译者不推荐一些高阶操作例如脚本，把逻辑运算放到引擎中，有点类似关系型数据库的存储过程或者函数，效率并不高，而且使用场景可以通过业务进行优化。**）

#### 集群健康
我们先进行一个基本的健康检查，看下集群如何运作。我们将使用 `curl` 命令工具，但你也可以使用任何能够发起 http/rest 协议请求的工具（**译者批注：Windows 用户也要先安装 curl 工具，或者使用一些浏览器插件，例如火狐的 HttpRequester，或者 chrome 的 sense，我个人推荐 sense，这个是专门为ES定制的**）。假设我们在刚才启动的 ES 节点，我们打开一个新的控制台。
检查集群健康要使用 [`_cat` API](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/cat.html "cat APIs")，记得之前我们节点的端口是`9200``:
>curl 'localhost:9200/_cat/health?v'

响应内容如下：
>epoch           timestamp   cluster          status  node.total node.data shards pri relo init unassign
1394735289   14:28:09    elasticsearch  green               1           1        0        0         0        0            0

可以看出我们的集群 `elasticsearch` 状态是 `green`。
每当我们请求集群健康状态时，我们会得到 `green` , `yellow` 或 `red` 。绿色表示一切 OK （此时集群功能齐全），黄色表示集群的数据齐全，但是有些副本还没有分配（**译者批注：或者说副本丢失**）(此时集群功能齐全)，红色表示无论出于什么原因，部分数据丢失。注意即使集群处于红色状态，他仍旧是部分可用的（搜索请求在可用的分片上仍旧有效），但是你还是需要尽快修复它，毕竟数据不全了。

我们也能从相应中看出，我们有一个节点，由于没有数据我们的分片数是0。注意，因为我们使用默认的集群名 `elasticsearch`，并且因为 Elasticsearch 默认使用单播的节点发现方式发现同台机器上的其他节点，有可能你会无意中启动了多个节点而把他们加入到集群中，这样你可能会在相应中看到不止一个节点。
我们可以通过如下命令得到节点列表：

>curl 'localhost:9200/_cat/nodes?v'

响应如下：

>host         ip        heap.percent ram.percent load node.role master name
mwubuntu1    127.0.1.1            8           4 0.00 d         *      New Goblin

我们可以看到名为 `New Goblin` 的节点，集群中唯一的节点。


#### 展示全部节点
一起来看下我们的索引信息
>curl 'localhost:9200/_cat/indices?v'

响应结果：
> health index pri rep docs.count docs.deleted store.size pri.store.size

很明显这代表我们的集群中没有索引


#### 创建索引

现在我们创建一个名为 `customer` 的索引，然后再看下索引列表：
>curl -XPUT 'localhost:9200/customer?pretty'
curl 'localhost:9200/_cat/indices?v'

第一条命令使用 PUT 请求创建一个 "customer" 索引，后面加上 `pretty` 参数是为了让响应的 json 格式化输出。
响应分别如下:

```json 
{
  "acknowledged" : true
}
```

> health index    pri rep docs.count docs.deleted store.size pri.store.size
yellow customer   5   1          0            0       495b           495b

第二条命令我们知道我们有一个 customer 索引，它包含5个主分片和1个副本（默认配置），并且索引不包含文档。

可能你也注意到了，集群状态现在被标注了黄色，之前讨论过黄色表示存在副本未分配，该现象的原因是 ES 默认为这个索引创建一个副本，由于我们集群只有一个节点，为保证高可用，副本无法进行分配，直到有新的节点加入集群才行。一旦副本能够分配到第二个节点上，节点的健康状态就会切换成绿色。

#### 索引并查询文档
现在我们写点数据到我们的 customer 索引中，记得我们之前说过，文档插入索引需要指定索引的类型。
现在插入一个简单的文档，类型叫 `external`,ID为1，json 文档为：{ "name": "John Doe" }，命令如下：
>curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "John Doe"
}'

响应如下：
>{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "created" : true
}

上面可以看出我们在 customer 索引的 external 类型中创建了一个文档，文档有一个内部 id，在索引期间指定的。

有个关键点需要注意，在你创建文档到某个索引的时候，ES并不强求你显示的先创建索引，如果 customer 索引不存在，ES 会自动创建。

下面我们取回刚才创建的文档：
>curl -XGET 'localhost:9200/customer/external/1?pretty'

响应如下：

```json
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true, "_source" : { "name": "John Doe" }
}
```

除了一个 `found` 字段，它表示我们得到了一个id为1的文档，以及另一个字段 `_source` ，它包含了我们上一步索引的文档完整内容之外，其他没有什么特别的。

#### 删除索引

现在我们删除刚才创建的索引，并再次列出全部索引
>curl -XDELETE 'localhost:9200/customer?pretty'
curl 'localhost:9200/_cat/indices?v'

响应分别如下：

```json
{
  "acknowledged" : true
}
```

>health index pri rep docs.count docs.deleted store.size pri.store.size

表明我们成功删除了一个索引，现在集群恢复到了刚才启动时候的状态。

进行下一步之前，我们再仔细看下之前学习的 API：
>curl -XPUT 'localhost:9200/customer'
curl -XPUT 'localhost:9200/customer/external/1' -d '
{
  "name": "John Doe"
}'
curl 'localhost:9200/customer/external/1'
curl -XDELETE 'localhost:9200/customer'

如果仔细分析下以上的命令，我们可以发现访问ES中数据的一个模式。这个模式可以被总结为：
>curl -X <REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>

这个REST命令普适于 ES 中全部 API 命令，记住这个会为你学习 ES 开个好头。



