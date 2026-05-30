# MongoDB 100问——MongoDB核心技术深度指南

> 本文档面向MongoDB学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 MongoDB基础（高频 ★★★★★）

### 1. 什么是MongoDB？

#### 1.1 定义

> MongoDB是面向文档的NoSQL数据库，使用JSON-like文档存储数据。

```plaintext
MongoDB特点：
- 文档数据库（Document）
- 无模式（Schemaless）
- 高性能
- 高可用
- 易扩展
```

#### 1.2 回答模板

> MongoDB是基于文档的NOSQL数据库，使用BSON存储BSON (Binary JSON)，灵活的模式设计适合快速迭代开发。

---

### 2. MongoDB和数据有什么区别？

#### 2.1 对比

| 特性 | MongoDB | RDBMS(MySQL) |
|------|--------|-------------|
| 数据模型 | 文档 | 行 |
| 表结构 | 集合(Collection) | 表(Table) |
| 模式 | 无模式 | 有模式 |
| 事务 | 4.0+支持多文档 | 完全支持 |
| 扩展 | 水平扩展 | 垂直扩展 |
| JOIN | $lookup聚合 | JOIN |

#### 2.2 回答模板

> MongoDB是无模式文档数据库，对比RDBMS更灵活。4.0后支持多文档事务。

---

### 3. MongoDB数据结构？

#### 3.1 核心概念

```plaintext
核心概念：
- Document：文档，一组键值对(BSON)
- Collection：集合，文档组
- Database：数据库，集合的容器
- _id：主键，自动生成ObjectId
```

#### 3.2 文档示例

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "zhangsan",
  "age": 25,
  "email": "zhangsan@example.com",
  "address": {
    "city": "Beijing",
    "district": "Haidian"
  },
  "tags": ["developer", "coder"],
  "created_at": ISODate("2024-01-01T00:00:00Z")
}
```

#### 3.3 回答模板

> MongoDB的核心数据结构是文档，由键值对组成，使用BSON格式 Binary JSON存储。

---

### 4. BSON是什么？

#### 4.1 定义

> BSON是Binary JSON，二进制编码的JSON，为MongoDB设计。

```
BSON类型：
- String
- Number(Int32/Int64/Double)
- Boolean
- Date
- ObjectId
- Array
- Object
- Binary
- Null
```

#### 4.2 回答模板

> BSON是二进制JSON，是MongoDB的内部存储格式，比JSON更紧凑，支持更多数据类型。

---

### 5. MongoDB基本命令？

#### 5.1 CRUD操作

```bash
# 插入文档
db.users.insertOne({name: "zhangsan", age: 25})
db.users.insertMany([{name: "lisi"}, {name: "wangwu"}])

# 查询文档
db.users.find()
db.users.find({age: {$gte: 18}})
db.users.findOne({name: "zhangsan"})

# 更新文档
db.users.updateOne({name: "zhangsan"}, {$set: {age: 26}})
db.users.updateMany({}, {$inc: {age: 1}})

# 删除文档
db.users.deleteOne({name: "zhangsan"})
db.users.deleteMany({age: {$lt: 18}})
```

#### 5.2 回答模板

> MongoDB使用find/update/delete方法操作集合，基本语法db.collection.method()。

---

### 6. MongoDB的_id字段？

#### 6.1 ObjectId

```javascript
// ObjectId结构
ObjectId("507f1f77bcf86cd799439011")
// 4 bytes timestamp | 3 bytes machine | 2 bytes process | 3 bytes counter
```

#### 6.2 自定义_id

```javascript
// 自定义主键
db.users.insertOne({_id: "my-custom-id", name: "zhangsan"})
```

---

### 7. MongoDB数据类型？

#### 7.1 日期时间

```javascript
// 创建日期
new Date()
ISODate("2024-01-01T00:00:00Z")

// 时间戳
Timestamp
```

#### 7.2 数字类型

```javascript
// NumberInt (32位整数)
NumberInt("123")

// NumberLong (64位整数)
NumberLong("9223372036854775807")

// NumberDecimal (高精度十进制)
NumberDecimal("99.99")
```

#### 7.3 回答模板

> MongoDB有Int/Long/Decimal/int等数值类型，大数值用字符串存储。

---

### 8. MongoDB安装？

#### 8.1 Docker安装

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v mongodb_data:/data/db \
  mongo:latest
```

