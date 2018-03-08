>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/_modifying_your_data.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/_modifying_your_data.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 修改数据
Elasticsearch提供近实时的数据操作和搜索功能。默认情况下，数据从写入/更新到被检索到需要1s（刷新间隔配置的时间）左右。这是和其他存储引擎一个很重要的不同，像数据库中的数据，事务执行完后数据就立马可见。

### 索引/替换文档
我们之前已经知道如何创建一个文档了，在执行一次：
>curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "John Doe"
}'

上面将指定文档索引到 customer 索引的 externaltype 中，id为1。如果我们再次执行上面的命令，用不同或相同的文档，Elasticsearch 将会用新的文档覆盖上面的现有的id为1的那个文档（或称作`reindex`）。
>curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "Jane Doe"
}'

上面的命令将id为1的文档名从 "John Doe" 改为了 "Jane Doe"。另一方面，如果我们使用不同的id，将会创建一个新的文档到索引中，原有的文档将不会被改变。
>curl -XPUT 'localhost:9200/customer/external/2?pretty' -d '
{
  "name": "Jane Doe"
}'

上面的命令索引了一个ID为2的新文档。
索引的时候，ID参数是可选的，如果没有指定，Elasticsearch 将会生成一个随机的ID并用他作为文档 ID，ES 生成的 ID（或者我们指定了的ID）将会作为调用API的返回结果返回。
下面的例子告诉我们如何不指定ID创建文档。
>curl -XPOST 'localhost:9200/customer/external?pretty' -d '
{
  "name": "Jane Doe"
}'

注意，上面的例子因为没有指定ID，所以我们用的是POST请求，而不是PUT
（**译者批注：实际使用中译者使用POST和PUT都可以实现插入更新，并没有发现什么不同，实际上源码里也能看到，创建文档的handler里面能同时处理POST和PUT两类请求**）
***
## 更新文档
除了新建或覆盖文档，我们还能修改文档。但是要注意 Elasticsearch 底层并不是直接在原来的数据上做更新。每当我们执行更新操作，ES删除老文档然后创建新文档，这两步是在一次更新请求中完成的。
下面的例子展示如何修改之前那个ID为1的文档，修改名字为”Jane Doe“。
>curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
{
  "doc": { "name": "Jane Doe" }
}'

下面的例子展示如何修改文档的名字为 ”Jane Doe“ 并增加一个 age 字段：
>curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
{
  "doc": { "name": "Jane Doe", "age": 20 }
}'

更新操作也可以使用简单的脚本。注意，像下面这种动态脚本在1.4.3版本默认是禁止的，细节请看[scripting docs](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/modules-scripting.html "Scripting")。这个脚本的作用是给年龄加5：
>curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
{
  "script" : "ctx._source.age += 5"
}'

（**译者批注：脚本功能十分强大，但是并不推荐使用，特别是查询。类似关系型数据库的存储过程，虽然能处理复杂业务逻辑但是把过于复杂的逻辑置于ES引擎对性能是一种损耗，ES默认禁用这个特性也是有道理的。**）

上面的例子中，`ctx._source`引用的是你要更新的文档（也就是ID为1的文档）。
注意一次只能修改一个文档，未来版本可能会提供批量按条件修改的API，类似SQL中的 update where条件。
***
# 删除文档
删除文档相当简单，下面的例子表示删除ID为2的文档：
>DELETE /customer/doc/2?pretty

删除同样也提供了批量删除API（插件），不过值得一提的是，如果需要清空数据，删除整个索引比批量删除所有文档效率更高。

（**译者批注：关于批量修改，还有批量删除，都有对应的插件，但实际上也是使用滚动查询然后一条条删除，执行速度十分慢，若该过程执行较长，服务层等待队列可能会出现请求超时，因此该特性建议在测试环境，或命令行模式流量低峰期使用，线上接口不推荐提供。推荐使用批量更新或删除接口，或多起几个线程单条删除**。）

***

# 批量操作
除了增删改单个文档之外，以上操作ES还提供了批量执行： [`_bulk` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/docs-bulk.html)，批量机制十分高效，能利用尽可能少的网络开销来尽可能提升数据处理速度。
示例如下，下面使用一个批量接口索引两个文档：
>POST /customer/doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }

更新第一个文档并删除第二个文档：
>POST /customer/doc/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}

注意第二个删除操作，操作中并不包含源文档信息，因为删除只需要提供ID即可。

批量操作其中一个操作失败了，并不会影响整个操作，如果一个操作失败了其他操作还是会继续执行。当批量执行结束，会得到一个返回响应，里面包括每个操作的状态码信息，你可以通过这个信息观察某个操作是否失败。
