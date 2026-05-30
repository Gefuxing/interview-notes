# 系统设计100问——系统设计核心深度指南

> 本文档面向系统设计学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 系统设计基础（高频 ★★★★★）

### 1. 什么是系统设计？

#### 1.1 定义

> 系统设计是根据需求确定系统架构、技术选型和实现细节的过程，以满足可用性、性能、可扩展性等需求。

```plaintext
系统设计关注点：
- 功能实现
- 性能指标
- 可用性
- 扩展性
- 成本控制
```

#### 1.2 回答模板

> 系统设计是软件工程的顶层设计，包括架构选择、技术选型、数据设计，满足业务需求和非功能性需求。

---

### 2. 系统设计vs程序设计？

#### 2.1 区别

```
程序设计：
- 处理单个请求
- 函数调用关系
- 单一进程

系统设计：
- 处理并发请求
- 服务调用关系
- 分布式系统
```

---

### 3. 系统设计的步骤？

#### 3.1 步骤

```plaintext
步骤：
1. 需求分析
2. 能力估算
3. 高层设计
4. 详细设计
5. 技术选型
6. 评审优化
```

---

### 4. 设计原则？

#### 4.1 核心原则

```plaintext
设计原则：
- 简单（Simple）
- 合适（Appropriate）
- 模块化（Modular）
- 可扩展（Scalable）
- 松耦合（Loosely Coupled）
```

---

## 第二章 性能指标（高频 ★★★★★）

### 5. QPS/TPS是什么？

#### 5.1 定义

```
QPS：Queries Per Second（查询/秒）
TPS：Transactions Per Second（事务/秒）
PV：Page View
UV：Unique Visitor
```

#### 5.2 估算

```
QPS估算：
- 日均QPS = PV / 86400
- 峰值QPS = 日均 × 系数(3-6)
- 准实时：峰值QPS预留buffer
```

---

### 6. 并发用户数？

#### 6.1 并发计算

```
并发估算：
- 并发数 = QPS × 平均响应时间
- 活跃用户 = UV × 活跃比例
- QPS = 活跃用户 × 请求频次 / 时间
```

---

### 7. RT响应时间？

#### 7.1 指标

```
响应时间：
- P50：中位数
- P90：90分位
- P99：99分位
- P999：极端场景

标准参考：
- RT < 200ms良好
- RT < 1s可行
```

---

### 8. 吞吐量Throughput？

#### 8.1 定义

```
Throughput：
- 单位时间处理请求数
- 考虑CPU/IO/网络瓶颈
- 压测得到
```

---

### 9. 可用性Availability？

#### 9.1 计算

```
可用性计算公式：
Availability = MTBF / (MTBF + MTTR)
```

```plaintext
可用性等级：
- 99% = 87.6h/年 停机
- 99.9% = 8.76h/年
- 99.99% = 52.6min/年
- 99.999% = 5.26min/年
```

---

### 10. 性能指标之间关系？

#### 10.1 关系图

```
公式关系：
- QPS = 1 / RT RT = 100ms (10 QPS)
- 并发 = QPS × RT RT = 100ms, QPS提升到50，并发=5
- Capacity = Bottleneck × 并发 (木桶原理)
```

---

## 第三章 高性能架构（高频 ★★★★★）

### 11. 垂直扩展vs水平扩展？

#### 11.1 对比

| 维度 | 垂直扩展 | 水平扩展 |
|------|----------|----------|
| 定义 | 加硬件 | 加机器 |
| 复杂度 | 低 | 高 |
| 上限 | 有 | 无 |
| 成本 | 线性 | 线性+ |

#### 11.2 系统设计

```
水平扩展思路：
- 无状态：无状态设计
- 有状态：数据分片
- 缓存：分布式缓存
- 消息：分布式消息
```

---

### 12. Scale Up vs Scale Out？

#### 12.1 两者

- Scale Up：垂直升级硬件
- Scale Out：水平加机器

---

### 13. 无状态设计？

#### 13.1 Stateless���计

```plaintext
无状态设计要求：
- 状态外置（Redis/Memcached）
- Session/Token存储
- 分布式锁
- 请求带上下文
```

---

### 14. 负载均衡Load Balancer？

#### 14.1 类型

