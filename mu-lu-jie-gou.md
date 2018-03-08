>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-dir-layout.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/setup-dir-layout.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 目录结构

安装目录如下：

目录  | 说明 |默认路径|设置
--------|-------|-------|------
**home**|ES 安装主目录|    |**path.home**
**bin**|节点启动用的二进制启动脚本，包括 elasticsearch|{path.home}/bin ||
**conf**|配置文件，包括 elasticsearch.yml |{path.home}/config|**path.config**|
**data**|节点上存放索引/分片数据的目录。支持多个目录|{path.home}/data|**path.data**|
**plugins**|插件目录，每个插件都在它的子目录中|{path.home}/plugins|**path.plugins**|
**repo**|共享文件系统路径，支持多个目录。文件系统仓库可以这个目录下的任意子目录中|默认没有配置|**path.repo**|
**script**|脚本目录|{path.conf}/scripts|**path.script**|

指定多个 data 目录，目的是将数据分散到多个磁盘或目录中，但是来自同一个分片中文件都写到一个目录中。可以按如下配置：
>path.data: /mnt/first,/mnt/second

或者使用数组风格：
>path.data: ["/mnt/first", "/mnt/second"]

>提示：如果想将分片数据也分散到不同磁盘中，请使用 RAID 驱动代替。


