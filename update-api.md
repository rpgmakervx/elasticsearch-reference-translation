>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-update.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs-update.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# Update API

更新操作允许提供脚本进行更新，更新操作从索引中获取文档（从分片中整理），执行脚本（使用可选的脚本语言和参数），然后索引返回结果（也可以删除，或忽视操作）。更新操作使用版本控制确保 get 或 reindex 过程中不会有更新发生。

注意，这个操作还意味着重建索引，这样能降低网络切换并降低了 get 和 index 操作发生版本冲突的可能。`_source` 字段必须配置是打开的。

例如，我们先索引一个文档：

```json
curl -XPUT localhost:9200/test/type1/1 -d '{
    "counter" : 1,
    "tags" : ["red"]
}'
```

## 脚本更新

现在我们来使用脚本给 counter 自增：

```json
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.counter += count",
        "params" : {
            "count" : 4
        }
    }
}'
```

我们可以给 tags 列表字段添加 tag（如果 tag 存在了，还是会向列表中添加）

```json
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.tags += tag",
        "params" : {
            "tag" : "blue"
        }
    }
}'
```
除了 `_source` 字段，以下的变量也是可以用的：`_index`，`_type`，`_id`，`_version`，`_routing`，`_parent`，和`_now`（当前的时间戳）。

我们也可以将新字段添加到文档：

```json
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : "ctx._source.name_of_new_field = \"value_of_new_field\""
}'
```

也可以从文档中删除字段：

```json
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : "ctx._source.remove(\"name_of_field\")"
}'
```

而且，我们甚至可以改变已执行的操作。下面的例子如果 tags 包含 blue则删除，否则就什么也不做（noop）：

```json
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.tags.contains(tag) ? ctx.op = \"delete\" : ctx.op = \"none\"",
        "params" : {
            "tag" : "blue"
        }
    }
}'
```

## 部分更新文档

更新 API 还支持部分文档的更新，将合并到现有的文档中（简单的递归合并，内合并，更换核心“键/值”对和数组），例如：

```json
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "doc" : {
        "name" : "new_name"
    }
}'
```
如果同时 doc 和 script 被指定，那么 doc 将被忽略。最好是在脚本中做部分更新。

## 检测空操作更新

如果指定了 `doc` ,值将被整合到现有的 `_source` 中。默认情况下，如果 `_source` 不同于老的文档则重建文档。如果配置了 `detect_noop` 为 true，即使文档不做修改，Elasticsearch也会修改文档。例如：

```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}'
```
即使 name 原来的值就是 `new_name`，也要重新索引。

## 更新插入(Upsert)

如果文档不存在，upsert 将作为一个新文档插入索引。如果文档存在，则执行 script 部分。

```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "script" : {
        "inline": "ctx._source.counter += count",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}'
```

### scripted_upsert

如果你想无论文档是否都存在，都执行脚本——即执行脚本而不是 upset 部分，则设置`scripted_upsert` 为 true：

```
curl -XPOST 'localhost:9200/sessions/session/dh3sgudg8gsrgl/_update' -d '{
    "scripted_upsert":true,
    "script" : {
        "id": "my_web_session_summariser",
        "params" : {
            "pageViewEvent" : {
                "url":"foo.com/bar",
                "response":404,
                "time":"2014-01-01 12:32"
            }
        }
    },
    "upsert" : {}
}'
```

### doc_as_upsert

设置 doc_as_upsert 为true，将使用 `doc` 代替 `upsert` 值，来替代 doc 加 upsert更新文档：

```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}'
```

## 参数

更新操作支持以下操作：

 - `retry_on_conflict` - 在更新中的 get 和 index 过程中，另一个进程可能也在修改相同的文档。默认情况下会抛出版本冲突异常，`retry_on_conflict` 参数用来控制抛出异常前的重试次数。
- `routing` - 用来将更新请求路由到正确的分片，如果文档更新不存在把其设置为更新插入请求。
- `parent` - 用来将父文档路由到正确的分片，文档不存在则把parent 设置为更新插入请求。
- `timeout` - 更新请求后等待分片可用的等待时间。
- `consistency` - 更新操作时所需要活跃分片个数。
- `refresh` - 更新操作后立马刷新相关的主副分片（不是整个索引），这样更新的内容就立马可见了。
- `version & version_type` - 更新API通过elasticsearch的版本控制机制，确保文档在更新的时候不会被修改。可以使用`version`参数指定更新版本，只能更新版本匹配的文档。使用 force 类型可以迫使修改文档使用新的版本。

### 更新API不支持外部版本控制
```
update API 不支持外部版本控制(类型为 `external & external_gte`)，这将导致elasticsearch版本控制与外部版本不一致。
```