#### 8.2 yum安装

```bash
# 添加repo
cat > /etc/yum.repos.d/mongodb-org.repo <<EOF
[mongodb-org]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2023/mongodb-org/RPMS/x86_64/
enabled=1
gpgcheck=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
EOF

# 安装
sudo yum install -y mongodb-org
```

---

### 9. MongoDB Compass？

#### 9.1 GUI工具

```bash
# Compass下载
https://www.mongodb.com/products/compass
# 功能：可视化查询、聚合、性能分析
```

#### 9.2 回答模板

> MongoDB Compass是官方GUI工具，可视化查询和分析数据。

---

### 10. MongoDB连接方式？

#### 10.1 连接字符串

```
mongodb://[username:password@]host[:port][/database][?options]
```

#### 10.2 Node.js连接

```javascript
const { MongoClient } = require('mongodb');
const uri = "mongodb://localhost:27017";
const client = new MongoClient(uri);
await client.connect();
const db = client.db("mydb");
```

---

## 第二章 查询操作（高频 ★★★★★）

### 11. find查询？

#### 11.1 基本查询

```javascript
// 查询所有
db.users.find()

// 条件查询
db.users.find({age: 25})

// 字段过滤
db.users.find({}, {name: 1, age: 1, _id: 0})

// 排序
db.users.find().sort({age: -1})  // -1降序，1升序

// 分页
db.users.find().skip(10).limit(10)
```

#### 11.2 比较运算符

```javascript
// $eq 相等
{age: {$eq: 25}}

// $gt/$gte 大于/大于等于
{age: {$gt: 18}}

// $lt/$lte 小于/小于等于
{age: {$lt: 60}}

// $ne 不等于
{city: {$ne: ""}}

// $in/$nin 在/不在数组中
{status: {$in: ["active", "pending"]}}
```

#### 11.3 回答模板

> MongoDB使用find方法配合查询运算符过滤数据，如$gt、$in、$regex。

---

### 12. 逻辑运算符？

#### 12.1 $and/$or/$not/$nor

```javascript
// $and
db.users.find({$and: [{age: {$gte: 18}}, {status: "active"}]})

// $or (或)
db.users.find({$or: [{city: "Beijing"}, {city: "Shanghai"}]})

// $not
db.users.find({age: {$not: {$lte: 18}}})

// $nor (都不)
db.users.find({$nor: [{status: "deleted"}, {status: "banned"}]})
```

---

### 13. 数组查询？

#### 13.1 数组运算符

```javascript
// $all 包含所有
{tags: {$all: ["a", "b"]}}

// $elemMatch 元素匹配
{scores: {$elemMatch: {$gte: 90, $lt: 100}}}

// $size 数组长度
{tags: {$size: 3}}
```

#### 13.2 查询数组元素

```javascript
// 直接匹配
{tags: "developer"}

// $in 任一元素
{tags: {$in: ["developer", "coder"]}}
```

---

### 14. 内嵌文档查询？

#### 14.1 嵌套查询

```javascript
// 点记号访问
{"address.city": "Beijing"}

// 完整匹配
{address: {city: "Beijing", district: "Haidian"}}

// $elemMatch
{address: {$elemMatch: {city: {$exists: true}}}}
```

---

### 15. 正则表达式查询？

#### 15.1 $regex

```javascript
// 模糊匹配
{name: {$regex: "^zhang"}}

// $options
{name: {$regex: "zhang", $options: "i"}} // i忽略大小写

// 性能优化使用索引的正则
{name: {$regex: "^zhang"}} // 前缀匹配用索引
```

---

### 16. null处理？

#### 16.1 查询null

```javascript
// 查询字段不存在的
{email: {$exists: false}}

// 查询值为null
{email: null}

// 查询值为null或字段不存在的
{$or: [{email: null}, {email: {$exists: false}}]}
```

---

### 17. 投影操作？

#### 17.1 字段选择