```plaintext
LB算法：
- Random：随机
- Round Robin：轮询
- Weighted RR：加权轮询
- Least Connections：最少连接
- IP Hash：源IP哈希
- Consistent Hash：一致性哈希
```

#### 14.2 实现

```
LB分类：
- 硬件负载均衡F5/A10
- 软件LVS/Nginx/HAProxy
- 云厂商NLB/ALB/CLB
```

---

### 15. DNS负载均衡？

#### 15.1 DNS LB

```plaintext
DNS LB：
- 成本低
- 就近访问
- TTL控制
- 不足：无法感知后端状态
```

---

### 16. CDN加速？

#### 16.1 内容分发

```
CDN原理：
- 就近获取
- 缓存内容
- 回源获取
```

```plaintext
CDN场景：
- 静态资源
- 视频点播
- 下载加速
```

---

### 17. 主从读写分离？

#### 17.1 读写分离

```
读写分离：
- 主库write
- 从库read
- 延迟问题考虑
- 数据最终一致性
```

---

### 18. 缓存架构？

#### 18.1 多级缓存

```plaintext
缓存架构：
浏览器 → CDN → Nginx缓存 → 应用缓存(LRU) → Redis/Memcached → DB
```

```
缓存策略：
- Cache-Aside：旁路缓存
- Read Through：读穿透
- Write Through：写穿透
- Write Back：写回
```

---

### 19. 本地缓存vs分布式缓存？

#### 19.1 对比

| 实例 | 本地缓存 | 分布式缓存 |
|------|----------|------------|
| 速度 | 快 | 网络开销 |
| 一致性 | 难 | 易 |
| 容量 | 受限单机 | 大 |
| 场景 | 不变数据 | 热点数据 |

---

## 第四章 高可用架构（高频 ★★★★★）

### 20. 什么是HA？

#### 20.1 高可用

```
高可用High Availability：
- 单点故障消除
- 故障自动切换
- 定期健康检查
```

---

### 21. 主备自动切换？

#### 21.1 Failover

```
自动切换：
- VIP漂移
- 心跳检测
- 脑裂处理（多数投票）
```

---

### 22. 冗余设计？

#### 22.1 Redundancy

```
冗余设计：
- 同城双活
- 异地多活
- 数据多副本
```

---

### 23. 容错设计？

#### 23.1 Fault Tolerance

```
容错机制：
- 请求重试
- 幂等设计
- 熔断降级
- 限流
```

---

### 24. 熔断器Circuit Breaker？

#### 24.1 熔断

```java
// Circuit Breaker状态
- CLOSED：正常
- OPEN：断开
- HALF_OPEN：半开
```

---

### 25. 降级服务降级？

#### 25.1 Degradation

```
服务降级：
- 核心功能保障
- 非核心功能关闭
- 返回缓存/默认值
```

---

### 26. 限流策略？

#### 26.1 Rate Limiting

```
限流算法：
- 计数器算法
- 滑动窗口
- 令牌桶
- 漏桶
```

---

### 27. 防刷策略？

#### 27.1Anti-Crawler

```
防刷手段：
- Captcha验证码
- 限频
- IP黑名单
- 账号异常检测
```

---

### 28. 幂等性设计？

#### 28.1 Idempotency

```
幂等实现：
- 去重表/Set
- 消息ID
- 状态机
- 分布式锁
```

---

### 29. 数据一致性？

#### 29.1 Consistency

```
一致性模型：
- 强一致性
- 弱一致性
- 最终一致性

实现：
- 2PC
- TCC
- 事务消息
- 异步补偿
```

---

### 30. 分布式事务？

#### 30.1 DTC

```plaintext
分布式事务方案：
- 2PC（不推荐）
- TCC（Try-Confirm-Cancel）
- SAGA（补偿）
- 消息事务
- 本地消息表
```

---

## 第五章 存储架构（高频 ★★★★★）

### 31. 分库分表？

#### 31.1 Sharding

```
分库分表策略：
- 垂直拆分：按业务
- 水平拆分：按数据
- 分片键选择
```

#### 31.2 实施

```
分片中间件：
- ShardingSphere
- MyCat
- Vitess
- TDDL
```

---

### 32. 分片键选择？

#### 32.1 Sharding Key

