>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-upgrade.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-upgrade.html)
>* 译者：[code4j](https://github.com/rpgmakervx)


# 升级

升级前请注意：

- 参考[版本变化](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/breaking-changes.html "Breaking changes")文档。
- 升级前请在在开发环境测试升级版本。
- 升级前备份数据。不做备份的话是没法回滚到上一版的。

Elasticsearch 通常可以使用滚动升级步骤，服务不会中断。这章详细介绍如何进行滚动升级和停机升级。

当前版本是否支持滚动升级，请参照如下表格：

升级前  | 升级后|支持的升级方式 
--------|-------|--------|
0.90.x|2.x|[停集群升级]()
1.x|2.x|[停集群升级]()
2.x|2.y|[滚动升级]()(y>x)

***

# 备份你的数据
升级之前请经常做备份，这样出了问题你可以回滚。升级有时候还包括 Lucene 版本升级，Lucene 升级后更新的索引可能在原来版本的 Elasticsearch 上跑不起来。

## 备份1.0往后的版本

在1.0往后的版本做备份，使用快照这一特性会方便很多，详细指示看这里：[backup and restore with snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/modules-snapshots.html "Snapshot And Restore")。

### 备份0.9及以前的版本
备份0.90.x版本步骤如下：

##### 第一步：
下面这一步防止备份的时候索引刷新到磁盘：

```json
PUT /_all/_settings
{
  "index": {
    "translog.disable_flush": "true"
   }
}
```

##### 第二步：
下面这步防止集群备份的时候，数据从一个节点平衡到另一个节点。

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "none"
  }
}
```

##### 第三步：备份你的数据
等再平衡和索引刷新禁用后，用你喜欢的方法开始备份数据（打成 tar 包，使用存储快照或者备份软件）

##### 第四步：重新打开再平衡和索引刷新
当备份完成，不需要再从 Elasticsearch 读取数据后，必须要重新打开再平衡和索引刷新：

```json
PUT /_all/_settings
{
  "index": {
    "translog.disable_flush": "false"
  }
}

PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "all"
  }
}
```

***
# Elasticsearch 滚动升级

滚动升级允许 Elasticsearch 集群在业务不中断的情况下更新一个节点。集群中不支持同时运行多个版本，因为分片不会从新版本分配到旧版本的节点上。

从这个列表[table](setup-upgrade.html "Upgrading")中检查当前版本的ES是否支持滚动升级。

滚动升级步骤如下：


#### 第一步: 滚动升级步骤如下：

当你关闭一个节点之后，分片分配进程会尝试立即将分片从当前节点复制到集群上的其他节点，导致浪费了大量的 I/O 操作。要想避免这个问题可以通过在关闭节点前禁用这个进程

```
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "none"
  }
}
```

#### 第二步：停止不必要的索引，并执行同步刷新（这一步可选）


你可以在升级期间愉快的进行索引操作，当然，如果你使用如下命令，停止不必要的索引，并执行同步刷新[synced-flush](indices-synced-flush.html "Synced Flush")请求，分片恢复速度会更快：

```
POST /_flush/synced
```

同步刷新是锦上添花的操作。如果出现索引挂起的现象操作就会失败，为了安全起见有必要多试几次。

#### 第三步：单个节点停机并升级

**升级前**关闭一个节点。


> 注意：当使用 zip 或 tar 包升级，默认情况下 Elasticsearch home 目录下的 config，data，log，plugins 等目录都会被覆盖。
最好解压到不同的目录，这样升级期间就不会删除原来的目录了。自定义的目录可以通过 path.conf 和 path.data 来[设置](setup-configuration.html#paths "Pathsedit")。
 RPM 或 DEB 包会把目录放到 [合适的位置](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-dir-layout.html "Directory Layout")


使用 [rpm/deb](setup-repositories.html "Repositories") ) 安装包升级：

*  使用 `rpm` 或 `dpkg` 安装新包，所有的目录都会被放到合理的位置，配置文件不会被覆盖。

使用zip或tar包解压安装：

*   解压安装包，确保不要覆盖 `config` 和 `data` 目录。
*   从旧的安装目录拷贝 `conf` 目录到新安装目录，或者使用 `--path.conf` 选项到外部的config目录
*   从旧的安装目录拷贝 `data` 目录到新的安装目录，或修改 `config/elasticsearch.yml` 中的 `path.data` 设置 data 目录为原来的目录。

#### 第四步：启动升级过的节点

启动升级后的节点并确认加入到集群中，可以通过日志或下面的命令来确认：

```
GET _cat/nodes
```

#### 第五步：重新打开分片再平衡

一旦节点重新加入集群，解禁分片分配进程再平衡：

```
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "all"
  }
}
```

#### 第六步：等待节点恢复正常

等待集群分片平衡结束后，再升级下一个节点。这一过程可以使用[`_cat/health`](cat-health.html "cat health")命令检查：

```
GET _cat/health
```

等到 `status` 这一列由 `yellow` 变成 `green`，Green 表示主分片和副本都分配完了。


> 重点：滚动升级过程中，高版本上的主分片不会把副本分配到低版本的节点，因为高版本的数据格式老版本不认。
>  如果高版本的主分片没法分配副本，换句话说如果集群中只剩下了一个高版本节点，那么节点就保持未分配的状态，集群健康会保持 `yellow`。
> 这种情况下，检查下有没有初始化或分片分配在执行。
> 一旦另一个节点升级结束后，分片将会被分配，然后集群状态会恢复到 `green` 。

没有使用[同步刷新](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/indices-synced-flush.html "Synced Flush")的分片恢复时间会慢一点。分片的状态可以通过[`_cat/recovery`](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/cat-recovery.html "cat recovery")请求监控：


```
GET _cat/recovery
```

如果你在这之前停止索引操作，那么在节点恢复完成之后重启也是安全的。

#### 第七步：重复上述步骤

当集群稳定并且节点恢复后，对剩下的节点重复上述过程。

***

# 停集群升级
当升级跨越大版本的 Elasticsearch 时，需要停集群重启：从 0.x 到 1.x 或 1.x 到 2.x 等。滚动升级不支持跨大版本的集群。
按照以下步骤进行停机群升级：

#### 第一步：禁用分片分配

当你关闭一个节点时，分片分配进程将分片从当前节点复制到集群另一个节点上，这将会浪费大量的 I/O 操作。可以在关闭节点前禁用这个特性：

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "none"
  }
}
```

