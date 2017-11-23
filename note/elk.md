# [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

# 零散点

- 一个`Elasticsearch`集群可以包含多个`索引`，相应的每个索引可以包含多个`类型`。 这些不同的类型存储着多个`文档` ，每个文档又有多个`属性`。
- 索引实际上是指向一个或者多个物理`分片`的`逻辑命名空间`。
- 分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。
- 在请求的查询串参数中加上`pretty`参数，会调用 Elasticsearch 的 pretty-print 功能，该功能 使得 JSON 响应体更加可读。
- `DSL`结构化查询语言.

## 检查文档是否存在

```
curl -i -XHEAD http://host:9200/{index}/{type}/{id}
```

## 创建文档

### 方法一

```
PUT /{index}/{type}/{id}?op_type=create
```

### 方法二

```
PUT /{index}/{type}/{id}?_create
```

成功返回`201 Created`的`HTTP`响应码。
文档已经存在，将会返回`409 Conflict`响应码。

## 删除文档

```
DELETE /{index}/{type}/{id}
```

## 处理冲突

### 乐观并发控制

- 指定`version`
- 外部版本号`version_type=external`
- 文档的部分更新`_update`
- 失败重试`_update?retry_on_conflict=5`

## 取回多个文档

- `GET /_mget`

  `mget` API 要求有一个`docs`数组作为参数，每个元素包含需要检索文档的元数据，包括`_index` 、`_type`和`_id` 。`_source`参数来指定想检索一个或者多个特定的字段

- `GET /{index}/{type}/_mget`

## 创建测试数据

curl https://gist.githubusercontent.com/clintongormley/8579281/raw/563aecba1910a492da1e273cce5400ce240021c1/load_test_data.sh | sh

## 搜索

### 搜索

`GET /_search`

设置超时 `GET /_search?timeout=10ms`

### 多索引，多类型

### 分页

`size` `from`

### `轻量`搜索

### 精确值 VS 全文

### 测试分析器

`GET /_analyze`

### 最重要的查询

- `match_all` 查询简单的 匹配所有文档。
- 无论你在任何字段上进行的是全文搜索还是精确查询，`match`查询是可用的标准查询。
- `multi_match`查询可以在多个字段上执行相同的`match`查询
- `range`查询找出那些落在指定区间内的数字或者时间

  - `gt`大于
  - `gte`大于等于
  - `lt`小于
  - `lte`小于等于

- `term`查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 `not_analyzed`的字符串
- `terms`查询和`term`查询一样，但它允许你指定多值进行匹配。
- `exists`查询和`missing`查询被用于查找那些指定字段中有值`(exists)`或无值`(missing)`的文档。
- 组合多查询

  - `must` 文档*必须*匹配这些条件才能被包含进来
  - `must_not` 文档*必须不*匹配这些条件才能被包含进来
  - `should` 如果满足这些语句中的任意语句，将增加`_score`，否则，无任何影响。
  - `filter` *必须*匹配，但它以不评分、过滤模式来进行

- `constant_score`查询

### 验证查询

`POST /{index}/{type}/_validate/query`

#### 查询不合法的原因

`PSOT /{index}/{type}/_validate/query?explain`

#### 理解查询语句

`POST /_validate/query?explain`

## 排序与相关性

#  执行分布式检索

## 查询阶段

## 取回阶段

## 搜索选项

- 偏好
- 超时问题
-  路由
-  搜索类型

  缺省的搜索类型是`query_then_fetch`。 在某些情况下，可能想明确设置`search_type`为`dfs_query_then_fetch`来改善相关性精确度

## 游标查询*Scroll*

`scroll`查询可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。

# 索引管理

## 创建一个索引

## 删除一个索引

`action.destructive_requires_name: true`

## 索引设置

- `number_of_shards` 每个索引的主分片数，默认值是 5 。这个配置在索引创建后不能修改。
- `number_of_replicas` 每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。

## 配置分析器

- `standard`分词器，通过单词边界分割输入的文本。
- `standard`语汇单元过滤器，目的是整理分词器触发的语汇单元（但是目前什么都没做）。
- `lowercase`语汇单元过滤器，转换所有的语汇单元为小写。
- `stop`语汇单元过滤器，删除停用词--对搜索相关性影响不大的常用词。

## 自定义分析器

### 字符过滤器

### 分词器

### 词单元过滤器

## 类型和映射

类型 | 表示的数据类型
---- | -------------
String | string(6.0 改为 text)
Whole number | byte, short, integer, long
Floating point | float, double
Boolean | boolean
Date | date

### 查看映射

```
GET /{index}/_mapping/{type}
```

## 根对象

##  动态映射

## 自定义动态映射

- 区分全文(full text)字符串字段和准确字符字段(就是分词与不分词,全文的一般要分词,准确的就不需要分词)
- 使用特定语言的分析器
- 优化部分匹配字段
- 指定自定义日期格式

映射中最重要的字段参数是`type`

## 缺省映射

`string`类型的字段默认的考虑到包含全文本,它们的值在索引前要经过分析器分析,并且在全文搜索此字段前要把查询语句做分析处理.

对于`string`字段,两个最重要的映射参数是`index`和`analyer`

### `index`

值  | 解释
--- | ---
analyzed | 首先分析这个字符串,然后索引.
`not_analyzed` | 索引这个字段,使之可以被搜索,但是索引内容和指定值一样.不分析此字段
no | 不索引这个字段,这个字段不能为搜索到

`string`类型字段默认值是`analyzed`

### 分析

对于`analyer`类型的字符串字段,使用`analyer`参数来指定哪一种分析器将在搜索和索引的时候使用,默认的`Elasticsearch`使用`standard`分析器,但可以通过指定一个内建的分析器来更改它,例如`whitespace`,`simple`或`english`.

