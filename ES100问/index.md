# Elasticsearch夺命连环100问——ES技术栈深度指南

> 本文档面向Elasticsearch学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」→「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是Elasticsearch？为什么要用ES？

#### 1.1 ES定义

> Elasticsearch（简称ES）是一个基于Lucene的分布式搜索和分析引擎，由ShayBanlon创建，现为Elastic公司产品。主要用于全文搜索、日志分析、指标分析、安全洞察等场景。

#### 1.2 为什么要用ES？

```
关系数据库MySQL痛点：
- 亿级数据查询慢（LIKE %% 全表扫描）
- 全文搜索支持差
- 聚合分析能力弱
- 实时性一般

ES优势：
- 倒排索引：毫秒级响应
- 分布式：PB级数据轻松处理
- 聚合分析强大
- 近实时（NRT：1秒）
```

#### 1.3 典型使用场景

```
1. 搜索引擎：电商搜索、内容搜索
2. 日志分析：ELK stack
3. 应用性能监控APM
4. 安全分析：SIEM
5. 业务分析：BI、数据看板
```

#### 1.4 回答模板

> Elasticsearch是基于Lucene的分布式搜索引擎，主要解决海量数据下的搜索和分析问题。它使用倒排索引实现毫秒级查询，支持PB级数据横向扩展。典型应用场景包括站内搜索、日志分析（ELK）、APM监控、安全分析等。相比MySQL的LIKE模糊查询，ES可以做到毫秒级返回，而且支持复杂聚合分析。

---

### 2. ES的核心概念有哪些？

#### 2.1 核心术语一览

```
Cluster（集群）：多个ES节点组成
Node（节点）：ES运行实例
Index（索引）：相当于数据库
Document（文档）：相当于数据库一行
Type（类型）：7.x废弃，8.x只有_doc
Field（字段）：文档的属性
_shard（分片）：数据分片
Replica（副本）：提高可用性
```

#### 2.2 Cluster和Node

```
Cluster（集群）：
  - 多个ES节点组成
  - 每个节点有cluster.name
  - 自动选举Master

Node类型：
  - Master eligible：参与选主
  - Data node：存储数据
  - Ingest node：数据预处理
  - Coordinating node：协调请求
```

#### 2.3 Index和Document

```
Index（索引）：
  - 类似数据库
  - 可以有多个type（7.x废弃）
  - Mapping定义结构

Document（文档）：
  - 类似数据库一行
  - JSON格式
  - 唯一标识：_id（可自定义或自动生成）
```

#### 2.4 分片和副本

```
主分片（Primary Shard）：
  - 数据存储单元
  - 创建Index时指定，默认5
  - 不可以修改

副本分片（Replica）：
  - 主分片副本
  - 提高可用性
  - 可以动态调整
```

#### 2.5 回答模板

> ES的核心概念：Cluster是集群、Node是节点、Index是索引（类似数据库）、Document是文档（类似记录）、Field是字段。数据存储在分片（Shard）中，默认主分片5个，可以通过副本提高可用性。每个节点可以扮演不同角色（Master/Data/Ingest/Coordinating）。理解这些概念是ES基础。

---

### 3. ES的架构是怎样的？

#### 3.1 ES架构图示

```
ES Cluster
  ┌──────────────────────────────────────┐
  │        Node1(Master+Data)              │
  │    ┌─Primary Shard 0──→Replica 1    │
  │    │  Primary Shard 2 ─→Replica 0    │
  │    └─Primary Shard4───→Replica2     │
  ├──────────────────────────────────────┤
  │        Node2(Data)                    │
  │    ┌─Primary Shard1───→Replica2     │
  │    │  Primary Shard3───→Replica4    │
  └──────────────────────────────────────┘
```

#### 3.2 工作原理

```
写入流程：
1. 请求发送到Coordinating Node
2. Coordinating Node路由到主分片
3. 主分片写入后并行复制到副本
4. 副本成功返回后，主分片返回成功
5. 1秒后数据可见（NRT）
```

#### 3.3 Master选举

```
集群选举流程：
1. 所有Master eligible节点广播
2. discovery.zen.minimum_master_nodes/2+1参与
3. 票数过半数选Master
4. Master负责集群元数据管理

注意：奇数个节点，建议3个
```

#### 3.4 回答模板