```
选择原则：
- 查询频率高
- 基数适当
- 数据均匀
- 避免副作用
```

---

### 33. 数据库拆分？

#### 33.1 DB Split

```
拆分策略：
- 按业务拆分
- 按冷热拆分
- 按时间拆分
- 按类型拆分
```

---

### 34. 冷热数据分离？

#### 34.1 Cold/Hot Data

```plaintext
冷热分离：
- 近期数据：热存储（SSD/内存）
- 历史数据：冷存储（HDD/对象存储）
- 自动/手动归档
```

---

### 35. 数据同步？

#### 35.1 Sync

```plaintext
数据同步方式：
- Binlog同步（Canal/DTS）
- 消息同步
- 双写同步
```

---

### 36. NoSQL数据库选型？

#### 36.1 NoSQL Selection

```plaintext
选型依据：
- KV：Redis/Memcached
- Document：MongoDB/ES
- Column：HBase/Cassandra
- Graph：Neo4j
- 时序：InfluxDB

场景：
- 缓存穿透：Redis
- 文档搜索：ES
- 大数据：HBase
- 图关系：Neo4j
```

---

### 37. 数据备份策略？

#### 37.1 Backup

```
备份策略：
- 全量备份（周/月）
- 增量备份（日）
- 实时同步（主从延迟）
- 异地容灾
```

---

### 38. SQL优化思路？

#### 38.1 SQL Tuning

```
优化方向：
- 添加适当索引
- 避免全表扫描
- 减少JOIN
- 优化LIMIT
```

---

### 39. 分页实现？

#### 39.1 Pagination

```plaintext
分页方式：
- OFFSET：偏移量大性能差
- ID游标：性能好
- 双向游标：前后翻页
```

---

### 40. 数据库连接池？

#### 40.1 Connection Pool

```
连接池配置：
- 连接数=max(50)
- 连接获取超时=30s
- 连接复用
- 关闭超时=30min
```

---

## 第六章 缓存架构（高频 ★★★★★）

### 41. Redis数据类型？

#### 41.1 Data Types

```plaintext
数据结构：
- String
- Hash
- List
- Set
- Sorted Set
- Bitmap
- HyperLogLog
- GeoSpatial
- Stream
```

---

### 42. Redis持久化？

#### 42.1 Persistence

```plaintext
持久化方式：
- RDB：快照
- AOF：日志追加
-mixed：混合
```

```plaintext
策略：
- RDB：save/bgsave
- AOF：always/everysec/no
-appendonly = yes
```

---

### 43. Redis淘汰策略？

#### 43.1 Eviction

```plaintext
淘汰策略：
volatile-lru（LRU只淘汰已设置过期key）
allkeys-lru（LRU淘汰所有）
volatile-ttl（TTL最早过期）
volatil-random（随机淘汰已过期）
no-eviction（不淘汰）
```

---

### 44. Redis高可用？

#### 44.1 HA

```
HA方案：
- 主从复制
- Sentinel哨兵
- Cluster集群（>=3 master）
```

---

### 45. Redis Cluster？

#### 45.1 Cluster

```
Cluster哈希槽分片：
- 16384 slots
- CRC16(key) % 16384
- 节点Failover
- 自动数据迁移
- Moved/ASK重定向
```

---

### 46. 缓存穿透解决？

#### 46.1 Cache Penetration

```
穿透（缓存+DB都没有）：
- 空值缓存
- BloomFilter
- 白名单
```

---

### 47. 缓存击穿？

#### 47.1 Cache Breakdown

```
击穿（热点key失效）：
- 互斥锁
- 永不过期
- 逻辑过期
```

---

### 48. 缓存雪崩？

#### 48.1 Cache Avalanche

```
雪崩（大量key失效）：
- 加随机TTL
- 互斥锁
- 多级缓存
- 不设TTL（Redis）
```

---

### 49. 布隆过滤器BloomFilter？

#### 49.1 BF

```
BloomFilter原理：
- 位数组存储
- Hash函数映射
- 可能误判
- 不删除
```

---

### 50. Caffeine Cache？

#### 50.1 本地缓存

```
Caffeine配置：
- initialCapacity：初始容量
- maximumSize：最大容量
- expireAfterWrite���写入过期
- refreshAfterWrite：刷新过期
- weakKeys/values：弱引用
```