```javascript
// 包含字段
db.users.find({}, {name: 1, age: 1, _id: 0})

// 排除字段
db.users.find({}, {password: 0, token: 0})

// $slice保留数组前N个元素
{education: {$slice: 2}}

// $elemMatch返回第一个匹配
{scores: {$elemMatch: {$gte: 90}}}
```

---

### 18. 游标操作？

#### 18.1 游标方法

```javascript
// 迭代
db.users.find().forEach(doc => printjson(doc))

// hasNext()/next()
const cursor = db.users.find()
while (cursor.hasNext()) {
  printjson(cursor.next())
}

// count()
db.users.countDocuments({age: {$gt: 18}})
```

---

### 19. 分页查询？

#### 19.1 实现

```javascript
// limit + skip
db.users.find().limit(10).skip(20)

// 游标分页
const page = 2, size = 10
db.users.find().skip((page-1)*size).limit(size)
```

---

### 20. distinct去重？

#### 20.1 distinct

```javascript
// 去重
db.users.distinct("city")

// 带条件去重
db.users.distinct("city", {status: "active"})
```

---

## 第三章 更新操作（高频 ★★★★★）

### 21. update方法？

#### 21.1 更新文档

```javascript
// updateOne更新单条
db.users.updateOne(
  {name: "zhangsan"},
  {$set: {age: 26}}
)

// updateMany更新多条
db.users.updateMany(
  {status: "active"},
  {$set: {vip: true}}
)

// replaceOne替换
db.users.replaceOne(
  {name: "zhangsan"},
  {name: "zhangsan", age: 26}
)
```

---

### 22. 更新运算符？

#### 22.1 $set/$unset

```javascript
// $set设置字段
{$set: {age: 26, email: "new@example.com"}}

// $unset删除字段
{$unset: {token: ""}}

// $inc原子递增
{$inc: {age: 1, score: 10}}

// $mul原子乘
{$mul: {price: 0.9}}
```

#### 22.2 $push/$pull

```javascript
// $push添加到数组
{$push: {tags: "developer"}}

// $push each批量添加
{$push: {tags: {$each: ["a", "b"]}}}

// $pull从数组删除
{$pull: {tags: "unused"}}

// $addToSet添加到set（不重复）
{$addToSet: {tags: "unique"}}
```

#### 22.3 回答模板

> MongoDB有丰富的更新运算符：$set/$inc/$push/$pull等。

---

### 23. upsert？

#### 23.1 插入或更新

```javascript
// upsert:true存在则更新，不存在则插入
db.users.updateOne(
  {name: "zhangsan"},
  {$set: {age: 26}},
  {upsert: true}
)
```

#### 23.2 多用于：

```
upsert用于：
- 计数器
- 统计
- 缓存更新
```

---

### 24. 原子操作？

#### 24.1 原子操作

```javascript
// findAndModify原子操作
db.users.findAndModify({
  query: {_id: 1},
  update: {$inc: {count: 1}},
  new: true
})
```

#### 24.2 回答模板

> findAndModify是原子操作，返回更新前后文档。

---

### 25. bulk批量操作？

#### 25.1 批量

```javascript
const bulk = db.users.initializeUnorderedBulkOp();

// 插入
bulk.insert({name: "a"})
bulk.insert({name: "b"})

// 更新
bulk.find({name: "a"}).updateOne({$set: {age: 20}})

// 删除
bulk.find({name: "b"}).removeOne()

bulk.execute()
```

---

## 第四章 聚合操作（高频 ★★★★★）

### 26. 聚合管道？

#### 26.1 Pipeline

```javascript
// 聚合管道
db.users.aggregate([
  {$match: {status: "active"}},
  {$group: {_id: "$city", count: {$sum: 1}}},
  {$sort: {count: -1}},
  {$limit: 5}
])
```

#### 26.2 Stage

```plaintext
常用Stage：
- $match 过滤
- $group 分���
- $project 投射
- $sort 排序
- $limit/$skip 分页
- $lookup 关联
- $unwind 数组展开
- $addFields/$set 添加字段
```

---

### 27. $match过滤？

#### 27.1 使用

```javascript
// $match放在管道前面（优化性能）
db.orders.aggregate([
  {$match: {status: "completed", created_at: {$gte: ISODate("2024-01-01")}}},
  {$group: {_id: "$product", total: {$sum: "$amount"}}}
])
```