> ES是天然分布式的，通过分片存储数据。每个主分片可以有副本提高可用性。写入时会先路由到主分片，成功复制到副本后才返回。集群通过Zen Discovery机制选Master，建议奇数个节点配置。Data节点负责数据存储，Master节点负责元数据，Coordinating节点负责请求聚合。

---

### 4. ES是如何工作的？

#### 4.1 倒排索引

```
传统正排索引：
  文档ID → 词项列表

ES倒排索引：
  词项 → 文档ID列表

示例：
文档1: "ES是非常好用的搜索引擎"
文档2: "MySQL是关系型数据库"

倒排索引：
  ES → [1]
  是 → [1,2]
  非常 → [1]
  好好 → [1]
  用 → [1]
  搜索引擎 → [1]
  MySQL → [2]
  关系型 → [2]
  数据库 → [2]
```

#### 4.2 分词器Analyzer

```
Analyzer组成：
  1. Character Filter（字符过滤）
  2. Tokenizer（分词）
  3. Token Filter（词过滤）

ES自带分词器：
  - Standard：默认，英文/中文
  - Ik：中文IK分词
  - Pinyin：拼音分词
  - Whitespace：按空格
```

#### 4.3 Document写入过程

```
写入流程：
1. 写入translog（内存buffer）
2. 每秒refresh到filesystem cache（可见）
3. 定期flush到磁盘（持久化）
4. segment合并优化
```

#### 4.4 查询流程

```
查询流程：
1. Coordinating Node收请求
2. 分别查询所有相关Shard
3. 聚合结果返回
4. 排序分页返回
```

#### 4.5 回答模板

> ES的核心是倒排索引，把"词项→文档"映射，实现毫秒级搜索。分词器把文本分解成词素，中文推荐用IK分词。写入时会先到内存，每秒refresh到文件系统缓存后可见，定期flush持久化。查询时由Coordinator节点聚合所有分片结果后返回。

---

## 第二章 索引与Mapping（高频 ★★★★★）

### 5. 如何创建索引？

#### 5.1 创建索引基本语法

```json
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": {"type": "text"},
      "price": {"type": "integer"},
      "create_time": {"type": "date"}
    }
  }
}
```

#### 5.2 Mapping定义

```json
PUT /product
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",          // 全文搜索
        "analyzer": "ik_max_word" // IK分词
      },
      "price": {
        "type": "integer"
      },
      "tags": {
        "type": "keyword"        // 精确匹配
      },
      "location": {
        "type": "geo_point"     // 地理位置
      }
    }
  }
}
```

#### 5.3 ES数据类型

| 类型 | 说明 | 场景 |
|------|------|------|
| text | 全文搜索 | 标题、内容 |
| keyword | 精确匹配 | 状态、标签 |
| integer/long | 整数 | 价��、数量 |
| float/double | 浮点 | 坐标 |
| boolean | 布尔 | 开关 |
| date | 日期 | 时间 |
| geo_point | 地理位置 | 附近搜索 |
| nested | 嵌套对象 | 复杂结构 |
| object | 对象 | 嵌套 |

#### 5.4 回答模板

> 创建索引用PUT命令，可以设置分片数和副本数。mapping定义数据结构，text用于全文搜索会被分词，keyword用于精确匹配不做分词。建议先定义mapping再插入数据，否则ES会自动推断可能不符合预期。常用类型：text、keyword、integer、date、geo_point。

---

### 6. 分词器和Analyzer？

#### 6.1 ES内置分词器

```
Standard：默认分词
Whitespace：按空格分
Simple：按非字母字符分
Keyword：不分词
Language：多语言
```

#### 6.2 中文分词IK

```json
// 安装ik插件
./bin/elasticsearch-plugin install analysis-ik

// 使用IK
POST /_analyze
{
  "analyzer": "ik_max_word",
  "text": "我爱中华人民共和国"
}
// 结果：["我","爱","中华人民共和国","中华","华人","人民","共和国"]
```