---

## 第七章 异步架构（高频 ★★★★★）

### 51. 消息队列作用？

#### 51.1 MQ Functions

```
消息队列价值：
- 应用解耦
- 流量削峰
- 异步处理
- 消息分发
```

---

### 52. 消息顺序性保障？

#### 52.1 Messages Order

```
顺序保障：
- 单一队列
- 分区有序
- 业务SeqID
- 服务端排序
```

---

### 53. 重复消息处理？

#### 53.1 Message Idempotent

```
重复处理：
- 业务幂等
- Message ID唯一标识
- 去重表
- 分布式锁
```

---

### 54. 可靠消息？

#### 54.1 Reliable Messaging

```
可靠性：
- 持久化
- 确认（ACK）
- 重试机制
- 幂等
```

---

### 55. 延迟消息实现？

#### 55.1 Delay Message

```
延迟发送：
- 消息TTL + 死信
- 延迟队列插件
- 定时任务扫描
```

---

### 56. 消息优先级实现？

#### 56.1 Message Priority

```
优先级队列：
- 多队列按优先级
- 优先发送高优先级
- 消息带优先级
```

---

### 57. 消息积压处理？

#### 57.1 Message Accumulation

```
处理方案：
- 临时扩容消费者
- 消息批量处理
- 消息转为离线处理
- 消息过期策略
```

---

### 58. RocketMQvs Kafka？

#### 58.1 选型

```

RocketMQ vs Kafka区别：
- 单机吞吐：Kafka更高
- 延迟：RocketMQ更低
- 功能完整性：RocketMQ更全
- 生态：Kafka更强（大数

据）

场景选择：
- 实时业务：RocketMQ
- 大数据流：Kafka
```
---

### 59. 消息追踪实现？

#### 59.1 Tracing

```
消息追踪：
- TraceID链路追踪
- 消息存储消息ID
- 定时上报
- 独立查询系统
```

---

### 60. 消息顺序发送？

#### 60.1 Sequenced Sending

```
顺序发送：
- 单一生产者
- 单一分区
- 消息带SEQ
- 消费者按序消费
```

---

## 第八章 分布式系统（中高级 ★★★★）

### 61. 分布式锁？

#### 61.1 Lock

```plaintext
分布式锁实现：
- Redis SETNX
- Redis RedLock
- Zookeeper瞬时节点
- 数据库排他锁
```

#### 61.2 实现

```lua
-- Redis SET NX EX
SET key value NX EX 30
```

---

### 62. 分布式ID生成？

#### 62.1 ID Generation

```
方案：
- UUID
- Snowflaketwitter
- 数据库号段
- Redis自增
```

---

### 63. 一致性Hash算法？

#### 63.1 Hash Alg

```java
// 一致性哈希
int hash = hash(key);
int bucket = hash % T;

virtual nodes:
virtual_nodes × real_nodes
```

---

### 64. CAP定理？

#### 64.1 CAP Theorem

```plaintext
CAP：
- Consistency一致性
- Availability可用性
- Partition tolerance分区容错
```

```
分布式不会CA全选：
- CP：ZooKeeper/Etcd/HBase
- AP：Cassandra/DynamoDB
- CA不可实现
```

---

### 65. BASE理论？

#### 65.1 BASE Theory

```plaintext
BASE：
- Basically Available
- Soft State
- Eventually consistent

本质是AP牺牲强一致性，得到更好的性能和规模。
```

---

### 66. 一致性协议？

#### 66.1 Consensus Protocols

```plaintext
Paxos：
- 少数服从多数
- Leaderless/Multi-Paxors

Raft（简化版Paxos）：Leader +Follower +Candidate
```

---

### 67. Lease租约机制？

#### 67.1 Lease

```plaintext
Lease租约：
- TTL过期释放
- 续租
- 在选主/分布式锁等使用
```

---

### 68. Gossip协议？

#### 69.1 Gossip

```plaintext
Gossip传播：
- 周期性随机传播
- 最终一致性
- 节点复杂度O(logN)
- Consul/DynamoDB使用
```

---

### 69. Vector Clock？

#### 69.1 时间向量

```java
VectorClock:
{
  node1: 3,
  node2: 2
}
// 对比向量决定数据新旧
```