#### 27.2 性能优化

```
$match使用索引：
- $match放在Pipeline最前
- 避免$match包含$or（可以用索引覆盖）
```

---

### 28. $group分组？

#### 28.1 分组统计

```javascript
// 按城市统计人数
db.users.aggregate([
  {$group: {_id: "$city", count: {$sum: 1}}}
])

// 多字段分组
db.orders.aggregate([
  {$group: {_id: {city: "$city", product: "$product"}, total: {$sum: "$amount"}}}
])
```

#### 28.2 累加器

```javascript
// $sum求和
// $avg平均值
// $min/$max最小最大值
// $first/$last首尾
// $push/$addToSet数组
```

---

### 29. $project投射？

#### 29.1 字段变换

```javascript
// 选择字段
db.users.aggregate([
  {$project: {name: 1, age: 1, _id: 0}}
])

// 计算字段
db.orders.aggregate([
  {$project: {total: 1, discount: {$multiply: ["$total", 0.1]}, final: {$subtract: ["$total", "$discount"]}}}
])
```

---

### 30. $lookup关联？

#### 30.1 左外连接

```javascript
// $lookup关联
db.orders.aggregate([
  {$lookup: {
    from: "products",
    localField: "product_id",
    foreignField: "_id",
    as: "product_info"
  }}
])
```

#### 30.2 关联条件

```javascript
// 关联
db.orders.aggregate([
  {$lookup: {
    from: "products",
    let: {productId: "$product_id"},
    pipeline: [
      {$match: {$expr: {$eq: ["$_id", "$$productId"]}}}
    ],
    as: "product"
  }},
  {$unwind: "$product"}
])
```

---

### 31. $unwind展开数组？

#### 31.1 数组展开

```javascript
// 展开tags数组
db.users.aggregate([
  {$unwind: "$tags"},
  {$group: {_id: "$tags", count: {$sum: 1}}}
])
```

#### 31.2 preserveNullAndEmptyArrays

```javascript
// 保留空数组
{$unwind: {path: "$tags", preserveNullAndEmptyArrays: true}}
```

---

### 32. $facet多管道？

#### 32.1 并行管道

```javascript
// 多管道
db.orders.aggregate([
  {$facet: {
    total: [{$count: "count"}],
    byCity: [{$group: {_id: "$city", count: {$sum: 1}}}],
    byMonth: [{$group: {_id: {$month: "$created_at"}, count: {$sum: 1}}}]
  }}
])
```

#### 32.2 回答模板

> $facet在一个查询中执行多个聚合管道。

---

### 33. 聚合表达式？

#### 33.1 条件表达式

```javascript
// $cond三元
{$cond: [{$gte: ["$score", 60]}, "PASS", "FAIL"]}

// $switch分支
{$switch: {
  branches: [
    {case: {$gte: ["$score", 90]}, then: "A"},
    {case: {$gte: ["$score", 60]}, then: "B"}
  ],
  default: "C"
}}
```

---

### 34. 聚合性能优化？

#### 34.1 优化

```plaintext
性能优化：
- $match前置
- $limit + $project减少数据
- $exclude fields不使用
- 使用索引
```

---

### 35. mapReduce？

#### 35.1 mapReduce

```javascript
db.orders.mapReduce(
  function() { emit(this.city, this.amount) },
  function(key, values) { return Array.sum(values) },
  {query: {status: "completed"}, out: "city_totals"}
)
```

---

## 第五章 索引（高频 ★★★★★）

### 36. 索引基础？

#### 36.1 创建索引

```javascript
// 单字段索引
db.users.createIndex({name: 1})

// 复合索引
db.users.createIndex({city: 1, age: -1})

// 唯一索引
db.users.createIndex({email: 1}, {unique: true})

// 稀疏索引
db.users.createIndex({email: 1}, {sparse: true})
```

---

### 37. 索引类型？

#### 37.1 类型

```plaintext
索引类型：
- 单字段索引
- 复合索引
- 多键索引(multikey) - 数组字段
- 全文索引(text)
- 地理位置索引(2dsphere)
- 哈希索引(hashed)
```

#### 37.2 text索引

```javascript
// 全文索引
db.articles.createIndex({content: "text"})
db.articles.find({$text: {$search: "mongodb"}})
```