#### 6.3 自定义分词器

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "my_stop"]
        }
      },
      "filter": {
        "my_stop": {
          "type": "stop",
          "stopwords": ["the", "a"]
        }
      }
    }
  }
}
```

#### 6.4 text vs keyword

| 类型 | 分词 | 聚合 | 排序 |
|------|------|------|------|
| text | ✅ | ❌ | ❌ |
| keyword | ❌ | ✅ | ✅ |

```json
// 合理设计
PUT /product
{
  "mappings": {
    "properties": {
      "title": {"type": "text"},
      "category": {"type": "keyword"},
      "category_raw": {
        "type": "text",
        "fields": {
          "keyword": {"type": "keyword"}
        }
      }
    }
  }
}
```

#### 6.5 回答模板

> 分词器把文本切分成词素。英文用Standard就够了，中文推荐IK分词。text类型会分词用于搜索但不能聚合排序，keyword不分词用于精确匹配。如果既要搜索又要聚合，可以用fields同时建两种类型。

---

### 7. 什么是Mapping？

#### 7.1 Mapping作用

```
Mapping定义：
- 文档结构
- 字段类型
- 分词规则
- 字段属性
```

#### 7.2 Dynamic Mapping

```
Dynamic三种模式：
1. true：动态添加字段
2. false：忽略新字段
3. strict：新字段报错

PUT /my_index
{
  "mappings": {
    "dynamic": "strict",
    "properties": {...}
  }
}
```

#### 7.3 显示Mapping

```json
GET /my_index/_mapping

// 响应
{
  "my_index": {
    "mappings": {
      "properties": {
        "title": {"type": "text"},
        "price": {"type": "integer"}
      }
    }
  }
}
```

#### 7.4 Meta Fields

```json
PUT /my_index
{
  "mappings": {
    "_routing": {
      "required": true
    },
    "_all": {
      "enabled": false"  // 7.x默认禁用
    },
    "properties": {
      "title": {
        "type": "text",
        "boost": 2.0  // 加权
      }
    }
  }
}
```

#### 7.5 回答模板

> Mapping定义索引的文档结构，可以动态添加也可以显示定义。dynamic可以控制是否自动识别新字段。建议生产环境用strict或false避免意外字段。可以设置字段权重boost来影响相关性排序。

---

### 8. 倒排索��原��？

#### 8.1 正排vs倒排

```
正排索引：
  Doc1 → {id:1, title:"ES教程", price:99}
  Doc2 → {id:2, title:"MySQL教程", price:88}

倒排索引：
  "ES教程" → [Doc1]
  "教程" → [Doc1, Doc2]
  "MySQL教程" → [Doc2]
  "99" → [Doc1]
```

#### 8.2 FST数据结构

```
FST（Finite State Transducer）：
- 词前缀公共复用
- 内存占用小
- 支持前缀查找
```

#### 8.3 索引压缩

```
Term Dictionary：
- 排序存储
- 二分查找

Posting List：
- 分片存储（docId, position）
- 压缩存储（Frame Of Reference）
- 跳表加速合并
```

#### 8.4 回答模板

> ES的倒排索引把词项映射到文档ID列表，实现快速查找。它用FST共享前缀节省内存，用Posting List存文档ID，用Frame of Reference和RBM压缩。对比MySQL的索引，ES的倒排索引特别适合多条件搜索和文本匹配场景。

---

## 第三章 查询与搜索（高频 ★★★★★）

### 9. ES查询语法？

#### 9.1 基本查询

```json
// Match查询（全文搜索）
GET /products/_search
{
  "query": {
    "match": {
      "title": "ES教程"
    }
  }
}

// Term查询（精确匹配）
GET /products/_search
{
  "query": {
    "term": {
      "category": "book"
    }
  }
}

// Terms查询（多值精确匹配）
GET /products/_search
{
  "query": {
    "terms": {
      "category": ["book", "ebook"]
    }
  }
}
```

#### 9.2 Bool查询

```json
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"title": "ES"}},
        {"term": {"category": "book"}}
      ],
      "filter": {
        "term": {"status": "active"}}
      },
      "should": [
        {"term": {"tag": "hot"}}
      ],
      "must_not": [
        {"term": {"deleted": true}}
      ],
      "minimum_should_match": 1
    }
  }
}
```

#### 9.3 复合查询

```json
// Range查询
GET /products/_search
{
  "query": {
    "range": {
      "price": {"gte": 50, "lte": 100}
    }
  }
}

// Exists查询
GET /products/_search
{
  "query": {
    "exists": {"field": "title"}
  }
}

