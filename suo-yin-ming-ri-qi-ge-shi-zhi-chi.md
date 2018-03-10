>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 索引名日期格式支持
索引名日期格式允许你搜索一系列时间序列格式的索引，而不用搜索全部的索引然后过滤结果，或者使用别名。限制搜索的索引个数可以降低集群负载提高搜索性能。例如如果你要从你每天的日志中搜索错误信息，你可以使用日期格式索引名模板限制搜索过去两天的索引。

日期计算索引名格式如下：

> \<static_name{date\_math\_expr{date\_format \| time\_zone}}>

说明：
- `static_name`：索引名静态部分
- `date_math_expr`：动态日期表达式，用来动态计算日期的。
- `date_format`：供时间计算的日期格式，默认格式 `YYYY.MM.dd`
- `time_zone`：时区，默认是 `utc`

你需要用尖括号把日期格式索引名包括起来，例如：

>curl -XGET 'localhost:9200/<logstash-{now%2Fd-2d}>/_search' {
  "query" : {
    ...
  }
}

>注意：括号中的 `/` 符号必须用 Url 编码 %2F 代替。

下面的例子展示了不同格式的日期格式索引，当前的日期为2024年3月22日晚上，时区 utc。

表达式  | 解析结果
--------|------
<logstash-{now/d}>|`logstash-2024.03.22`
<logstash-{now/M}>|`logstash-2024.03.01`
<logstash-{now/M{YYYY.MM}}>|`logstash-2024.03`
<logstash-{now/M-1M{YYYY.MM}}>|`logstash-2024.02`

在索引名静态部分使用`{`和`}`作为模板，使用反斜杠`/`转义它们。例如：
- <elastic\\{ON\\}-{now/M}> 转换成 elastic{ON}-2024.03.01

下面的例子，搜索三天内Logstash的索引，假设我们的索引名格式为：`logstash-YYYY.MM.dd`。

>curl -XGET 'localhost:9200/<logstash-{now%2Fd-2d}>,<logstash-{now%2Fd-1d}>,<logstash-{now%2Fd}>/_search' {
  "query" : {
    ...
  }
}

