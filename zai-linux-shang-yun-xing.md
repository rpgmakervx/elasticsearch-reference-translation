>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-service.html
>* 译者：[code4j](https://github.com/rpgmakervx)

# 在Linux上启动服务
为了在 Linux 上启动ES服务，官方提供的包尽可能让你在升级或重启集群的时候轻松地启停集群。
目前我们有构建 debian 和 rpm 两种安装包，都可以在下载页获取。包本身没有依赖，但是你必须确认你安装了 JDK。
每个包都有个配置文件，它允许你设置如下配置参数：

参数  | 说明 
--------|------
`ES_USER `           | 启动ES使用的用户，默认是`elasticsearch`  
`ES_GROUP `        | 启动ES使用的用户的群组，默认是`elasticsearch`
`ES_HEAP_SIZE ` | 启动时堆内存大小
`ES_HEAP_NEWSIZE `  | 年轻带大小
`ES_DIRECT_SIZE`|直接内存大小
`MAX_OPEN_FILES`|最大文件打开数，默认是 65535
`MAX_LOCKED_MEMORY`|最大的锁内存大小。如果你配置了`bootstrap.mlockall`选项，把它设置为 `unlimited`。而且必须设置ES_HEAP_SIZE
`MAX_MAP_COUNT`|最大虚拟地址映射空间。如果你使用mmapfs作为索引存储类型，确保这个值设置的高一点。更多信息请看这里 [linux kernel documentation](https://github.com/torvalds/linux/blob/master/Documentation/sysctl/vm.txt)的`max_map_count`项，启动elasticsearch之前通过 sysctl  设置，默认是65535
`LOG_DIR`|日志目录，默认是 /var/log/elasticsearch
`DATA_DIR`|数据目录，默认是 /var/lib/elasticsearch
`CONF_DIR`|配置文件目录（目录中必须包含elasticsearch.yml 和logging.yml 文件），默认是 /etc/elasticsearch
`ES_JAVA_OPTS`|所有的JAVA启动参数都可以设置。当你需要修改`node.name`属性，但不想修改elasticsearch.yml配置文件的时候这一点十分有用，因为它是通过类似 puppet 或 chef 等配置管理工具分发到系统中的。例如：ES_JAVA_OPTS="-Des.node.name=search-01"
`RESTART_ON_UPGRADE`|配置 Elasticsearch 是否在包升级时重启。默认被配置为 false，这意味着你需要在安装新包或者升级包时手动重启。这样做的目的是确保集群升级的时候不会因为索引分片分配导致网络延迟，会降低集群响应时间
`ES_GC_LOG_FILE`|GC 日志的绝对路径，通过 JVM 配置，注意 GC 日志可能涨得很快随意默认是禁止的。

##Debian/Ubuntu
Debian软件包有你需要的一切，它使用标准的 debian 工具，类似改变`update-rc.d`来改变服务运行级别一样。初始化脚本如你预期，放在 `/etc/init.d/elasticsearch` 下，配置文件放在 `/etc/default/elasticsearch` 下。

debian安装包安装后默认不会启动服务，目的是防止实例在配置不当的情况下意外的加入集群。使用 `dpkg -i` 安装后确保Elasticsearch能开机自启，并启动 Elasticsearch。
>sudo update-rc.d elasticsearch defaults 95 10
sudo /etc/init.d/elasticsearch start

使用Debian 8或Ubuntu 14以上版本的用户需要使用`systemd`代替`update-rc.d`。这些情况下，请参考**使用Systemd**章节

##安装Oracle JDK
通常建议大家在使用 Oracle JDK 安装 Elasticsearch。然而Ubuntu和Debian因为许可证的问题只提供 Oracle JDK。你可以很轻松的安装oracle的安装包。以防你在 Debian 系统下漏加了 `add-apt-repository` 命令，确认至少要有 Debian 的 Jessie 源并且安装了`python-software-properties`包：
>sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
java -version

最后一条命令验证你的oracle JDK是否成功安装。

## 基于RPM包的安装

#### 使用chkconfig

有些基于RPM包安装的发行版需要 `chkconfig` 启用禁用服务。初始化脚本在 `/etc/init.d/elasticsearch`，配置文件放在 `/etc/sysconfig/elasticsearch`。类似debian的安装包，RPM包默认安装后是不启动的。你需要手动输入如下命令：
>sudo /sbin/chkconfig --add elasticsearch
sudo service elasticsearch start

#### 使用systemd

有些发行版类似Debian Jeessie（**译者批注：Jessie系列就是Debian 8**），Ubuntu 14还有一些SUSE的衍生版本不使用`chkconfig`注册服务，而是使用 `systemd`，命令是`/bin/systemctl`,，来启停服务（至少是新版本，否则使用`chkconfig`）。基于RPM安装的配置文件也是放在`/etc/sysconfig/elasticsearch`下，如果是deb包的话在`/etc/default/elasticsearch`中。安装好RPM包后，你需要修改 systemd 的配置并启动 elasticsearch。
>sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl start elasticsearch.service

注意在 `/etc/sysconfig/elasticsearch` 中修改MAX_MAP_COUNT并不起作用，你需要在`/usr/lib/sysctl.d/elasticsearch.conf`配置文件中修改，确保启动生效。