// Wildcard查询
GET /products/_search
{
  "query": {
    "wildcard": {
      "title": "ES*"
    }
  }
}
```

#### 9.4 回答模板

> ES查询用JSON格式：match是全文搜索会分词，term精确匹配不分词。Bool查询可以组合must/filter/should/must_not实现复杂逻辑。filter不评分更快，must评分。Range查范围，Exists查非空。Query DSL很强大，可以组合任意复杂条件。

---

### 10. DSL查询示例？

#### 10.1 常用查询示例

```json
// 查询+排序+分页
GET /products/_search
{
  "query": {"match_all": {}},
  "sort": [{"price": "asc"}],
  "from": 0,
  "size": 10,
  "highlight": {
    "fields": {"title": {}}
  }
}

// 聚合查询
GET /products/_search
{
  "size": 0,
  "aggs": {
    "avg_price": {"avg": {"field": "price"}},
    "category_terms": {"terms": {"field": "category"}}
  }
}
```

#### 10.2 嵌套查询

```json
// 嵌套对象查询
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.content": "good"}
      }
    }
  }
}

// 父子查询
{
  "query": {
    "has_parent": {
      "parent_type": "blog",
      "query": {"match": {"title": "ES"}}
    }
  }
}
```

#### 10.3 Script查询

```json
{
  "script_score": {
    "script": {
      "source": "doc['price'].value * params.factor",
      "params": {"factor": 0.8}
    }
  }
}
```

#### 10.4 回答模板

> ES DSL很丰富，基本查询有match/term/range，复合查询可以用bool组合。嵌套查询用nested/has_child/has_parent。聚合用aggs可以求均值、sum、terms等。脚本查询script可以自定义评分公式。实际使用中经常是查询+聚合+分页+高亮的组合。

---

### 11. 聚合查询有哪些？

#### 11.1 聚合类型

```json
// Bucket Aggregation（桶聚合）
{
  "aggs": {
    "category_bucket": {
      "terms": {"field": "category", "size": 10}
    }
  }
}

// Metric Aggregation（指标聚合）
{
  "aggs": {
    "avg_price": {"avg": {"field": "price"}},
    "max_price": {"max": {"field": "price"}},
    "min_price": {"min": {"field": "price"}},
    "sum_price": {"sum": {"field": "price"}},
    "count": {"value_count": {"field": "price"}},
    "stats": {"stats": {"field": "price"}}
  }
}

// Pipeline Aggregation（管道聚合）
{
  "aggs": {
    "max_price": {"max": {"field": "price"}},
    "avg_bucket": {
      "bucket_selector": {
        "buckets_path": "max_price",
        "script": "params.max_price > 100"
      }
    }
  }
}
```

#### 11.2 嵌套聚合

```json
// 先按category分桶，再算每个桶的平均价格
{
  "size": 0,
  "aggs": {
    "cat_avg": {
      "terms": {"field": "category"},
      "aggs": {
        "avg_price": {"avg": {"field": "price"}}
      }
    }
  }
}
```

#### 11.3 常用聚合函数

| 函数 | 说明 |
|------|------|
| avg/sum/min/max | 基础统计 |
| percentiles | 百分位数 |
| cardinality | 去重计数 |
| top_hits | Top结果 |
| date_histogram | 日期直方图 |

#### 11.4 回答模板

> ES聚合分三类：Bucket（桶分桶）、Metric（计算指标）、Pipeline（管道）。常用聚合有terms分桶、avg/sum/min/max统计、percentiles百分位、cardinality去重。聚合可以嵌套，先分桶再在桶内计算。要注意的是agg内有size:0否则只会返回10个bucket结果。

---

### 12. 查询优化？

#### 12.1 性能影响因素

```
1. 分片过多
2. search白查太多
3. 深分页
4. 复杂聚合
5. 正排+倒排混用
```

#### 12.2 优化方案

```json
// 1. 只返回需要的字段（过滤_source）
GET /products/_search
{
  "_source": ["title", "price"],
  "query": {"match_all": {}}
}

// 2. 查询路由到特定分片
GET /products/_search
{
  "preference": "_shards:1,2,3",
  "query": {"match_all": {}}
}

// 3. 使用filter代替query
{
  "bool": {
    "filter": {"term": {"status": "active"}},
    "must": {"match": {"title": "ES"}}
  }
}