---

### 70. Merkle Tree？

#### 70.1 Merkle

```plaintext
Merkle树：
- Hash Hash
- 二叉树结构
- 快速比较
- Cassandra数据同步
```

## 第九章 微服务架构（中高级 ★★★★）

### 71. 微服务拆分原则？

#### 71.1 Service Boundaries

```
微服务拆分原则：
- 单一职责
- 业务边界
- 团队边界
- 独立性
- 异步替代同步
```

---

### 72. 服务注册发现？

#### 72.1 Discovery

```plaintext
发现方式：
- 客户端发现（Ribbon）
- 服务端发现（Nginx/Kong）
- 注册中心（Nacos/Eureka/Consul）
```

---

### 73. API Gateway？

#### 73.1 Gateway

```
Gateway功能：
- 路由
- 鉴权
- 限流
- 日志
- 协议转换
```

---

### 74. 服务熔断降级？

#### 74.1 Circuit Breaker

```java
// Spring Cloud Sentinelresilience4j
@CircuitBreaker(name = "serviceA", fallbackMethod = "fallback")
public String call() { ... }

public String fallback(Exception e) { return "降级"; }
```

---

### 75. 全链路追踪？

#### 75.1 Tracing

```plaintext
追踪系统：
- SkyWalking
- Zipkin
- Jaeger

TraceID：
- requestId -> traceId -> spanId
```

---

### 76. 配置中心？

#### 76.1 Config Center

```plaintext
配置中心：
- Apollo
- Nacos
- Spring Cloud Config

配置特点：
- 动态刷新
- 版本管理
- 环境隔离
```

---

### 77. 服务Mock测试？

#### 77.1 Mock Test

```java
// WireMock
stubFor(post(urlEqualTo("/api/user"))
    .willReturn(aResponse()
        .withBody(userJson)
        .withStatus(200)));
```

---

### 78. 服务契约测试？

#### 78.1 Contract Test

```plaintext
PACT测试：
- Consumer Driven Contracts
- Provider验证契约
- 独立演进
```

---

### 79. 服务权限设计？

#### 79.1 Authorization

```plaintext
权限设计：
- 统一认证：OAuth2/OpenID Connect
- 权限校验：RBAC/ABAC
- 鉴权：JWT Token
- 网关拦截
```

---

### 80. 服务监控体系？

#### 80.1 Monitoring

```plaintext
监控内容：
- Metric指标（Prometheus）
- Log日志（ELK）
- Trace链路（SkyWalking）
- Health健康检查

告警：
- 阈值
- 告警收敛
```

---

## 第十章 系统设计案例（中高级 ★★★★）

### 81. 微信扫码登录设计？

#### 81.1 Design

```
流程：
1. 前端展示二维码（ticket）
2. 用户扫码 → Server生成唯一key
3. Server返回二维码+ticket
4. 用户确认 → Server返回Token+openid
5. 前端用Token获取用户信息
```

---

### 82. 红包系统设计？

#### 82.1 Design

```
抢红包设计：
1. 数据库记录红包信息
2. 预估领取人数抢红包时判断
3. Redis记录已抢人数
4. Redis原子扣减
5. 事后补账对账
```

---

### 83. 抽奖系统设计？

#### 83.1 Lottery

```
抽奖设计方式：
- 有券：数据库记录奖项
- 预发：Redis Set记录
- 实时：库存扣减
- 抽奖：概率算法根据并发决定

去重：
- 用户ID Set/数据库
- 每ip/User限制
```

---

### 84. 消息推送系统设计？

#### 84.1 Notification Design

```plaintext
推送设计：
- 设备token存储
- 离线消息存储
- 消息分组batch
- 定时重试

推送通道：
- 极光/APNs/小米
- 统一推送服务
```

---

### 85. Feed流系统设计？

#### 85.1 Feed System

```plaintext
Feed流：
- Timeline Timeline聚合

Pull模式：
- 用户主动拉取
- 缓存用户feed

Push模式：
- 写时推送
- 写扩大

混合：
- 大V Pull
- 小V Push
```

---

### 86. 搜索建议设计？

#### 86.1 Suggestion Design

```plaintext
搜索建议：
- Trie字典树
- 前缀匹配
- 自动补全
- 频率排序

更新：
- 实时更新
- 定时同步
- 异步队列
```

