> \* 原文地址：[Getting Start](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/index.html) 
>
> \* 原文作者：[https://www.elastic.com](https://www.elastic.com)
>
> \* 译者：[code4j](https://github.com/rpgmakervx)

# Elasticsearch简介

Elasticsearch 是一个高可扩展的开源全文搜索分析引擎，可以用它近实时的来存储、搜索和分析大量的数据。通常我们使用它作为底层引擎技术给拥有复杂搜索功能需求的应用赋能。

以下是 Elasticsearch 的几个适用场景:

* 你经营一家网店，用户可以搜索你出售的商品。此时，你可以用 Elasticsearch 存储全部商品的目录和存货，然后给用户提供搜索和自动提示功能.

* 你想要收集日志或交易数据用于分析趋势、统计数据、概要和异常。此时，你可以使用 Logstash\(Elasticsearch/Logstash/Kibana技术栈的一部分\)来收集，聚合，解析数据，然后将其存入ES。一旦数据在ES里了，你就可以用搜索和聚合挖掘任何你感兴趣的数据。

* 你有一个可以让懂行的顾客制定类似“我对这个东西挺感兴趣的，当这个东西的价格在下个月之前降到X块钱了通知我”规则的价格预警平台。此时，你可以抹去卖主的价格，存入ES中，使用逆向搜索能力(Percolator)，根据用户的查询来匹配价格的变动，一旦价格匹配，给用户推送提醒.

* 你有分析和商业策略的需求，想快速的在大数据（有上十亿的记录）里研究，分析，做可视化，特定的询问。此时，你可以用ES存储你的数据，然后用Kibana\(Elasticsearch/Logstash/Kibana技术栈的一部分\)来定制可以让你的重要数据可视化的仪表盘。不仅如此，你可以用ES的聚合功能，根据你的数据作复杂的商业策略查询.

接下来的教程中会指引你从启动 elasticsearch 到基本的操作，了解内部机制。最后你将知道它是什么以及它内部的原理，希望你启发您使用 elasticsearch 构建更复杂的搜索应用或数据挖掘应用.