// 4. 避免深度分页，使用scroll或search_after
{
  "search_after": [last_sort_value],
  "sort": [{"id": "asc"}]
}
```

#### 12.3 慢查询分析

```json
// Profile API分析
GET /products/_search
{
  "profile": true,
  "query": {"match_all": {}}
}
```

#### 12.4 回答模板

> ES查询慢的常见原因：分片过多、深分页search from large、复杂聚合、query评分浪费。优化方法：用filter替代query减少评分、_source过滤字段、避免深度分页用scroll或search_after、复杂查询用profile分析。ES默认对query+filter+must排序优化，前者不评分更快。

---

## 第四章 分布式与集群（中高频 ★★★★）

### 13. ES是如何分布式的？

#### 13.1 分片路由

```
文档路由公式：
  shard_num = hash(_routing) % num_primary_shards

默认_routing = _id
```

```json
// 自定义routing
PUT /products/_doc/1?routing=category
{
  "title": "ES Guide",
  "category": "book"
}
```

#### 13.2 分片分配

```
primary分配策略：
- 一个分片的多个副本不能在一个节点
- 同一分片的副本要分布到不同节点

rebalance策略：
- 负载均衡
- 冷热分离
```

#### 13.3 故障转移

```
节点宕机流程：
1. 节点失联
2. Master检查健康
3. 标记副本失败
4. 重新分配副本到其他节点
5. 激活新副本
```

#### 13.4 回答模板

> ES通过hash(_routing)%num_primary_shards计算数据路由到哪个分片。自定义_routing可以优化查询路由到特定分片。副本默认分布在不同节点保证高可用。节点故障时Master会重新分配副本并激活，保证数据不丢失服务不断。

---

### 14. 集群健康状态？

#### 14.1 健康检查

```bash
# 查看集群健康
GET /_cluster/health

# 查看索引健康
GET /_cluster/health?level=indices

# 查看分片健康
GET /_cluster/health?level=shards
```

#### 14.2 三种状态

```
green：所有分片都健康
yellow：主分片健康，副本有问题
red：有主分片不可用
```

#### 14.3 常见问题处理

```bash
# 查看Unassigned分片
GET /_cat/shards?v&s=state:unassigned

# 手动分配
POST /_cluster/reroute
{
  "commands": [
    {"allocate_replica": {
      "index": "products",
      "shard": 1,
      "node": "node_name"
    }}
  ]
}
```

#### 14.4 回答模板

> 集群状态green/yellow/red三种，green最健康，yellow主分片健康副本有问题，red主分片不可用。可以通过_cluster/health查看，也可以_cat/shards?v查看具体分片状态。yellow/red通常是因为节点宕机或磁盘满，可以等恢复或手动reroute。

---

### 15. 如何配置ES集群？

#### 15.1 配置文件

```yaml
# elasticsearch.yml
cluster.name: my-es-cluster
node.name: node-1
network.host: 0.0.0.0
discovery.seed_hosts:
  - node-1:9300
  - node-2:9300
  - node-3:9300
```

#### 15.2 集群基本配置

```yaml
# 脑裂配置（至少3个节点）
discovery.zen.minimum_master_nodes: 2

# 分片分配
cluster.routing.allocation.enable: all
cluster.routing.allocation.allow_rebalance: always

# 节点角色
node.master: true/false
node.data: true/false
node.ingest: true/false
```

#### 15.3 跨集群查询

```json
// 跨集群搜索
GET /cluster-a:products,cluster-b:products/_search
{
  "query": {"match_all": {}}
}
```

#### 15.4 回答模板

> ES集群配置关键是discovery.seed_hosts设成各个节点地址，minimum_master_nodes设成(节点数/2)+1防脑裂。生产环境建议至少3个节点，数据节点和Master节点分开。跨集群用CCR或 Tribes/node来搜索多个集群。

---

## 第五章 实战问题（高频 ★★★★★）

### 16. 如何处理亿级数据查询？

#### 16.1 写入优化

```properties
# bulk写入
action.auto_create_index: false
bulk.size: 5000
concurrent_requests: 10