---

### 87. 商品秒杀设计？

#### 87.1 Seckill Design

```plaintext
秒杀设计：
- 独立系统/数据库
- 预热缓存
- 验证码
- IP限流
- 答题限购
- 消息队列削峰
- 库存原子扣减
```

---

### 88. 排行榜设计？

#### 88.1 Ranking Design

```
排名计算：
- 定时batch计算
- Redis ZSet
- 实时分页显示

实现：
- Redis ZRANGEWITHSCORES
- 分数相同按时
```

---

### 89. 评价系统设计？

#### 89.1 Comment design

```
评价设计：
- 订单维度存储
- MongoDB文档
- 评价内容ES索引
- 缓存Redis
- 异步积分
```

---

### 90. 分布式日志收集？

#### 90.1 Log Collection

```
架构选择：
- ELK stack（Elastic+Logstash+Kibana）
- EFK stack（Elastic+Fluentd+Kibana）
- Loki + Grafana

FileBeat：采集
Logstash：处理
Elastic：存储
Kibana：展示
```

---

### 91. 灰度发布设计？

#### 91.1 Canary Release

```
灰度发布：
- 流量切分（权重百分比）
- 按用户ID切分
- 按比例灰度
- 按机房/区域

方案实现：
- Nginx权重
- 司
- Istio
- 路由
```

---

### 92. 双11零点流量设计？

#### 92.1 Peak Design

```
峰值应对：
- 缓存预热
- 限流降级
- 异步非核心
- 静态化页面
- 客服休息室排队
- 动态页面推CDN
```

---

### 93. 订单号设计？

#### 93.1 Order ID Design

```
订单号构成：
- 时间戳（秒）
- 机器ID
- 流水号
- 随机

示例：20240101123455XXXXX
```

---

### 94. 库存超卖解决？

#### 94.1 Overselling Solution

```
方案：
- 数据库乐观锁
- 扣减SQL返回affected rows
- Redis原子扣减Redis decrby
- 消息队列串行化
```

---

### 95. 订单30分钟未支付关闭？

#### 95.1 Close Design

```
关闭实现：
- 定时任务（TMS）
- 消息TTL过期
- delayed消息队列
- 定时轮询
```

---

### 96. 短链系统设计？

#### 96.1 Short Link Design

```plaintext
短链生成：
- ID自增→62进制
- Hash md5/SHA碰撞检查
- Redis计数器

存储：
- 映射关系：Redis+DB持久化
- 请求跳转：302
```

---

### 97. 第三方登录流程？

#### 97.1 OAuth Flow

```
OAuth2.0流程：
1. 前端跳转到授权页
2. 用户确认
3. 回调code
4. 服务端换Token
5. 获取用户信息
6. 登录
```

---

### 98. 视频点播架构？

#### 98.1 VOD Design

```
点播架构：
- 上传：分片上传
- 处理：FFmpeg转码/截图
- 存储：OSS/ES
- 播放：HLS/DASH
- CDN：全球加速
```

---

### 99. CDN回源策略？

#### 99.1 Origin Pull

```
回源策略：
- 首次请求回源
- 回源带Last-Modified/ETag
- 条件请求
- cachecontrol: max-age
```

---

### 100. 系统设计面试准备？

#### 100.1 Interview Preparation

```
面试要点：
- 语言组织能力
- 架构设计经验
- 深度广度兼顾
- 解决问题能力
- STAR法则

常见问题：
请设计一个短网址系统
请设计一个秒杀系统
请设计一个消息推送系统
```

---

## 附录：面试追问

1. **如何评估系统容量？**
   答：以基准TPS评估，考虑峰值、buffer。

2. **如何保证高可用？**
   答：冗余、容错、自动切换。

3. **系统瓶颈在哪里？**
   答：数据库、内存、网络IO。

4. **如何处理突发流量？**
   答：限流、降级、熔断。

5. **系统设计有什么优势？**
   答：可扩展、可维护、成本低。

---

## 参考资料

- 《系统设计面试指南》
- 《高性能分布式计算系统》
- 《数据密集型系统应用设计》
- 《Architecture Patterns》

---

> 整理by Claude Code | 系统设计面试高频100问