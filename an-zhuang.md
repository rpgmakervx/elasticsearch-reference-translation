# 安装

Elasticsearch 需要JDK版本最低1.7，特别是撰写本文时，建议你使用Oracle JDK版本1.8.0\_72，这里我们不关注不同Java版本的细节。Oracle推荐安装文档在此： [Oracle’s website](http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)。总之在你安装ES之前，使用以下命令检查你的Java环境：

> java -version

echo $JAVA\_HOME

一旦我们Java环境配好了，我们就可以下载运行ES了。二进制安装包以及所有版本都可以从这里获得[`www.elastic.co/downloads`](http://www.elastic.co/downloads)，每个版本都有`zip`，`tar`压缩包，或者\`deb\`和\`rpm\`安装包，简单来说我们用\`tar\`包安装。

下面我们使用如下命令下载ES2.2.1（Windows用户请自行下载zip包）：

> curl -L -O [https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.2.1/elasticsearch-2.2.1.tar.gz](https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.2.1/elasticsearch-2.2.1.tar.gz)

然后解压：

> tar -xvf elasticsearch-2.2.1.tar.gz

之后会释放一系列文件和文件夹，然后我们进入 \`bin\`目录：

> cd elasticsearch-2.2.1/bin

现在我们可以运行一个单节点的集群了（Windows用户双击elasticsearch.bat脚本）：

> ./elasticsearch

启动成功，你将看到如下启动日志：

> \[2014-03-13 13:42:17,218\]\[INFO \]\[node           \] \[New Goblin\] version\[2.2.1\], pid\[2085\], build\[5c03844/2014-02-25T15:52:53Z\]\[2014-03-13 13:42:17,219\]\[INFO \]\[node           \] \[New Goblin\] initializing ...\[2014-03-13 13:42:17,223\]\[INFO \]\[plugins        \] \[New Goblin\] loaded \[\], sites \[\]
>
> \[2014-03-13 13:42:19,831\]\[INFO \]\[node           \] \[New Goblin\] initialized
>
> \[2014-03-13 13:42:19,832\]\[INFO \]\[node           \] \[New Goblin\] starting ...
>
> \[2014-03-13 13:42:19,958\]\[INFO \]\[transport      \] \[New Goblin\] bound\_address {inet\[/0:0:0:0:0:0:0:0:9300\]}, publish\_address {inet\[/192.168.8.112:9300\]}
>
> \[2014-03-13 13:42:23,030\]\[INFO \]\[cluster.service\] \[New Goblin\] new\_master \[New Goblin\]\[rWMtGj3dQouz2r6ZFL9v4g\]\[mwubuntu1\]\[inet\[/192.168.8.112:9300\]\], reason: zen-disco-join \(elected\_as\_master\)
>
> \[2014-03-13 13:42:23,100\]\[INFO \]\[discovery      \] \[New Goblin\] elasticsearch/rWMtGj3dQouz2r6ZFL9v4g
>
> \[2014-03-13 13:42:23,125\]\[INFO \]\[http           \] \[New Goblin\] bound\_address {inet\[/0:0:0:0:0:0:0:0:9200\]}, publish\_address {inet\[/192.168.8.112:9200\]}
>
> \[2014-03-13 13:42:23,629\]\[INFO \]\[gateway        \] \[New Goblin\] recovered \[1\] indices into cluster\_state
>
> \[2014-03-13 13:42:23,630\]\[INFO \]\[node           \] \[New Goblin\] started

简单看下日志信息，我们能够知道节点已经启动，名字是`New Goblin`（漫威中的英雄名字），并且选举它自己作为集群的主节点。

不用担心不知道主节点的含义，现在只要关注我们我们在一个集群中启动了一个节点就行了。

之前我们提到过，我们可以自定义集群名和节点名，我们可以在ES启动的时候使用如下命令：

> ./elasticsearch --cluster.name my\_cluster\_name --node.name my\_node\_name

同时注意日志中标有Http的那行，表示HTTP协议地址（`192.168.8.112`）和端口（`9200`），通过这个地址我们可以访问集群。ES默认使用9200端口提供REST API访问，有必要的话这个端口可以配置

（**译者批注：安全起见端口还是自定义一下比较好，节点名最好有含义，比如带上服务器ip，这样使用一些运维工具或日志中查看可以比较友好迅速的得知节点信息**）

