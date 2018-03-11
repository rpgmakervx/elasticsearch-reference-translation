>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/common-options.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/common-options.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 通用选项

以下选项可以用于全部的 REST API。

### 格式化的结果
当任意请求加入参数 `?pretty=true` 参数，返回的 JSON 将会被格式化（推荐只在 debug 的时候使用！）。另一个选项设置参数为 `?format=yaml` ，可以设置返回结果是可读的YAML格式。

### 人可读的输出
统计数据可以返回合适的人可读的格式（例如 `"exists_time": "1h"` 或 `"size": "1kb"`）和计算机的格式（例如`"exists_time_in_millis": 3600000` 或 `"size_in_bytes": 1024`）。人可读的值可以通过在 querystring 后面添加 `?human=false` 来关闭。这个功能在 stats 结果提供给监控工具而不是给人直接看的时候比较有意义。`human` 的默认值是 false。

### 日期数学表达式
绝大多数接受格式化的日期值的查询，例如 [range](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-range-query.html "Range Query")查询中的 `gt` 和 `lt`，或者 [`daterange` 集合](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/search-aggregations-bucket-daterange-aggregation.html "Date Range Aggregation")中的`from`和`to`，都能够识别日期数学。
表达式以固定的日期开始，可以是`now`或者日期字符串，然后以`||`结尾。这个固定的日期后面可以选择加上一个或多个数学表达式。
- +1h - 加一小时
- -1d - 减一天
- /d - 四舍五入到最近的一天

