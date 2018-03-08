>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-configuration.html
>* 译者：[code4j](https://github.com/rpgmakervx)


# 配置

# 环境变量
Elasticsearch会使用脚本中内置的 `JAVA_OPTS` 变量值作为JVM启动参数，最重要参数 `-Xmx`，它控制进程最大的堆内存，`-Xms`，控制进程分配最小堆内存（通常来说内存分配越多越好）。
通常来说，推荐做法是不改变`JAVA_OPTS`变量，而使用`ES_JAVA_OPTS`来改变JVM参数配置。
`ES_HEAP_SIZE `参数用来配置Java为ES进程分配的堆内存，最大最小值是一样的，当然也可以通过指定的参数分别设置，最小最大分别是 `ES_MIN_MEM`（默认256M）和`ES_MAX_MEM `（默认是1G）。
（**译者批注：通常建议最大最小内存设置成一样的，因为通常 ES 集群所在服务器资源尽可能都是提供给集群使用的，所以资源管够，而设置不同的最大最小值，会产生内存扩容导致过多开销**）
建议将最大值最小值设置一样，并且打开（mlockall）
（**译者批注：`mlockall` 这个参数可以防止进程进行swap内存交换，内存交换效率很低，毕竟要读写磁盘。详细原因可以看这里：[把bootstrap.mlockall设为true](http://zhaoyanblog.com/archives/826.html)**）

### 系统配置
**文件描述符**
确保增大你机器（或者运行 ES 的用户）的最大文件打开数，推荐设置为32K或64K。
为了测试一个进程的最大文件打开数，启动时配置参数 `-Des.max-open-files` 为 `true`。这样系统启动时就会打印最大文件打开数。
另外，你也可以通过解析下面API的结果中的 `max_file_descriptors ` 参数获取每个节点的最大文件打开数：
> curl localhost:9200/_nodes/stats/process?pretty


**虚拟内存**
Elasticsearch默认使用 `mmaps / niofs` 混合的目录存储类型存储索引。默认的操作系统限制 `mmaps`（内存映射模式）中的虚拟地址空间限制太小了，可能会导致内存溢出，在Linux上，你可以使用 `root` 用户通过如下命令扩大限制：
>sysctl -w vm.max_map_count=262144

如果想要永久改变这个参数的花，修改配置文件 `/etc/sysctl.conf` 中的 `vm.max_map_count` 这项配置。
>注意：如果你是用安装包安装 ES（.deb或rpm），这个配置会自动改变，你可以使用命令 `sysctl vm.max_map_count` 来校验。

**内存配置**
大多数操作系统会尽可能多的使用内存给文件系统缓存并迫切的交换（后称 `swap` ）无用的应用内存到磁盘，这有可能导致 Elasticsearch 进程被 swap 到磁盘，swap 十分消耗性能并且影响节点的稳定性，因此要不惜一切代价避免它发生。
你有三种选择：
- **禁用swap**
简单粗暴的禁用 swap，通常 Elasticsearch 作为一个服务运行在沙箱环境中，其内存通过系统变量 `ES_HEAP_SIZE` 来控制，因此swap不应该启用。
Linux系统中，你可以通过命令 `sudo swapoff -a` 来关闭，如果想永久关闭，需要修改配置文件 `/etc/fstab` 并找到注释中包括 `swap` 的那行。
Windows 下，你可以完全禁用分页文件，设置方法：我的电脑右键属性->高级系统设置->高级/性能，点击设置->高级->虚拟内存，点击更改->选择无分页文件。
- **配置 `swappiness`**
第二个选择就是确保sysctl的配置 `vm.swappiness` 设置为0。这样能够降低内核的 swap 频率，正常情况下是不会产生 swap 的，当然系统在紧急情况下还是会进行swap。
>注意：在内核版本 3.5-rc1 及以上，`vm.swappiness` 设置为0的时候，触发 OOM 则会杀掉进程而不是进行 swap，紧急情况下你需要设置该配置为1来确保 swap 是能够执行的。

- **mlockall**
第三个选择是在 Linux/Unix 使用 `mlockall` ，Windows 使用 [VirtualLock](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366895%28v=vs.85%29.aspx)，能够将进程地址空间锁在内存中，防止 Elasticsearch 进程内存被 swap 出去，可以通过在配置文件 `elasticsearch.yml` 中加入下面的配置：
> bootstrap.mlockall: true

启动后，你可以通过检验下面这个命令的返回结果中的 `mlockall` 值来看配置是否成功：
>curl http://localhost:9200/_nodes/process?pretty

如果你发现 `mlockall` 的值是 false，说明配置失败了，最有可能的原因是在 Linux/Unix 系统中启动ES的用户没有权限锁住内存，你可以在启动前，通过 root 用户执行 `ulimit -l unlimited` 来授权。

另一个原因可能是临时文件目录（ Linux 下通常是 `/tmp` ）挂载时指定了 `noexec` 选项，你可以在启动 ES 的时候指定新的临时目录来解决：
>./bin/elasticsearch -Djna.tmpdir=/path/to/new/dir

>注意：启用 mlockall ，如果尝试申请超过可用内存大小的内存，可能会导致 JVM 或 shell 回话退出。

# Elasticsearch配置
elasticsearch 的配置文件在 `ES_HOME/config` 目录下，目录中有两个文件，`elasticsearch.yml` 用来配置ES的不同模块， `logging.yml` 用来配置ES日志相关设置。
配置风格是[YAML](http://www.yaml.org/)，下面我们来一个示例，修改所有网络模块绑定的地址信息，改为如下：
>network.host: 10.0.0.4

（**译者批注：格式按照默认配置文件来即可，官网的 yaml 配置风格和我是用的略有不同，这里我直接使用默认的配置文件中的注释 demo **）

#### 目录

生产环境使用，我们肯定需要修改 data 和 log 的存放目录：
> path.data: /var/data/elasticsearch
path.logs: /var/log/elasticsearch

（**译者批注：data 目录可以写多个，通过逗号分隔，实现磁盘阵列，把数据写到多个磁盘上，降低读写锁的开销。不过目前译者没有这么实践过。**）

#### 集群名

别忘了给你的集群命名，用来使节点发现并自动加入集群。
>cluster.name: <NAME OF YOUR CLUSTER>

确保你的集群名不会在不同环境复用，否则可能会导致节点加入错误的集群而出错。例如你可以使用  `logging-dev`, `logging-stage`，`logging-prod`表示开发集群，预发布(**译者公司称作沙箱**)集群和生产集群。

#### 节点名

也许你也需要修改节点名称，就像主机名一样。默认情况下节点启动时会从3000个漫威人物名字中随机选取一个。
>node.name: <NAME OF YOUR NODE>

（**译者批注：推荐命名用编号区分方便管理维护**）

机器的主机名可以通过系统变量 `HOSTNAME`获取，如果你的机器只运行集群中的一个节点，可以设置节点名为主机名，使用标签 `${...}`。
> node.name: ${HOSTNAME}

（**译者批注：这种配置方式极不推荐，不利于管理，风险高，把它当做一个 trick 测试玩好了。除非你做了一些安全或权限的插件，用到输入密码这种配置。**）

ES内部在处理这些配置的时候都会使用 “namespaced” 压缩处理（**译者批注：也就是译者现在为大家展示的这种风格**）。你也可以使用 JSON 风格的配置，配置文件命名为：`elasticsearch.json` 就行了：
**Code style**

```json
{
    "network" : {
        "host" : "10.0.0.4"
    }
}
```
也就是说你可以很轻松的通过外部配置，使用 `ES_JAVA_OPTS` 或者启动时参数进行配置，例如：
>./elasticsearch -Des.network.host=10.0.0.4

另外，如果你不希望存储你的配置，还可以使用通配符的方式在启动 ES 时从前台传值，使用 `${prompt.text}` 或 `${prompt.secret}` 配置，前者控制台输入显示明文，后者则不显示，示例如下：
>node.name: ${prompt.text}

启动ES后，会提示你输入参数值，如下：
>Enter value for [node.name]:

如果 `${prompt.text}` 或 `${prompt.secret}` 已经存在于配置中，或者你使用后台启动 ES，则这两个配置不会起作用。

# 索引配置
集群中创建的索引拥有它自己的配置，例如，以下创建索引的配置设置了刷新间隔为5s，而没有使用默认的刷新间隔（可以使用 YAML 或 JSON 格式）:
>$ curl -XPUT http://localhost:9200/kimchy/ -d \
'
index:
    refresh_interval: 5s
'

索引级别的配置也可以用于节点级别的配置，比如刚才的配置我们也可以在 `elasticsearch.yml` 中设置：
>index.refresh_interval: 5s

就是说在特定节点上创建的每个索引都将使用5s的刷新间隔，除非创建索引时设置它。也可以说，索引级别的配置可以覆盖节点级别的配置。

所有的配置信息都可以在[索引模块](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/index-modules.html "Index Modules")找到。

# 日志配置
elasticsearch 内部使用 log4j 日志系统，开箱即用，并使用 YAML 配置风格简化 log4j 的配置，配置文件是 `conf/logging.yml` ，同样 JSON 风格的配置文件也支持。多个配置文件也是支持的，它们会被整合到一起，前提是文件名要以 `logging.` 开头，并以支持的后缀结尾（目前可以是 `.yml`, `.yaml`, `.json` 或 `.properties`）。日志器包括 Java 包名和相应的日志级别，你可以省略 `org.elasticsearch` 前缀。 Appender 部分包括日志目的地。更多地自定义日志配置和 appender 类型请看 [log4j documentation](http://logging.apache.org/log4j/1.2/manual.html)。
额外的 Appender 还有 [log4j-extras](http://logging.apache.org/log4j/extras/)
提供的其他日志类也是支持的，开箱即用。

### 过期日志
除了常规日志，ES 允许你启动过期日志记录。例如如果你要迁移某些功能，需要你提前确定。默认的过期日志是禁用的，你可以通过如下配置启用：
>deprecation: DEBUG, deprecation_log_file

这将会每天在你的日志目录创建滚动日志，经常检查


