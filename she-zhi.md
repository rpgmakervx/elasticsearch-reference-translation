>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 设置
这一部分主要讲解的内容是如何设置Elasticsearch并让他跑起来，如果你还没有准备好，那么先[下载](https://www.elastic.co/downloads)，然后查阅[安装](https://www.jianshu.com/p/988f5753d040)步骤。

>注意：Elasticsearch可以使用`apt`或`yum`源安装，仓库详见 [*Repositories*](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-repositories.html "Repositories").

### 支持的平台
官方支持的操作系统和JVM版本在这里：[支持的平台](https://www.elastic.co/support/matrix)。ES在这些平台上都进行过测试，当然在其他版本平台上也可能运行成功。

### 安装
下载最新版本并解压，使用下面的命令运行ES：
>bin/elasticsearch
在类unix系统下，该命令将会在前台启动ES进程。

### PID
Elasticsearch进程可以被写入指定文件，这样后面关闭进程会更方便：
>bin/elasticsearch -d -p pid
kill `cat pid`

- 进程号将会被写入`pid`这个文件。
- `kill`命令将对`pid`文件中的进程号发起一个`TERM`信号量。

（**译者批注：非常建议大家做这个配置，运维起来相当方便**）

>注意：Linux和Windows中都有启动脚本，来管理ES的启停。

#### 类Unix系统
使用ES脚本启动还可以添加其他参数。首先我们刚才讨论的，可以让ES在前台或后台执行。
其他参数可以直接向脚本传递-D 或 getopt 长风格配置参数。这里配置的参数，会覆盖JAVA_OPTS 或 ES_JAVA_OPTS 里配置的同名参数，例如：
>bin/elasticsearch -Des.index.refresh_interval=5s --node.name=my-node

### Java(JVM)版本
Elasticsearch由Java编写，并且需要最低版本为1.7才能够运行。只支持Oracle JDK还有OpenJDK。所有的Elasticsearch节点和客户端需要使用相同版本的JVM。
我们推荐安装Java 820 或 Java 755 以上的版本。Java7早期版本的bug可能会造成索引损坏和数据丢失。如果在错误的JDK版本上运行Elasticsearch，将会拒绝运行。
可以通过JAVA_HOME环境变量配置使用的Java版本。

这些日志，特别是你打算升级你的ES版本。