支持的[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/common-options.html#time-units "Time unitsedit")有y (year), M (month), w (week), d (day), h (hour), m (minute), 和 s (second).

展示几个示例：
- `now+1h` - 当前时间加一小时，毫秒处理
- `now+1h+1m` - 当前时间加一小时和一分钟，毫秒处理
- `now+1h/d` - 当前时间加一小时，并四舍五入到最近的一天
- `2015-01-01||+1M/d` - `2015-01-01`加一个月，并四舍五入到最近的一天

### 响应过滤器
所有的响应都接收一个 `filter_path` 参数，可以减少elasticsearch的返回结果。这个参数后面跟一组表达式使用逗号进行分割。
> curl -XGET 'localhost:9200/_search?pretty&filter_path=took,hits.hits._id,hits.hits._score'

```json
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "3640",
        "_score" : 1.0
      },
      {
        "_id" : "3642",
        "_score" : 1.0
      }
    ]
  }
}
```

同样也接收通配符匹配字段或字段名的一部分。
>curl -XGET 'localhost:9200/_nodes/stats?filter_path=nodes.*.ho*'
{
  "nodes" : {
    "lvJHed8uQQu4brS-SXKsNA" : {
      "host" : "portable"
    }
  }
}

`**`通配符可用于匹配字段中不确定确切路径的字段。例如，返回Lucene每个分片的版本：

> curl 'localhost:9200/_segments?pretty&filter_path=indices.**.version'

```json
{
  "indices" : {
    "movies" : {
      "shards" : {
        "0" : [ {
          "segments" : {
            "_0" : {
              "version" : "5.2.0"
            }
          }
        } ],
        "2" : [ {
          "segments" : {
            "_0" : {
              "version" : "5.2.0"
            }
          }
        } ]
      }
    },
    "books" : {
      "shards" : {
        "0" : [ {
          "segments" : {
            "_0" : {
              "version" : "5.2.0"
            }
          }
        } ]
      }
    }
  }
}
```

注意，Elasticsearch 有时候直接返回一个字段的原始值，就像 `_source` ，如果你想过滤 `_source` 字段，你要把 `filter_path` 参数和 `_source` 参数一起用（详见 [Get API](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-get.html#get-source-filtering "Source filteringedit")）。如下：

```json
curl -XGET 'localhost:9200/_search?pretty&filter_path=hits.hits._source&_source=title'
{
  "hits" : {
    "hits" : [ {
      "_source":{"title":"Book #2"}
    }, {
      "_source":{"title":"Book #1"}
    }, {
      "_source":{"title":"Book #3"}
    } ]
  }
}
```

### 展开设置
`flat_settings` 参数可以影响 settings 的渲染效果，当设置`flat_settings` 为 true，settings 将会平铺渲染展示：

```json
{
  "persistent" : { },
  "transient" : {
    "discovery.zen.minimum_master_nodes" : "1"
  }
}
```

当他设置为fasle的时候，将会返回更人性化的结构：

```json
{
  "persistent" : { },
  "transient" : {
    "discovery" : {
      "zen" : {
        "minimum_master_nodes" : "1"
      }
    }
  }
}
```
默认`flat_settings`是false。

### 参数
其余参数（当使用HTTP，HTTP映射到URL的字符串）遵循下划线约定。

## 布尔值
所有的 REST API（请求体和响应 JSON）都支持布尔值 false 有以下几种表示方式：`false`，`0`，`no`，`off`。剩下其他值都是`true`，注意这个和入索引的文档中的字段无关。

## 数值
所有 REST API 除了原生的 JSON 数字类型还支持数字格式的字符串。

## 时间单位
无论什么时候要指定时间，例如 `timeout` 参数，时间必须指定单位，比如 `2d` 表示两天，支持的单位如下：
- `y`  - Year （年）
- `M` - Month（月）
- `w` - Week（周）
- `d` - Day（日）
- `h` - Hour（小时）
- `m` - Minute（分钟）
- `s` - Second（秒）
- `ms` - Milli-second（毫秒）

## 数据大小单位

无论什么时候需要指定数据大小，例如设置缓冲区大小，单位必须要指定，例如 10kb 就是 1000 字节。支持的单位如下：
- `b` - Byte
- `kb` - 千字节
- `mb` - 兆字节
- `gb` - GB
- `tb` - TB
- `pb` - PB

## 距离单位
无论什么时候需要指定距离，例如在[Geo Distance Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-geo-distance-query.html "Geo Distance Query")查询中使用 `distance` 字段，没指定的情况下默认是米，也可以指定其他单位，例如 `1km` 或 `2mi`
单位列表如下：
- Mile - `mi` or `miles`
- Yard - `yd` or `yards`
- Feet - `ft` or `feet`
- Inch - `in` or `inch`
- Kilometer - `km` or `kilometers`
- Meter - `m` or `meters`
- Centimeter - `cm` or `centimeters`
- Millimeter - `mm` or `millimeters`
- Nautical mile - `NM`, `nmi` or `nauticalmiles`

 [Geohash Cell Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-geohash-cell-query.html "Geohash Cell Query")中的 `precision` （精度）字段可以接受上述距离单位，如果没有指定，使用 geohash 的长度。

## 模糊行
有些查询或API支持不精确的模糊匹配，使用 `fuzziness` 参数。 `fuzziness` 是上下文敏感的，也就是说这取决于被查询字段的类型。

#### 数字，日期和IPV4字段
当查询数字，日期和IPV4字段时，`fuzziness` 被解析为 `+/-`，表现和[Range Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-range-query.html "Range Query")类似：
>-fuzziness <= field value <= +fuzziness

`fuzziness` 字段应该使用数字值，例如 2 或 2.0。`date` 类型应该被解析为 long 值，当然也可以用包含时间的字符串—— “1h” —— 使用[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/common-options.html#time-units "Time unitsedit")[edit](https://github.com/elastic/elasticsearch/edit/2.2/docs/reference/api-conventions.asciidoc "Edit this page on GitHub")解析。 IPV4 字段可以接受 long 值或者另一个 IPV4 值（也被解析成 long 值得 IPV4 ）

#### String字段
当查询 `string` 类型字段，`fuzziness` 使用[编辑距离算法](https://en.wikipedia.org/wiki/Levenshtein_distance)解析，一个字符串的改变可以使一个字符串等价于另一个字符串。
`fuzziness` 参数可以指定如下值：
- `0`, `1`, `2` - 最大的编辑距离。
- `AUTO` 基于term的长度生成编辑距离，例如长度在0-2的必须要完全匹配，3-5的可以有一个字符不同，大于5的可以有两个。

通常来说AUTO是 `fuzziness` 参数的首选。

## 返回结果风格
所有API都接受 `case` 参数。当设置了 `camelCase` ，所有的参数都会使用驼峰命名，否则是下划线命名。注意这个在文档的 source 字段中不起作用。

（**译者批注：大概是因为 source 是使用者自定义的，而API里面的名字是 ES 可控的**）

## query string 中的请求体
对于不接受非 POST 请求的请求体的库，可以将请求体放在 query string 的 `source` 参数中传递。

***

# 基于 URL 的访问控制

很多用户使用代理访和基于 URL 的访问控制来确保 elasticsearch 的索引安全，对于 `multi_search`，`multi_get` 以及 `bulk` 请求，用户可以在URL中指定索引名，并且还可以在请求体中指定索引名。这样就基于URL的访问控制就比较有挑战了。

为了防止用户请求体中的索引名覆盖了 URL 中的索引名，可以在配置文件中加入如下配置：

>rest.action.multi.allow_explicit_index: false

默认值是 true。但如果配置为 false，Elasticsearch 就会拒绝请求体重指明索引名的请求。