如果从 0.90.x 升级到 1.x ，使用以下配置：

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disable_allocation": true,
    "cluster.routing.allocation.enable": "none"
  }
}
```
#### 第二步：执行同步刷新
停止不必要的索引，并执行同步刷新，分片恢复速度会更快：
>POST /_flush/synced

同步刷新是锦上添花的操作。如果出现索引挂起的现象操作就会失败，为了安全起见有必要多试几次。

#### 第三步：关闭并升级全部的节点：
关闭集群全部节点的 ES 服务，每个节点使用和 滚动重启 一样的方法升级每个节点。

#### 第四步：启动集群
如果你有专门的 master 节点群（ master 节点就是 node.master 为 true 并且 node.data 为 false 的节点），最好先启动它们。等待它们形成集群并选举出主节点。这个过程可以通过日志检查。

一旦达到 [最小可选举主节点](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/modules-discovery-zen.html#master-election "Master Electionedit")个数时，将会形成一个集群并选举出主节点。然后就可以通过[`_cat/health`](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/cat-health.html "cat health") 和 [`_cat/nodes`](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/cat-nodes.html "cat nodes") API 监控节点加入鸡群的情况：
>GET _cat/health
GET _cat/nodes

使用上述 API 检查节点是否成功加入集群。

#### 第五步：等待集群恢复至 `yellow`
只要节点加入集群，节点上的主分片就开始在本地恢复。最初 `_cat/health` 将会返回 `red` 状态，表示有主分片还没分配。
一旦每个节点的本地分片都恢复了，状态将会变成 `yellow` ，表示所有的主分片都分配了，但并不是所有副本都有分配。这就是期待的效果，毕竟分片平衡还被禁用中。

#### 第六步：重启平衡
延后副本分配，直到所有节点都加入集群，master 就可以给本地已有分片拷贝的节点分配副本了。这时所有节点都在集群中，可以安全得重启分片平衡了。

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
```

如果从 0.90.x 升级到 1.x，使用以下配置：

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disable_allocation": false,
    "cluster.routing.allocation.enable": "all"
  }
}
```

集群此时开始分配副本到所有节点，这时索引和搜索操作都是安全的了。但是如果你等恢复结束后再操作索引分片会恢复得快一点。

你可以通过 [`_cat/health`](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/cat-health.html "cat health") 和 [`_cat/recovery`](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/cat-recovery.html "cat recovery")监控这个过程

一旦 `_cat/health` 返回结果的 `status`  s列返回 `green`，说明所有的主分片和副本都分配完毕。









