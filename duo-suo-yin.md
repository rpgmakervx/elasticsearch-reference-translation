>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/multi-index.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/multi-index.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# 多重索引

大多数用到索引名的 API 支持使用多个索引执行，仅仅使用 `test1`,`test2`,`test3` 这样的分隔符（或者 `_all` api，获取全部索引）即可。还支持通配符，例如：`test*`，还有"包含"和(`+`)"排除"(`-`)操作，例如：`+test*,-test3`。

（**译者批注：包含排除的例子，表示所有 test 开头但不包含 test3 的索引**）

所有多索引 API 支持在 querystring 下加入以下参数：

- `ignore_unavailable`：控制当索引不可用的时候是否忽略掉，不可用的索引包括索引不存在或索引被关闭了。可以设置 `true` 或 `false` 。
- `allow_no_indices`：控制当使用通配符索引名时，索引不存在会不会失败。可以设置 `true` 或 `false`。例如指定了通配符 `foo*` 但是没有 foo 开头的索引，请求成功与否就取决于这个参数
- `expand_wildcards`：控制通配符匹配到哪些具体的索引。如果指定为 `open` 则表示通配符副只匹配打开的索引，如果指定为 `closed` 则通配符只匹配关闭的索引。两个值都指定表示匹配全部索引。如果指定 `none` 则表示禁用通配符匹配，如果指定 `all` 则表示和同时指定 `open,closed` 一样的含义

（**译者批注：其实一般不指定就可以了，特殊需求要查询关闭的索引则指定 closed **）

>注意：单索引类的 API 例如[文档类](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/docs.html)以及[别名类](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/indices-aliases.html)操作不支持多索引。