# refresh_interval
index.refresh_interval: -1  # 写入时关闭
```

```bash
# 写入完成后恢复
PUT /my_index/_settings
{"refresh_interval": "1s"}
```

#### 16.2 写入流程优化

```java
// BulkProcessor批量写入
BulkProcessor bulkProcessor = BulkProcessor.builder(
    (request, bulkListener) -> client.bulkAsync(request, bulkListener),
    new BulkProcessor.Listener() {
        @Override
        public void beforeBulk(long l, BulkRequest request) {}
        @Override
        public void afterBulk(long l, BulkRequest request, BulkResponse response) {}
        @Override
        public void afterBulk(long l, BulkRequest request, Failure failure) {}
    }
).build();
```

#### 16.3 查询优化

```json
// 只返回必要字段
GET /large_index/_search
{
  "_source": ["field1", "field2"],
  "query": {...},
  "size": 100
}
```

#### 16.4 分层架构

```
热数据 → SSD + 只读索引
温数据 → HDD + 正常索引
冷数据 → 归档 + shrink + freeze
```

#### 16.5 回答模板

> 亿级数据处理要点：写入用BulkProcessor批量，设置refresh_interval:-1减少segment生成，查询只返回必要字段并用filter替代query。可以按热度分层，热数据用SSD+只读索引、温数据HDD、冷数据冻结归档。rollover+ILM可以自动管理生命周期。

---

### 17. 分页问题如何处理？

#### 17. from/size深分页问题

```json
// from+size问题
GET /products/_search
{
  "from": 10000,
  "size": 10
}
- 每个分片取10010条
- Coordinating Node合并
- 10000*3分片 = 30030条数据排序
```

#### 17.2 scroll深分页

```bash
# 首次查询
GET /products/_search?scroll=1m
{
  "query": {"match_all": {}},
  "scroll_id": "DXF1ZXJ5QW5kRm50..."

# 获取后续
GET /_search/scroll
{
  "scroll_id": "DXF1ZXJ5QW5kRm50..."
}

# 删除scroll
DELETE /_scroll/_all
```

#### 17.3 search_after深分页

```json
// 第一次
GET /products/_search
{
  "sort": [{"id": "asc"}],
  "size": 10
}

// 后续（使用上次的最后一条排序值）
GET /products/_search
{
  "search_after": [100, "DXF1..."],
  "sort": [{"id": "asc"}],
  "size": 10
}
```

#### 17.4 paginations选择

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| from/size | 简单 | 深分页慢 | 页码<1000 |
| scroll | 能遍历全量 | 快照，中间不能写 | 数据导出 |
| search_after | 高性能 | 需要排序值 | 深层分页 |
| PitP | 不重复 | 需维护point | 不间断翻页 |

#### 17.5 回答模板

> from/size深分页会很慢因为要取很多数据合并。scroll适合数据导出但光snapshot中间不能写入。search_after性能好但需要排序值。生产环境根据场景选择：一般页面展示用from/size到1000页，导出用scroll，需要深分页用search_after。

---

### 18. 集群扩展如何做？

#### 18.1 水平扩展

```bash
# 添加新节点
# 1.解压ES
# 2.配置cluster.name相同
# 3.配置discovery.seed_hosts
# 4.启动自动加入集群
```

#### 18.2 分片再分配

```json
// 手动分配主分片
POST /_cluster/reroute
{
  "commands": [
    {"allocate_empty_primary": {
      "index": "products",
      "shard": 0,
      "node": "node-4"
    }}
  ]
}
```

#### 18.3 Shrink索引

```json
// 收缩索引
POST /large_index/_shrink/my_index_shrunk
{
  "settings": {
    "index.number_of_shards": 1,
    "index.codec": "best_compression"
  }
}
```

#### 18.4 冷热分离

```json
// 索引模板：热数据
{
  "index_patterns": ["hot-*"],
  "settings": {
    "index.routing.allocation.include.box_type": "hot"
  }
}

// 索引模板：冷数据
{
  "index_patterns": ["cold-*"],
  "settings": {
    "index.routing.allocation.include.box_type": "cold"
  }
}
```

#### 18.5 回答模板

> ES水平扩展加节点即可。新节点启动会自动加入集群。可以通过reroute手动分配分片。需要减少分片���以���_shrink合并。生产环境常用冷热分离，热数据用SSD+高副本，冷数据HDD+低副本降低成本。

---

### 19. 如何做监控和运维？

#### 19.1 监控接口

```bash
# 节点健康
GET /_nodes/stats

# 集群状态
GET /_cluster/stats

# 索引状态
GET /_cat/indices?v

# 分片状态
GET /_cat/shards?v

# 阻塞任务
GET /_tasks?detailed=true
```

#### 19.2 常用运维命令

```bash
# 查看节点
GET /_cat/nodes?v

# 查看健康
GET /_cat/health?v

# 清理缓存
POST /_cache/clear

# 关闭索引
POST /my_index/_close

# 打开索引
POST /my_index/_open
```

#### 19.3 修复问题

```json
// 删除异常分片
POST /_cluster/reroute
{
  "commands": [
    {"cancel": {
      "index": "products",
      "shard": 0,
      "task": "node1:r:123456"
    }}
  ]
}
```

#### 19.4 回答模板

> ES运维用_cat接口查看节点、索引、分片、health。集群问题用_cluster/stats排查。可以通过_cache/clear清理缓存。异常分片可以用reroute的cancel命令取消然后重新分配。重要操作前先备份索引。

---

### 20. 如何设计亿级搜索系统？

#### 20.1 整体架构

```
─────────────────────────────
│      Application            │
└─────────────────────────────┘
              ↓
┌─────────────────────────────┐
│     Load Balancer           │
└─────────────────────────────┘
              ↓
┌─────────────────────────────┐
│      ES Coordinating        │
│         Nodes               │
└─────────────────────────────┘
              ↓
┌─────────────────────────────┐
│     ES Data Nodes           │
│  ┌─────┐ ┌─────┐ ┌─────┐   │
│  │S0,1│ │S2,3│ │S4,R│   │
│  └─────┘ └─────┘ └─────┘   │
└─────────────────────────────┘
```

#### 20.2 核心设计

```json
// 1. mapping设计
{
  "dynamic": "strict",
  "properties": {
    "id": {"type": "keyword"},
    "title": {
      "type": "text",
      "analyzer": "ik_max_word",
      "fields": {
        "kw": {"type": "keyword"}
      }
    },
    "category": {"type": "keyword"},
    "price": {"type": "integer"},
    "sales": {"type": "integer"},
    "create_time": {"type": "date"}
  }
}

// 2. 索引设计
{
  "number_of_shards": 5,
  "number_of_replicas": 1,
  "codec": "best_compression",
  "refresh_interval": "5s"
// 5. 写入优化
{
  "refresh_interval": "-1",
  "bulk_size": 5000
}
```

#### 20.3 搜索优化

```json
// 核心查询
{
  "query": {
    "bool": {
      "must": [
        {"multi_match": {
          "query": keyword,
          "fields": ["title^3", "category^2", "brand"],
          "type": "best_fields"
        }}
      ],
      "filter": [
        {"term": {"status": "online"}},
        {"range": {"price": {"lte": maxPrice}}}
      ]
    }
  },
  "sort": [{"_score": "desc"}, {"sales": "desc"}],
  "highlight": {
    "fields": {"title": {}}
  }
}
```

#### 20.4 回答模板

> 亿级搜索系统设计：1）5主分片+1副本，合理mapping用keyword精确匹配、text全文搜索加ik分词；2）写入时关闭refresh_interval批量写入；3）查询用bool+filter组合，关键字段加权boost；4）注意_scroll/size分页限制。生产环境要监控集群状态、及时扩容。

---

## 第六章 高级特性（中高频 ★★★★）

### 21. Document CRUD操作？

#### 21.1 基本CRUD

```bash
#Create
POST /products/_doc
{"title": "ES Guide", "price": 99}

#Read
GET /products/_doc/1

#Update
POST /products/_doc/1/_update
{"doc": {"price": 88}}

#Delete
DELETE /products/_doc/1
```

#### 21.2 bulk操作

```bash
# bulk写入
POST /_bulk
{"index": {"_index": "products"}}
{"title": "Book1", "price": 10}
{"index": {"_index": "products"}}
{"title": "Book2", "price": 20}

# 批量读取
GET /products/_mget
{"ids": ["1", "2"]}
```

#### 21.3 并发控制

```json
// 乐观锁：_version
PUT /products/_doc/1?version=3
{"title": "ES Guide", "price": 88}

// 外部版本号
PUT /products/_doc/1?version_type=external
{"title": "ES Guide", "price": 88}
```

#### 21.4 回答模板

> ES的Document操作：POST/_doc新增、PUT添加或全量更新、POST/_doc/_update增量更新。并发用_version控制，外部版本号用external防止并发冲突。bulk可以一次操作多个doc提高效率。

---

### 22. 什么是Ingest Node？

#### 22.1 Ingest作用

```
Ingest Node：
- 数据预处理
- Pipline管道
- ETL功能

在写入ES前转换数据
```

#### 22.2 Pipeline定义

```json
PUT /_ingest/pipeline/my_pipeline
{
  "description": "Transform data",
  "processors": [
    {
      "set": {
        "field": "created_at",
        "value": "{{_ingestion.timestamp}}"
      }
    },
    {
      "grok": {
        "field": "message",
        "patterns": ["%{TIMESTAMP_ISO8601:timestamp} %{WORD:level} %{DATA:msg}"]
      }
    },
    {
      "convert": {
        "field": "price",
        "type": "integer"
      }
    }
  ]
}
```

#### 22.3 使用Pipeline

```bash
# 写入时使用pipeline
POST /logs/_doc?pipeline=my_pipeline
{"message": "2024-01-01 INFO Started"}

# reindex配合
POST /_reindex
{
  "source": {"index": "old"},
  "dest": {"index": "new", "pipeline": "my_pipeline"}
}
```

#### 22.4 回答模板

> Ingest Node是前置数据处理节点，可以定义Pipeline管道做ETL。常用Processor有set设值、grok解析日志、convert类型转换、remove删除字段。先定义pipeline，文档写入时指定pipeline或reindex时使用即可。

---

### 23. ES和关系型数据库如何同步？

#### 23.1 Logstash同步

```
ES ← JDBC Input → MySQL

input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/es"
    jdbc_user => "root"
    jdbc_password => ""
    statement => "SELECT * FROM products WHERE updated_at > :sql_last_value"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "products"
  }
}
```

#### 23.2 Canal同步

```
MySQL → Binlog → Canal → ES

