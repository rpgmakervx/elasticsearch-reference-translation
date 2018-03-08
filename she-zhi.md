>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 设置
这一部分主要讲解的内容是如何设置 Elasticsearch 并让他跑起来，如果你还没有准备好，那么先[下载](https://www.elastic.co/downloads)，然后查阅[安装](https://www.jianshu.com/p/988f5753d040)步骤。

>注意：Elasticsearch 可以使用 `apt` 或 `yum` 源安装，仓库详见 [*Repositories*](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-repositories.html "Repositories").

### 支持的平台
官方支持的操作系统和 JVM 版本在这里：[支持的平台](https://www.elastic.co/support/matrix)。ES 在这些平台上都进行过测试，当然在其他版本平台上也可能运行成功。

### 安装
下载最新版本并解压，使用下面的命令运行 ES：
>bin/elasticsearch
在类unix系统下，该命令将会在前台启动 ES 进程。

### PID
Elasticsearch 进程可以被写入指定文件，这样后面关闭进程会更方便：
>bin/elasticsearch -d -p pid
kill `cat pid`

- 进程号将会被写入 `pid` 这个文件。
- `kill` 命令将对 `pid` 文件中的进程号发起一个 `TERM` 信号量。

（**译者批注：非常建议大家做这个配置，运维起来相当方便**）

>注意：Linux 和 Windows 中都有启动脚本，来管理 ES 的启停。

#### 类Unix系统
使用 ES 脚本启动还可以添加其他参数。首先我们刚才讨论的，可以让 ES 在前台或后台执行。
其他参数可以直接向脚本传递 -D 或 getopt 长风格配置参数。这里配置的参数，会覆盖 JAVA_OPTS 或 ES_JAVA_OPTS 里配置的同名参数，例如：
>bin/elasticsearch -Des.index.refresh_interval=5s --node.name=my-node

### Java(JVM)版本
Elasticsearch 由 Java 编写，并且需要最低版本为1.7才能够运行。只支持 Oracle JDK 还有 OpenJDK 。所有的 Elasticsearch 节点和客户端需要使用相同版本的 JVM。
我们推荐安装 Java 820 或 Java 755 以上的版本。Java7 早期版本的 bug 可能会造成索引损坏和数据丢失。如果在错误的 JDK 版本上运行 Elasticsearch，将会拒绝运行。
可以通过 JAVA_HOME 环境变量配置使用的 Java 版本。

这些日志，特别是你打算升级你的 ES 版本。