---

### 38. 复合索引？

#### 38.1 设计原则

```plaintext
复合索引字段顺序：
- 等值查询字段在前
- 范围查询字段在后
- 排序字段最后
```

#### 38.2 例子

```javascript
// 查询：{status: "active", age: {$gt: 18}, created_at: -1}
// 复合索引：{status: 1, age: 1, created_at: -1}
```

---

### 39. 索引特性？

#### 39.1 选项

```javascript
// 唯一
{unique: true}

// 稀疏（仅索引非null值）
{sparse: true}

// TTL（过期自动删除）
{expireAfterSeconds: 3600}

// 部分索引
{partialFilterExpression: {status: "active"}}
```

---

### 40. 索引分析explain？

#### 40.1 执行计划

```javascript
db.users.find({name: "zhangsan"}).explain("executionStats")
```

```plaintext
explain输出：
- IXSCAN 使用索引扫描
- COLLSCAN 全表扫描
- FETCH 根据索引取文档
```

---

### 41. 索引覆盖？

#### 41.1 Covered Queries

```javascript
// 覆盖查询（索引包含所有字段）索引：{name: 1, age: 1}
// 查询只返回索引字段
db.users.find({name: "zhangsan"}, {name: 1, age: 1, _id: 0})
```

---

### 42. 索引重建/删除？

#### 42.1 操作

```javascript
// 查看索引
db.users.getIndexes()

// 删除索引
db.users.dropIndex("name_1")
db.users.dropIndexes() // 删除所有

// 后台创建
db.users.createIndex({email: 1}, {background: true})
```

---

## 第六章 副本集（高频 ★★★★★）

### 43. Replica Set？

#### 43.1 副本集

```
副本集成员：
- Primary 主节点（写入）
- Secondary 从节点（复制）
- Arbiter 仲裁者（投票）
```

#### 43.2 配置

```javascript
// 初始化副本集
config = {
  _id: "rs0",
  members: [
    {_id: 0, host: "mongo1:27017"},
    {_id: 1, host: "mongo2:27017"},
    {_id: 2, host: "mongo3:27017"}
  ]
}
rs.initiate(config)
```

---

### 44. 副本集操作？

#### 44.1 查看状态

```javascript
// 状态
rs.status()
rs.isMaster()

// 投票
rs.add("mongo4:27017") // 添加
rs.remove("mongo4:27017") // 删除
```

---

### 45. 读写分离？

#### 45.1 Read Preference

```javascript
// 读偏好
db.users.find().readPref("primary")
db.users.find().readPref("secondary")
db.users.find().readPref("nearest")
db.users.find().readPref("secondaryPreferred")
```

---

### 46. 选举机制？

#### 46.1 选举

```
选举触发条件：
- 主节点不可达
- 副本集中超过半数节点不可达
- rs.stepDown() 手动降级
```

---

### 47. 故障转移？

#### 47.1 自动failover

```
故障转移：
- Primary不可达 → 选举新Primary
- 选举完成 → 自动恢复读写
- 配置arbiter确保奇数票
```

---

### 48. oplog？

#### 48.1 操作日志

```javascript
// oplog
db.oplog.rs.find()

// 大小
db.getSiblingDB('local').oplog.rs.stats()
```

---

### 49. 成员状态？

#### 49.1 状态

```
成员状态：
- PRIMARY
- SECONDARY
- RECOVERING
- STARTUP2 (初始化同步)
- ARBITER
- DOWN
- UNKNOWN
```

---

### 50. 副本集安全？

#### 50.1 认证

```javascript
// 密钥认证
// 1. 创建keyFile
openssl rand -base64 756 > keyFile

// 2. 配置mongod.conf
security:
  keyFile: /path/to/keyFile

replication:
  replSetName: "rs0"
```

---

## 第七章 分片集群（高频 ★★★★★）

### 51. Sharding分片？

#### 51.1 分片架构

```
分片集群组件：
- Shard Server 分片服务器
- Config Server 配置服务器
- Mongos Router 查询路由器
```

#### 51.2 分片键

```
分片键选择：
- 查询频率高的字段
- 基数（基数不能太小或太大）
- 不能是可变数组字段
```

