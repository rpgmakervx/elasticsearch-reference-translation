>* 原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/2.2/index.html
>* 原文作者：[https://www.elastic.com](https://www.gitbook.com/book/rpgmakervx/elasticsearch-reference-translation/edit#)
>* 译者：[code4j](https://www.gitbook.com/book/rpgmakervx/elasticsearch-reference-translation/edit#)


# 突出的版本变化
（**译者批注：这一章译者选择翻译 5.5 版本的变化，因为全文主要讲解 2.x 版本，所以有必要了解高版本的新特性**）
这一章主要讨论版本之间的不同，特别是当你要从一个版本迁移数据到另一个版本的时候，你需要注意。

通常的规律：
- 跨越大版本迁移 —— 例如 1.x 到2.x —— 需要停集群升级。
- 小版本迁移 —— 例如 1.x 到1.y —— 需要滚动升级。
- 跨越不连续版本迁移 —— 例如 1.x到5.x —— 不支持 
（**译者批注：建议日志类集群可以进行跨版本升级，业务类集群如果已经投入大量使用，就先缓缓吧，数据同步又是一个很麻烦的过程，特别是不同业务要求又不同，牵扯到的人力成本比较高，升级的性价比不高。可以选择搭建新集群接入新业务，老业务再有重构需求时一并升级**）

详细请看[集群升级]()。

***

# 5.5版本变动

### 忽略隐藏文件夹
之前的版本会跳过plugin目录下面的隐藏文件夹，这样会导致损坏的插件安装后不会被ES探查到，所以这个宽泛的特性被移除了。

### 跳过内核参数设置
Elasticsearch在安装的时候尝试设置 `vm.max_map_count` 参数（**译者批注：译者看了下2.2版本的启动脚本确实有执行 sysctl -q -w vm.max_map_count=$MAX_MAP_COUNT**），在某些系统下可能失败。在早先的版本，ES 接受一个 ES_SKIP_SET_KERNEL_PARAMTERS 参数来跳过内核参数设置。从 Elasticsearch5.5.0 开始，取消这个参数以及相应的功能，可以通过 `systemd-sysctl` 操作来实现相同的效果。

### RPM/DEB安装包在 `/etc/elasticsearch` 下配置了 `setgid`
现在RPM/DEB安装包中在 `/etc/elasticsearch` 里设置了 `setgid`，这样任何在 `/etc/elasticsearch`目录下的文件或子目录都属于同一个组（`root:elasticsearch`）。主要是保证这个目录下创建的文件都对 `elasticsearch` 用户可读。

### 获取别名和获取映射 API 返回 404
之前的版本，在请求获取别名 和 get mapping 两个api的时候，无论什么情况都不会返回404。在Elasticsearch5.5版本后，当获取别名或获取映射不存在的时候，返回404.

### Java API改变
`has_child`，`has_parent`，`paren_id` 等查询都放到了 parent-join模块。`children` 聚合也放到了这个模块下。parent-join模块需要在 `TransportClient` 使用的时候加载。或者当要动态加载模块时，使用 `PreBuiltTransportClient` 替代

***

#5.4版本变动

###Delete\_By\_Query API改变
Delete_By_Query 的请求体必须有查询条件，不包含请求体的 Delete_By_Query 在6.0.0是不合法的。

***

#5.3 版本变动

###日志配置
此前的版本 Elasticsearch 暴露了一个系统参数 `es.logs`，它包含了系统配置的日志绝对路径，并且它的前缀用来表示各种日志文件名（主日志文件，过期日志和慢日志）。这个参数已经被下面三个取代了：
- `es.logs.base_path`：日志目录绝对路径
- `es.logs.cluster_name`：默认的各种日志文件名前缀。
- `es.logs.node_name`：如果文件名中需要包含节点名的话，在这里用。
`es.logs`参数已经废弃并将在 6.0.0 版本移除。

### Netty 3 被废弃
Netty 3中的transport（`transport.type=netty3`）和Http（http.type=netty3）在6.0.0被废弃

### 弱检查的布尔值被废弃
布尔值 除了false , “false” 和 true “true” 其他都被废弃（**译者批注：2.2中有些配置支持使用 yes no**）。REST API和映射配置中也废弃了这种使用方式

***

#5.2版本变动

##
###启动检查系统调用
Elasticsearch 自 2.1.0 版本开始试图加入系统调用过滤器。在某些系统中，系统调用过滤器可能导致失败。在早先的版本会加入 waring 日志记录，但是会继续执行，可能用户并不知道这个情况。从 5.2.0 版本开始加入启动检查系统调用，看是否成功。如果你启动的时候因为这个检查而出问题了，你需要检查是需要修改配置中的系统调用过滤器，还是**自担后果**去掉这个系统调用过滤器。

###系统参数过滤器配置
简单说就是`bootstrap.seccomp`这个参数被更名为：`bootstrap.system_call_filter`。前者仍旧支持但是6.0版本会被移除。

###Java API 删除了 source 方法的一些重写
删除了以下方法：
- PutRepositoryRequest#source(XContentBuilder)
- PutRepositoryRequest#source(String)
- PutRepositoryRequest#source(byte[])
- PutRepositoryRequest#source(byte[], int, int)
- PutRepositoryRequest#source(BytesReference)
- CreateSnapshotRequest#source(XContentBuilder)
- CreateSnapshotRequest#source(String)
- CreateSnapshotRequest#source(byte[])
- CreateSnapshotRequest#source(byte[], int, int)
- CreateSnapshotRequest#source(BytesReference)
- RestoreSnapshotRequest#source(XContentBuilder)
- RestoreSnapshotRequest#source(String)
- RestoreSnapshotRequest#source(byte[])
- RestoreSnapshotRequest#source(byte[], int, int)
- RestoreSnapshotRequest#source(BytesReference)
- RolloverRequest#source(BytesReference)
- ShrinkRequest#source(BytesReference)
- UpdateRequest#fromXContent(BytesReference)

### _timestamp 元数据字段改变
`timestamp` 元字段的类型由 `java.lang.String` 改成了 `java.util.Date`。

#5.1版本变动

### 别名添加了和索引名几乎相似的校验规则
现在别名添加了和索引名几乎相似的校验规则。唯一不同的是别名允许使用大写字母。意味着别名不允许使用如下规则：
- 以 `_`, `-`, 或 `+`开头
- 包含这些字符：#, \, /, *, ?, ", <, >, |, ` , `
- 超过100个UTF编码字节
- 使用 `.` 或 `..`

5.1.0 版本以前上述规则的别名还支持，但是现在不支持了。由于使用`_aliases` API 修改别名实际上是删除别名再自动重新创建一个，修改别名如果非法也不再支持。

### Log4j的依赖版本升级
Log4j 的版本从 2.6.2 升级到 2.7。如果你使用 transport client，需要更新相应的Log4j版本

### 节点本地发现已经移除
节点本地发现已经移除。这种发现机制的实现，用于内部的tribe服务以及在同一个JVM环境下运行多个节点的测试。这意味着设置 `discovery.type` 为 `local` 启动 elasticsearch 将会失败。

### UnicastHostsProvider 现在基于拉的模式
插件基于zen发现的UnicastHostsProvider现在采用拉的模式。实现 `DiscoveryPlugin` 类并覆盖 `getZenHostsProviders` 方法。现在也可以通过 `discovery.zen.hosts_provider` 配置 hosts_provider 类型。

### ZenPing 和主节点选举服务的可插拔特性取消
ZenPing 还有选举服务类都不支持可插拔了。有必要的话，要么自己实现发现，要么扩展 ZenDiscovery 

### onModule 支持移除
之前通过 Guice 注入模块的插件可以实现一个 `onModule` 方法。插件中所有自定义 `onModule` 方法的实现都被转换成基于拉的插件。onModule 这个 hook 被移除。
（**译者批注：源码大概看了下，2.2 版本中，节点启动时先加载插件，然后读取插件中的 `onModule` 方法并把 plugin 对应的 module 作为参数传递进去。这个方法并不是 plugin 类要求继承的，而是一个约定**）

### 索引的 stats 和节点的 stats API 中无法识别的参数
索引和节点的 stats API 允许查询 Elasticsearch 中无法识别的一些指标。在之前的版本会默认接受这些不识别的参数（例如 `transport`）。在 5.1.0 中这个情况不再发生；不能识别的参数将导致请求失败。5.0.0 移除的这个特性在 5.1.0 以及之后的5.x系列会给出警告，但是 6.0 之后会和其他不识别的参数一样，导致请求失败。

***

# 5.0 版本变动

这一张讲解 5.0 的变动，当你迁移集群到这个版本时你要注意。

## 迁移插件
当你升级到Elasticsearch 5.0后，[`elasticsearch-migration`](https://github.com/elastic/elasticsearch-migration/blob/2.x/README.asciidoc)插件（ 2.3 以上的版本）会帮你发现需要解决的问题

## 5.0 之前版本创建的索引
Elasticsearch 5.0 可以解析 2.0 之后创建的索引，但是2.0之前的索引无法识别，节点启动会失败。

初次启动 Elasticsearch 5.0，他将会自动重命名索引目录，目录命名由原来的索引名改成索引的 UUID，