## 重新索引数据

1. 删除旧索引
2. PUT mappings

## 索引别名和零停机

## 分片内部原理

# 深入搜索

## 结构化搜索

### 精确值查找

`term`查询可以用来处理数字（numbers）、布尔值（Booleans）、日期（dates）以及文本（text）。

通常当查找一个精确值的时候，不希望对查询进行评分计算。只希望对文档进行包括或排除的计算，所以会使用`constant_score`查询以非评分模式来执行`term`查询并以一作为统一评分。

```
GET /{index}/{type}/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "term_key" : "term_value"
                }
            }
        }
    }
}
```

### 组合过滤器

`bool`（布尔）过滤器: 是个*复合过滤器（compound filter）*，它可以接受多个其他过滤器作为参数，并将这些过滤器结合成各式各样的布尔（逻辑）组合。

- `must` 所有的语句都*必须（must）*匹配，与`AND`等价。
- `must_not` 所有的语句都*不能（must not）*匹配，与`NOT`等价。
- `should` 至少有一个语句要匹配，与`OR`等价。

#### 嵌套布尔过滤器

### 查找多个精确值

`terms`查询

#### 包含，而不是相等

`term`和`terms`是*包含（contains）*操作，而非*等值（equals）*（判断）。

### 范围

- `gt: >` 大于（greater than）
- `lt: <` 小于（less than）
- `gte: >=` 大于或等于（greater than or equal to）
- `lte: <=` 小于或等于（less than or equal to）

`range`查询支持对`日期计算（date math）`进行操作。

`range`查询同样可以处理字符串字段， 字符串范围可采用`字典顺序（lexicographically）`或字母顺序（alphabetically）。

### 处理`Null`值

`exists`存在查询。

`missing`查询

### 关于缓存

#### 自动缓存行为

### 全文搜索

- 基于词项的查询
- 基于全文的查询

### 匹配查询

词频（term frequency，即词 quick 在相关文档的 title 字段中出现的频率）和反向文档频率（inverse document frequency，即词 quick 在所有文档的 title 字段中出现的频率），以及字段的长度（即字段越短相关度越高）相结合的计算方式。

### 多词查询

`match`查询支持`minimum_should_match`最小匹配参数

### 组合查询

与过滤器一样，`bool`查询也可以接受`must`、`must_not`和`should`参数下的多个查询语句。

#### 评分计算

`bool`查询会为每个文档计算相关度评分*_score*， 再将所有匹配的`must`和`should `语句的分数` _score`求和，最后除以`must`和`should`语句的总数。

`must_not`语句不会影响评分； 它的作用只是将不相关的文档排除。

### 如何使用布尔匹配

### 查询语句提升权重

通过指定`boost`来控制任何查询语句的相对的权重，`boost`的默认值为`1`，大于`1`会提升一个语句的相对权重。

### 控制分析

### 被破坏的相关度！

## 多字段搜索

### 多字符串查询

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }}
      ]
    }
  }
}
'
```

可以用`bool`查询来包裹组合任意其他类型的查询，甚至包括其他的`bool`查询。

### 单字符串查询

### 最佳字段

可以使用`dis_max`即分离*最大化查询（Disjunction Max Query）*。分离（Disjunction）的意思是*或（or）*，这与可以把结合（conjunction）理解成*与（and）*相对应。分离最大化查询（Disjunction Max Query）指的是：*将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回*

### multi_match 查询

```
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields",
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%"
    }
}
```

### 多数字段

全文搜索被称作是*召回率（Recall）*与*精确率（Precision）*的战场：*召回率*——返回所有的相关文档；*精确率*——不返回无关文档。

### 跨字段实体搜索

### 字段中心式查询

### 自定义`_all`字段

### `cross_fields`跨字段查询

## 近似匹配

### 短语匹配

`match_phrase`查询首先将查询字符串解析成一个词项列表，然后对这些词项进行搜索，但只保留那些包含*全部*搜索词项，且*位置*与搜索词项相同的文档。

### 多值字段

### 越近越好

### 使用邻近度提高相关度

用`minimum_should_match`参数去除长尾。

### 性能优化

### 寻找相关词

### 部分匹配

### 邮编与结构化数据

### prefix 前缀查询

### 通配符与正则表达式查询

`wildcard`通配符查询也是一种底层基于词的查询，与前缀查询不同的是它允许指定匹配的正则式。

`regexp`正则式查询允许写出这样更复杂的模式

###  查询时输入即搜索

通过设置`max_expansions`参数来限制前缀扩展的影响， 一个合理的值是可能是`50`

### Ngrams 在部分匹配的应用

### 索引时输入即搜索

### Ngrams 在复合词的应用

## 控制相关度

### 相关度评分背后的理论

Lucene（或 Elasticsearch）使用`布尔模型（Boolean model）`查找匹配文档， 并用一个名为`实用评分函数（practical scoring function）`的公式来计算相关度。这个公式借鉴了`词频/逆向文档频率（term frequency/inverse document frequency）`和 向量空间模型（vector space model），同时也加入了一些现代的新特性，如协调因子（coordination factor），字段长度归一化（field length normalization），以及词或查询语句权重提升。

### Lucene 的实用评分函数

### 查询时权重提升

### 使用查询结构修改相关度

### Not Quite Not

### 忽略 TF/IDF

### `function_score`查询

### 按受欢迎度提升权重

### 过滤集提升权重

### 随机评分

### 越近越好

### 理解 price 价格语句

### 脚本评分

### 可插拔的相似度算法

### 更改相似度

### 调试相关度是最后 10% 要做的事情