---

### 52. 分片集群配置？

#### 52.1 步骤

```javascript
// 1. 初始化config server
mongod --configsvr --replSet configReplSet --dbpath /data/config --port 27019

// 2. 初始化shard server
mongod --shardsvr --replSet shardReplSet --dbpath /data/shard --port 27018

// 3. 启动mongos
mongos --configdb configReplSet/config1:27019,config2:27019 --port 27017
```

---

### 53. sharded collections？

#### 53.1 启用分片

```javascript
// 1. 数据库启用分片
sh.enableSharding("mydb")

// 2. 集合启用分片
sh.shardCollection("mydb.users", {user_id: 1})
```

---

### 54. 平衡器Balancer？

#### 54.1 均衡

```javascript
// 查看状态
sh.startBalancer()
sh.stopBalancer()
sh.isBalancerRunning()
```

---

### 55. chunk管理？

#### 55.1 Chunk操作

```javascript
// 分裂
sh.splitAt("mydb.users", {user_id: 100})

// 迁移
sh.moveChunk("mydb.users", {user_id: 50}, "shard0001")
```

---

### 56. 片键策略？

#### 56.1 策略

```
片键策略：
- 基于范围 Ranged
- 基于哈希 Hashed
- 地理空间 Zones
```

---

### 57. 分片查询？

#### 57.1 $merge改写

```javascript
// 将结果写入分片集合
db.orders.aggregate([
  {$match: {status: "completed"}},
  {$merge: {into: "orders_report"}}
])
```

---

### 58. 集群监控？

#### 58.1 监控

```javascript
// shard状态
db.printShardingStatus()

// chunks信息
db.getSiblingDB('config').chunks.countDocuments({ns: "mydb.users"})
```

---

## 第八章 事务与安全（高频 ★★★★★）

### 59. 事务？

#### 59.1 多文档事务

```javascript
// 开启事务
const session = client.startSession();
session.withTransaction(async () => {
  const db = client.db("mydb");
  await db.collection("accounts").updateOne(
    {accountId: "A"},
    {$inc: {balance: -100}},
    {session}
  );
  await db.collection("accounts").updateOne(
    {accountId: "B"},
    {$inc: {balance: 100}},
    {session}
  );
});
```

#### 59.2 限制

```
4.0前不支持多文档事务，之后支持单分片多文档事务。5.0+支持分布式事务。
```

---

### 60. 集群中的事务？

#### 60.1 分布式事务

```javascript
// MongoDB 5.0+ 分布式事务
session.withTransaction(async () => {
  // 跨分片操作
});
```

---

### 61. 认证？

#### 61.1 认证方式

```
认证方式：
- SCRAM-SHA-1 (默认)
- SCRAM-SHA-256
- x.509证书
- LDAP (企业版)
- Kerberos (企业版)
```

#### 61.2 配置

```javascript
// 创建用户
db.createUser({
  user: "admin",
  pwd: "password",
  roles: [
    {role: "userAdminAnyDatabase", db: "admin"},
    {role: "readWriteAnyDatabase", db: "admin"}
  ]
})
```

---

### 62. 角色管理？

#### 62.1 内置角色

```plaintext
Built-in Roles：
- read / readWrite
- dbAdmin / dbOwner / userAdmin
- clusterAdmin / clusterMonitor / clusterOwner
- root
- backup / restore
```

#### 62.2 自定义角色

```javascript
db.createRole({
  role: "customRole",
  privileges: [
    {resource: {db: "mydb", collection: "users"}, actions: ["find", "update"]}
  ],
  roles: []
})
```

---

### 63. 权限控制？

#### 63.1 资源级别

```javascript
// 资源权限
{resource: {db: "mydb", collection: "users"}, actions: ["find"]}
{resource: {db: "mydb", collection: ""}, actions: ["dbStats"]} // 整个db
{resource: {db: "", collection: ""}, actions: ["serverStatus"]} // admin
```

---

### 64. TLS/SSL？

#### 64.1 加密

```yaml
# mongod.conf配置
net:
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/ssl/mongodb.pem
```

---

### 65. 审计Audit？

#### 65.1 审计配置

```yaml
# 配置审计
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.log
```

---