1. 开启MySQL Binlog
2. 部署Canal Server
3. 配置Canal Adapter写入ES
```

#### 23.3 同步中间件选择

| 方案 | 优点 | 缺点 |
|------|------|------|
| Logstash | 简单、配置灵活 | 无 CDC、实时性一般 |
| Canal | 京东开源、CDC支持 | 需要额外部署 |
| Debezium | ���侵���、CDC | 配置复杂 |
| 自研 | 灵活 | 工作量大 |

#### 23.4 回答模板

> ES和Mysql同步常用：1）Logstash的JDBC插件，简单但实时性一般；2）Canal改自Debezium，支持CDC增量同步；3）自研通过API写入。根据时效性和复杂度要求选择，Logstash够用，增量同步用Canal。

---

### 24. 常见面试问答

```
Q: ES写入慢怎么办？
A: 批量写入、关闭refresh_interval、增加refresh_interval间隔、优化mapping、减少副本

Q: ES查询慢怎么办？
A: 加filter、_ source过滤字段、避免深度分页、profile分析、用keyword替代text

Q: 脑裂问题怎么解决？
A: minimum_master_nodes=(节点数/2)+1、单播代替广播、配置zen.publish_minimum_ne

Q: 集群状态yellow/red怎公办？
A: 检查节点健康、手动reroute、等待自动恢复、如果数据丢失可能需要reindex

Q: 为什么用ES不用Solr？
A: ES天然分布式、比Solr易用、REST API友好、7.x集成beats生态
```

---

## 附录：面试追问

1. **ES vs Solr区别？**
   - ES天然分布式、Solr需要CloudSolr等
   - ES安装配置简单
   - ES RESTful友好

2. **ES集群脑裂怎么预防？**
   - minimum_master_nodes设置
   - 避免网络分区
   - Zen Discovery配置

3. **ES数据如何备份？**
   - snapshot/restore
   - curator定时任务

4. **ES倒排索引优缺点？**
   - 查快+写慢
   - 需要额外存储
   - 适合搜索场景

5. **ES中文分词器有哪些？**
   - IK
   - HanLP
   - Jieba
   - 结巴

---

## 参考资料

- ES官方文档：https://www.elastic.co/guide/index.html
- 《Elasticsearch权威指南》
- ELK Stack权威指南

---

> 整理by Claude Code | ES面试高频100问