## 第九章 性能优化（高频 ★★★★★）

### 66. 性能分析工具？

#### 66.1 工具

```javascript
// explain
db.users.find({name: "zhangsan"}).explain("executionStats")

// currentOp
db.currentOp()

// top命令
db.adminCommand({top: 1})
```

---

### 67. Profiler性能分析？

#### 67.1 开启Profiler

```javascript
// 开启
db.setProfilingLevel(1, {slowms: 100})

// 查询日志
db.system.profile.find()

// 关闭
db.setProfilingLevel(0)
```

---

### 68. 性能优化策略？

#### 68.1 优化

```
优化策略：
- 合理使用索引
- 减少返回字段
- 使用Covered Queries
- 查询尽量带上分片键
- 高并发用连接池
```

---

### 69. 连接池？

#### 69.1 连接管理

```javascript
// Java driver
MongoClientOptions.builder()
  .connectionsPerHost(100) // 最大连接数
  .threadsAllowedToBlockForConnection(50)
  .maxWaitTime(120000) // 最大等待
  .connectTimeout(10000)
  .socketTimeout(60000)
```

---

### 70. 慢查询优化？

#### 70.1 优化

```javascript
// 优化慢查询
1. 添加索引
2. 优化查询条件
3. 使用projection
4. 分页优化
5. 避免$where
```

---

### 71. 内存使用？

#### 71.1 WiredTiger缓存

```yaml
# 配置内存
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4  # 建议50%RAM减去1GB
```

---

## 第十章 备份与恢复（中高级 ★★★★）

### 72. 备份方式？

#### 72.1 mongodump

```bash
# 全量备份
mongodump --host localhost --port 27017 -u admin -p password -o /backup/

# 备份单库
mongodump --db mydb --out /backup/
```

#### 72.2 物理备份

```bash
# 停止MongoDB
# 复���数据文件
tar -czvf backup.tar.gz /data/db/*
```

---

### 73. 恢复mongorestore？

#### 73.1 恢复

```bash
# 全量恢复
mongorestore --host localhost --port 27017 -u admin -p password /backup/

# 恢复单库
mongorestore --db mydb --drop /backup/mydb/
```

---

### 74. Oplog备份？

#### 74.1 增量备份

```bash
# 备份oplog
mongodump --db local --collection oplog.rs --query '{ts: {$gt: Timestamp}}'
```

---

### 75. Cloud备份？

#### 75.1 云服务

```
云备份服务：
- MongoDB Atlas (官方云)
- MongoDB Enterprise Cloud
- Ops Manager备份
```

---

## 第十一章 集群运维（中高级 ★★★★）

### 76. 配置备份？

#### 76.1 config server

```bash
# 备份config
mongodump --db config -o /backup/config
```

---

### 77. 升级MongoDB？

#### 77.1 升级

```
升级路径：
- 4.4 → 5.0 → 6.0 → 7.0
- 大版本跨越需先升级到中间版本
```

---

### 78. 存储引擎？

#### 78.1 WiredTiger

```
存储引擎：
- WiredTiger (默认)
- In-Memory
- MMAPv1 (4.2废弃)
```

---

### 79. 集群监控？

#### 79.1 监控指标

```
监控指标：
- QPS
- 延迟
- 内存使用
- 磁盘IO
- 连接数
- oplog lag
```

---

### 80. 常见错误？

#### 80.1 错误

```
常见错误：
- not master / secondaryelaytoo large
- TooManyMounts
- Journal not aligned
```

---

## 第十二章 MongoDB最佳实践（中高级 ★★★★）

### 81. Schema设计原则？

#### 81.1 设计

```
Schema设计原则:
- 文档大小<16MB字段合适
- 避免使用过长字段名
- 使用规范的分层结构
- 适当冗余避免join
```

---

### 82. 关联设计？

#### 82.1 引用vs内嵌

```
内嵌：
- 一对少量少 → 内嵌
- 数据一起读取
- 实时性要求高

引用：
- 一对大量大 → 引用
- 数据单独使用
- 复用场景
```

---

### 83. 片键选择？

#### 83.1 片键

```
片键选择：
- 高基数字段
- 查询频率高
- 不选可变数组
- 避免单调递增
```

---

### 84. 命名规范？

#### 84.1 命名

```plaintext
命名规则：
- 集合名：英文复数(users)
- 字段名：小写下划线(user_id)
- 避免$，开头
```

---

### 85. 变更streams？

#### 85.1 CDC

```javascript
// Change Streams
const changeStream = db.watch()
changeStream.on("change", change => console.log(change))
```

---

### 86. 事务块大小？

#### 86.1 16MB限制

```
MongoDB文档限制：
- 文档大小 < 16MB
- 单个BSON元素 < 1MB
- 索引键 < 1KB
```

---

### 87. GridFS存文件？

#### 87.1 大文件

```javascript
// GridFS存大文件
mongofiles put largefile.mp4
mongofiles list
mongofiles get largefile.mp4
```

---

### 88. TTL索引？

#### 88.1 自动过期

```javascript
// TTL索引
db.session.createIndex({createdAt: 1}, {expireAfterSeconds: 3600})
```

---

### 89. 数据库设计？

#### 89.1 分片设计

```
按业务分库：
- 用户库(users)
- 订单库(orders)
- 业务库(business)
```

---

### 90. 性能监控工具？

#### 90.1 免费工具

```plaintext
监控工具：
- Cloud: Atlas免费
- 自建: Prometheus + Grafana
- MMS: MongoDB Cloud Manager
```

---

## 第十三章 应用集成（中高级 ★★★★）

### 91. Spring Data MongoDB？

#### 91.1 Spring Boot

```java
// Maven依赖
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

#### 91.2 Repository

```java
public interface UserRepository extends MongoRepository<User, String> {
    Optional<User> findByName(String name);
}
```

---

### 92. mongoose？

#### 92.1 Node.js ODM

```javascript
// Schema定义
const userSchema = new Schema({
  name: String,
  age: Number,
});
const User = mongoose.model('User', userSchema);
```

---

### 93. Go MongoDB驱动？

#### 93.1 Go驱动

```go
// go.mongodb.org/mongo-driver
client, err := mongo.Connect(context.Background(),
    options.Client().ApplyURI("mongodb://localhost"))
```

---

### 94. PyMongo？

#### 94.1 Python驱动

```python
from pymongo import MongoClient
client = MongoClient('mongodb://localhost:27017')
db = client['mydb']
```

---

### 95. Aggregation管道Java？

#### 95.1 Java聚合

```java
Aggregation aggregation = newAggregation(
  match(Criteria.where("status").is("active")),
  group("city").count("total"),
  sort(Sort.Direction.DESC, "total")
);
```

---

### 96. 分布式ID生成？

#### 96.1 ObjectId

```
ObjectId组成：
- 时间戳 4字节
- 机器ID 3字节
- 进程ID 2字节
- 计数器 3字节
```

---

### 97. 日志收集？

#### 97.1 日志

```javascript
// MongoDB日志
/var/log/mongodb/mongod.log
systemd: journalctl -u mongod
```

---

### 98. 数据迁移？

#### 98.1 迁移

```
迁移工具：
- 原生mongodump/mongorestore
- Studio 3T 工具
- MongoDB Compass导入导出
```

---

### 99. 性能基准测试？

#### 99.1 YCSB

```bash
# YCSB
./ycsb load mongodb -P workloads/workloada -p recordcount=100000
./ycsb run mongodb -P workloads/workloada -p recordcount=100000
```

---

### 100. 学习资源？

#### 100.1 文档

```
学习资源：
- MongoDB官方文档
- MongoDB University
- MongoDB Atlas免费丛
```

---

## 附录：面试追问

1. **谈谈你项目中的MongoDB使用？**
   答：结合项目业务说明使用场景

2. **MongoDB的优缺点？**
   答：优点灵活、高扩展，缺点事务弱、大数据量查询有限制

3. **为什么选择MongoDB？**
   答：业务数据文档结构,json类型，不需要join

4. **如何保证数据一致性？**
   答：副本集保证高可用，事务保证一致性

5. **分片集群如何查询？**
   答：带分片键查一个分片，不带查全部分片

---

## 参考资料

- MongoDB Manual
- MongoDB University
- 《MongoDB: The Definitive Guide》

---

> 整理by Claude Code | MongoDB面试高频100问