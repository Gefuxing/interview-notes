# 分布式与微服务夺命连环100问——分布式核心技术深度指南

> 本文档面向分布式系统学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是分布式系统？

#### 1.1 分布式定义

> 分布式系统是由多个独立计算机组成的系统，对用户表现为单一计算机。核心特征：分布透明性、扩展性、容错性。

```plaintext
中心化 vs 分布式：

中心化：       分布式：
┌───────┐    ┌───────┐ ┌───────┐ ┌───────┐
│ 单点  │    │ 节点1 │ │ 节点2│ │ 节点3 │
│ 瓶颈  │    └───────┘ └───────┘ └───────┘
└───────┘          ↓       ↓       ↓
                ┌───────────────────┐
                │  统一对外服务   │
                └───────────────────┘
```

#### 1.2 核心特性

```plaintext
分布式特性：
1. 透明性：Location、Migration、Replication透明
2. 可扩展：水平扩展（加机器）
3. 容错性：部分失败不导致系统崩溃
4. 并发性：支持并发请求
5. 透明性：一致性、可用性、分区容错（CAP）
```

#### 1.3 回答模板

> 分布式系统是多台计算机协同工作呈现单点体验的系统，核心解决单点性能瓶颈和高可用问题。分布式的挑战是网络不可靠、节点会失败、需一致性。分布指的是物理部署，透明是对用户隐藏。

---

### 2. CAP定理？

#### 2.1 CAP定义

> CAP：Consistency一致性、Availability可用性、Partition Tolerance分区容错，三者只能同时满足两个。

```plaintext
      Consistency
           /\
          /  \
         /    \
        /  P   \
       /  \    /
      /   \   /
  Availability
          /\ /\
         /  X  \
        /  / \  \
    PartitionTolerance
```

#### 2.2 权衡选择

```plaintext
CP（一致性+分区容错）：
- ZooKeeper、HBase、Etcd
- 放弃可用性，保证一致性

AP（可用性+分区容错）：
- Cassandra、DynamoDB
- 放弃强一致性，保证最终一致

CA（不考虑分区）：
- 单机数据库
- 实际不存在
```

#### 2.3 BASE理论

```plaintext
BASE理论（Basically Available, Soft state, Eventually consistent）：
- Available：基本可用（可降级）
- Soft state：软状态（中间态）
- Eventually consistent：最终一致

是CAP的实践版，实际系统的选择
```

#### 2.4 回答模板

> CAP是分布式的基本定理：一致、可用、分区容错（网络分割时）。现实必须选PA，因为网络会出问题。Base是实际妥协：基本可用+最终一致。分布式一致性用最终一致+补偿事务。

---

### 3. 什么是一致性哈希？

#### 3.1 原理

```plaintext
传统哈希： hash(key) % N
问题：节点数变导致大规模数据迁移

一致性哈希：
- 哈希空间是0~2^32-1的环
- 节点分布在环上
- key顺时针找到第一个节点

            key
              ↓
        ┌───────────────┐
        │              ↓
   ●───┼────●────────────●──
   A   |         B    C    环
        ↑              ↑
        │              │
        └──key迁移──-->│
        改B不���响A
```

#### 3.2 虚拟节点

```plaintext
虚拟节点（VNode）：
- 每个物理节点对应多个虚拟节点
- 解决负载不均
- 如每个物理节点100个虚拟节点

节点A → A#1,A#2,...A#100
节点B → B#1,B#2,...B#100
```

#### 3.3 回答模板

> 一致性哈希用环形解决节点增减时数据迁移问题，key找顺时针第一节点。加虚拟节点解决倾斜/hash不均匀。Redis/Cassandra/Ketesize都用一致性哈希。数据倾斜用虚拟节点均衡。

---

### 4. 分布式ID生成？

#### 4.1 方式

```java
// 1. UUID
UUID.randomUUID().toString();
// 32字符16字节太长+无序

// 2. 数据库自增
// 单点瓶颈、跨数据库无法保证

// 3. 雪花算法 Snowflake
// 41位时间戳+10位机器ID+12位序列号
// 64位long
// 毫秒级有序
// 单机每秒约400万

// 4. Leaf算法（美团）
// 公司号+自增号段
```

```java
// 雪花实现
public class IdWorker {
    private long workerId;
    private long sequence;
    private long twepoch = 1288834974657L;

    public synchronized long nextId() {
        long timestamp = timeGen();
        if (timestamp < lastTimestamp) {
            throw new RuntimeException("Clock moved backwards");
        }
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            if (sequence == 0) {
                timestamp = tilNextMillis();
            }
        } else {
            sequence = 0;
        }
        lastTimestamp = timestamp;
        return ((timestamp - twepoch) << timestampLeftShift)
             | (workerId << workerIdShift)
             | sequence;
    }
}
```

#### 4.2 分布式ID方案对比

| 方案 | 优点 | 缺点 | 适用 |
|------|-----|------|------|
| UUID | 无中心 | 太长无序 | 日志ID |
| 数据库自增 | 简单有序 | 单点/跨库 | 单库 |
| 雪花算法 | 低延迟高性能 | 时钟依赖 | 高并发 |
| Leaf | 组件化 | 依赖ZK | 美团生态 |
| UidGenerator | 无时钟 | 需要cache | 百度 |

#### 4.3 回答模板

> 分布式ID方案有：UUID太长无序，数据库自增单点瓶颈，雪花算法41位时间戳+10位机器+12位序列高性能，Leaf是美团的号段模式。雪花算法是本地生成不依赖外部，适合高并发。

---

### 5. 分布式事务？

#### 5.1 刚性 vs 柔性

```plaintext
刚性事务（ACID）：
- 分布式数据库的XA协议
- 两阶段提交（2PC）
- 优点：强一致
- 缺点：阻塞时间长

柔性事务（BASE）：
- TCC、Saga、AT
- 最终一致
- 性能好
```

```plaintext
2PC (Two Phase Commit):
1. Prepare：协调者问各个参与者能否提交
2. Commit：都准备好则提交，否则回滚

问题：
- 协调者单点
- 阻塞
- 数据不一致
```

#### 5.2 TCC模式

```java
// Try：预留资源
@LocalTCC
public interface PlayerService {
    @TwoPhaseBusinessAction(method = "confirm")
    void tryReduceBalance(@BusinessActionContextParameter(parameterName = "accountId") String accountId,
                         @BusinessActionContextParameter(parameterName = "amount") BigDecimal amount);

    // Confirm：确认
    void confirm();

    // Cancel：取消
    void cancel();
}
```

#### 5.3 Saga模式

```java
// Saga：每个服务有正向和补偿操作
// 正向：扣款
// 补偿：退款

// 编排方式：
// 顺序编排：SagaRequestFlow
// 状态机：Spring Statemachine
```

#### 5.4 AT模式

```java
@GlobalTransactional
@Transactional
public void purchase() {
    // 自动生成undolog回滚
    // 自动拦截SQL
}
```

#### 5.5 回答模板

> 分布式事务有刚性2PC和柔性TCC/Saga/AT。2PC有协调者单点阻塞问题，TCC空位预留资源后确认/回滚，AT是Seata模式自动回滚。最终一致���+补偿是实际选择，不要求强一致。

---

### 6. 一致性协议Paxos？

#### 6.1 Paxos角色

```plaintext
Paxos角色：
- Proposer：提议者
- Acceptor：接收者
- Learner：学习者（最终知晓结果）

流程（basic paxos）：
1. Prepare： proposer编号N，广播给acceptor
2. Promise： acceptor承诺不批准小于N的提案，返回最大编号的value
3. Accept：proposer收到多数响应，选择一个value，广播Accept
4. Accepted：acceptor批准提案，记入本地
```

#### 6.2 Multi-Paxos

```plaintext
Multi-Paxos（连续多个instance）：
- 选Leader：多个Proposer选一个Leader
- 之后只有Leader发提案
- 简化流程，提高效率
```

#### 6.3 相关算法

```plaintext
- Raft（易理解版Paoxos）
  - Leader选举
  - 日志复制
  - 成员变更

- Zab（ZooKeeper原子广播）
  - 类似Paxos
  - 用于ZK
```

#### 6.4 回答模板

> Paxos是共识算法用于选举，Basic Paxos多轮Prepare/Accept。Multi-Paxos选Leader后续简化。Raft是易理解的Paxos实现（日志复制+Leader选举+安全）。ZooKeeper用Zab，etcd用Raft。

---

### 7. Raft算法？

#### 7.1 角色

```
Raft角色：
- Leader：领导者，写操作先进入日志，然后复制
- Follower：从领导者同步日志
- Candidate：候选者，用于Leader election

State:
- Leader→heartbeats维持领导
- Follower→超时未收到心跳→Candidate
- Candidate→收到多数投票→Leader
```

#### 7.2 任期Term

```
Term概念：
- Term是任期编号，连续单调递增
- 选举：Term开始
- Leader在Term内服务
- 遇到更高Term自动变为Follower

选举过程：
1.Follower超时，选举超时
2.Increment Term，转Candidate
3.Vote给自己，发RequestVote
4.收到多数，成为Leader
5.Heartbeat通知其他
```

#### 7.3 日志复制

```
日志复制Log Replication：
1.Client → Leader: set 5
2.Leader写本地日志（uncommitted）
3.AppendEntries发给Follower
4.多数确认
5.Apply到状态机
6.Respond Client

问题：
- 日志不一致
- Leader发AppendEntries带prevLogIndex/Term
- Follower不匹配拒绝，Leader递减重试
- 最终一致
```

#### 7.4 回答模板

> Raft是分布式共识算法，易理解。三个角色Leader/Follower/Candidate，任期Term表示。Leader选举Heartbeats维系，日志复制AppendEntries同步，收到多数确认apply。Raft比Paxos易实现，etcd/Consul/K8s用Raft。

---

## 第二章 服务治理篇（高频 ★★★★★）

### 8. 什么是服务注册发现？

#### 8.1 注册/发现

```plaintext
服务发现两种模式：
- 客户侧发现：不依赖中心，消费者直连实例
  优点：少一跳、无单点
  缺点：负载不均、能力未知

- 服务端发现：中心路由
  优点：统一路由、智能负载
  缺点：中心转发、需要高可用
```

#### 8.2 Eureka

```java
// 服务端
@EnableEurekaServer

// 客户端
@EnableDiscoveryClient
@Value("${spring.application.name}")
String appName;
```

```properties
eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    fetch-registry: true
  server:
    enable-self-preservation: true
```

#### 8.3 Nacos

```properties
spring:
  cloud:
    nacos:
      discovery:
        namespace: ${NACOS_NAMESPACE:}
        group: ${NACOS_GROUP:}
        server-addr: ${NACOS_SERVER:}
```

```java
@NacosInjected
private NamingService namingService;
```

#### 8.4 Consul

```properties
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

#### 8.5 回答模板

> 服务注册发现有客户端发现（Eureka/Nacos/Consul直接）和服务端发现（nginx/API gateway���。常用中心路由：Eureka（Neflix）、Nacos（阿里）、Consul（Spring Cloud原生）、zk。Eureka和Nacos都是AP模型。

---

### 9. 负载均衡？

#### 9.1 客户端负载均衡

```java
// Ribbon负载均衡
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// Feign集成Ribbon
@FeignClient(name = "provider")
public interface ProviderClient { }

// 配置
provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule
    ConnectTimeout: 1000
    ReadTimeout: 3000
```

#### 9.2 负载策略

```
策略类型：
- 简单轮询：RoundRobinRule
- 随机：RandomRule
- 响应时间：WeightedResponseTimeRule
- 最少并发：BestAvailableRule
- 可用过滤：AvailabilityFilteringRule
- 幂等重试：RetryRule
```

#### 9.3 服务端负载均衡

```java
// Nginx upstream
upstream provider {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
    ip_hash; // session保持
}

// 权重
server 127.0.0.1:8080 weight=3;
server 127.0.0.1:8081 weight=1;
```

#### 9.4 回答模板

> 负载均衡分客户端Ribbon和服务器端Nginx。Ribbon是进程内负载均衡，Feign集成。常用策略RoundRobin（默认）、Random、WeightedResponseTime（按响应时间权��）。Nginx是7层负载，ip_hash保持session。

---

### 10. 熔断降级？

#### 10.1 Hystrix

```java
@HystrixCommand(fallbackMethod = "fallback")
public String test() {
    return remote.call();
}

public String fallback() {
    return "降级";
}

// 公共降级类
@DefaultProperties(defaultFallback = "defaultFallback")
```

```properties
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=30000
hystrix.command.default.circuitBreaker.requestVolumeThreshold=20
hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds=500000
```

#### 10.2 Circuit Breaker

```
Circuit Breaker状态：
- Closed：正常
- Open：熔断，请求直接返回fallback
- HalfOpen：尝试请求，恢复
```

#### 10.3 Sentinel

```java
// 限流
@SentinelResource(value = "test", blockHandler = "blockHandler")
public String test() {
    return remote.call();
}

public String blockHandler(BlockException e) {
    return "限流";
}

// 熔断
@SentinelResource(value = "test", fallback = "fallback")
public String test() {
    return remote.call();
}

public String fallback() {
    return "降级";
}
```

#### 10.4 回答模板

> 熔断器是fail fast模式，三状态Open/Closed/HalfOpen。Hystrix用@HystrixCommand+CircuitBreaker实现。Sentinel是阿里开源更丰富的限流降级，支持热点、授权、系统规则。都是防雪崩。

---

### 11. 限流实现？

#### 11.1 限流算法

```plaintext
1.计数器（固定窗口）
时间段内计数，超过阈值丢弃
简单但有突刺

2.滑动窗口
把时间窗口分成小格
更平滑

3.令牌桶（Token Bucket）
令牌以恒定速度放入
有令牌才能请求
允许突发流量

4.漏桶（Leaky Bucket）
以固定速度漏出
平滑
```

#### 11.2 实现

```java
// Guava RateLimiter
RateLimiter limiter = RateLimiter.create(100); // 每秒100许可
limiter.acquire(); // 获取许可，无则阻塞
limiter.tryAcquire(10, 1, TimeUnit.SECONDS); // 尝试

// Bucket4j
Bucket bucket = Bucket4j.builder()
    .addLimit(create(1, Duration.ofSeconds(1)))
    .build();
bucket.tryConsume(1);
```

#### 11.3 Nginx限流

```nginx
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=100r/s;
location / {
    limit_req zone=mylimit burst=200 nodelay;
}

# 连接数限制
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_conn addr 100;
```

#### 11.4 回答模板

> 限流常用：计数器、滑动窗口、令牌桶、漏桶。令牌桶允许突发（Guava RateLimiter）。Sentinel支持系统/QPS/并发/冷启动限流。Nginx的limit_req_zone和limit_conn做7层限流。

---

### 12. 网关GateWay？

#### 12.1 Spring Cloud Gateway

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: route1
          uri: lb://service-provider
          predicates:
            - Path=/api/**
          filters:
            - StripPrefix=1
            - AddRequestHeader=X-Custom, hello

# 动态路由
discovery:
  locator:
    enabled: true
    lower-case-service-id: true
```

#### 12.2 路由Predicate

```
Predicate：
- Path=/user/**
- Method=GET
- Header=X-Request-Id,\d+
- Query=username,admin
- Cookie=token,abc
- After=2020-01-01T00:00:00Z
- Before=...
- Between=...
```

#### 12.3 Filter

```
Filter：
-	AddRequestHeader=X-Request-Time,${dates.now()}
-	StripPrefix=1
-	PrefixPath=/api
-	Redirect=302,/login
-	RequestRateLimiter=... (限流)
-	Retry=3 (重试)
-	GatewayFilter (自定义)
```

#### 12.4 回答模板

> Gateway用Route路由、Predicate匹配、Filter处理。Predicate有Path/Method/Header/Query等路由规则。Filter做StripPrefix/Redirect/RequestRateLimiter限流。Gateway是Reactor响应式，非阻塞���

---

### 13. 配置中心？

#### 13.1 Spring Cloud Config

```properties
# bootstrap.yml
spring:
  cloud:
    config:
      server-addr: http://config-server:8888
      uri: ${CONFIG_SERVER:}
      name: ${spring.application.name}
      profile: ${SPRING_PROFILES_ACTIVE:dev}
```

```java
// 动态刷新
@RefreshScope
@Value("${user.name}")
String name;

// POST /actuator/refresh
// POST /bus/refresh (广播刷新)
```

#### 13.2 Nacos配置

```properties
spring:
  cloud:
    nacos:
      config:
        server-addr: ${NACOS_SERVER:}
        file-extension: yaml
        refresh-enabled: true
        group: DEFAULT_GROUP
        namespace: ${NACOS_NAMESPACE:}
```

```java
// 热生效
@RefreshScope
@Value("${user.name}")
String name;
```

#### 13.3 Apollo

```java
// 接入Apollo
@Value("${somekey:#{null}}")
private String someKey;

// API
Config app = appHolder.getApp();
ConfigFile configFile = app.getConfigFile(fileName);

// 监听
app.addListener(configFile, listener);
```

#### 13.4 回答模板

> 配置中心有Config（Spring）、Nacos（阿里）、Apollo（携程）。Config GIT后端+Nacos支持KV/Group/Namespace，Apollo支持灰度/发布diff/监听。配置热更新@RefreshScope+POST /actuator/refresh。

---

### 14. 分布式链路追踪？

#### 14.1 Sleuth+Zipkin

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

```properties
spring.zipkin.service.name: ${spring.application.name}
spring.sleuth.sample.ratio: 1.0
```

#### 14.2 跟踪内容

```log
# 日志输出
[app] TRACE_ID SPAN_ID INFO - message

 TraceId=abc SpanId=xyz ParentId=123
```

#### 14.3 SkyWalking

```xml
<!-- SkyWalking agent -->
-javaagent:skywalking-agent.jar
```

```java
// 自动跟踪
@Trace @Tags({@Tag("error")})
public String method() {}
```

#### 14.4 回答模板

> 链路追踪用Sleuth（埋点自动）+Zipkin（收集/展示）。Sleuth埋点输出TraceId/SpanId/ParentId。SkyWalking是APM工具，自动跟踪+UI+告警。Pinpoint是韩国NAVER开源。

---

## 第三章 微服务架构篇（高频 ★★★★★）

### 15. 微服务是什么？

#### 15.1 微服务定义

```plaintext
微服务特征：
1. 小：一个服务只做一件事
2. 独：独立进程，独立数据
3. 松：松耦合
4. 自治：小团队负责全生命周期

优势：
- 独立部署
- 独立扩展
- 技术异构
- 容错

```

#### 15.2 vs 单体

```plaintext
单体：           微服务：
┌─────────┐   ┌──┐ ┌──┐ ┌──┐ ┌──┐
│ 代码    │   │S1│ │S2│ │S3│ │S4│
│ 一个    │   │  │ │  │ │  │
└─────────┘   └──┘ └──┘ └──┘ └──┘
  -         -  独立部署
  - 难扩展   - 随意扩展
  - 难上线   - 按需上线
```

#### 15.3 康威定律

```plaintext
康威定律（Conway's Law）：
Organizations which design systems are constrained
to produce designs which are copies of the
communication structures of these organizations.

系统设计受限于组织架构
→ 多团队 → 多服务
```

#### 15.4 回答模板

> 微服务是服务小、独立、松耦合，团队全生命周期负责。小团队（2-10人）+小服务+独立数据库。优点独立部署/扩展/技术栈/容错。问题是复杂度（服务多、网络、分布式事务、运维）需要基础设施。

---

### 16. 服务拆分？

#### 16.1 拆分策略

```plaintext
拆分的维度：
1. 业务维度（领域驱动Design）
   - 子域/聚合根/限界上下文

2. 组织架构（康威定律）
   - 前端→后端（BFF）
   - 跨职能团队

3. 功能维度
   - 粒度过细→网络成本高
   - 粒度过粗→耦合
```

#### 16.2 拆分方法

```
DDD领域驱动设计：
- 限界上下文（Bounded Context）
- 聚合根（Aggregate Root）
- 实体（Entity）
- 值对象（Value Object）
- 领域事件（Domain Event）

常用拆分：
- 水平扩展：加副本
- 垂直拆分：功能模块
- 数据拆分：分库分表
```

#### 16.3 回答模板

> 拆分用DDD领域驱动设计，按业务边界限界上下文。拆分粒度小团队可控，一般几十到几百行代码。服务拆分要考虑团队、数据库、事务边界、网络成本、运维成本。服务不拆分单体。

---

### 17. 服务通信方式？

#### 17.1 RPC vs REST

```
RPC（Remote Procedure Call）：
- 模式调用，如同本地方法
- gRPC（HTTP/2+Protobuf）
- Dubbo（注解+协议）

REST（Representational State Transfer）：
- 资源导向
- HTTP动词+URI
- JSON/XML格式
```

```java
// gRPC
service UserService {
    rpc GetUser(GetUserRequest) returns (User);
    rpc ListUsers(ListUsersRequest) returns (stream User);
}

message User {
    int64 id = 1;
    string name = 2;
}
```

#### 17.2 同步 vs 异步

```
同步RPC：REST/gRPC
- 请求→响应
- 阻塞
- 实时一致性

异步消息：
- MQ（Kafka/RocketMQ）
- 事件驱动
- 最终一致

```

#### 17.3 回答模板

> 服务通信有REST（JSON/资源）和RPC（gRPC二进制高性能/Dubbo）两种方式。同步RPC实时响应，但有同步阻塞。异步用消息队列（Kafka/RocketMQ）解耦、削峰填谷、最终一致。混合使用根据场景。

---

### 18. API版本管理？

#### 18.1 版本策略

```java
// URI版本
GET /v1/users
GET /v2/users

// Query参数
GET /users?version=1

// Header
GET /users
X-API-Version: 1

// HATEOAS
self: { href: /v2/users/{id} }
```

#### 18.2 兼容性

```
向后兼容（Backward Compatible）：
- 不删除字段
- 不改字段类型
- 可加字段（optional）
- 不改已有枚举含义

版本策略：
- 版本号标识
- 过渡期两端并存
- 兼容版本不升级
```

#### 18.3 回答模板

> 服务版本用URI（/v1/或/v2/）最明确。query参数和header是隐藏版本。向后兼容：只加不减字段（可选），不支持降级或改类型。过渡期两版本并存。

---

### 19. 服务容错设计？

#### 19.1 故障模式

```
常见的故障处理：
- 快速失败Fail Fast
- Failover（重试其他实例）
- Failback（恢复后切回）
- 分片隔离
- 降级Graceful Degradation
```

#### 19.2 常用技术

```java
// 1.超时+重试+熔断
@CircuitBreaker
@Retry
@Timeouter

// 2.舱壁隔离
Bulkhead

// 3.线程池隔离
@ThreadPool
```

#### 19.3 设计模式

```plaintext
设计原则：
1. 防止级联失败（超时+舱壁）
2. 优雅降级（降级逻辑）
3. 快速恢复（健康检查）
4. 限流熔断（防止雪崩）
5. 幂等（重复请求）
```

#### 19.4 回答模板

> 服务容错：超时+重试+幂等、熔断、降级、舱壁隔离、限流防止雪崩。Sentinel/Hystrix做熔断，线程池隔离防止拖垮。返回端到端一致要考虑幂等。

---

### 20. 服务安全认证？

#### 20.1 OAuth2.0

```
OAuth2.0流程：
1.Client → Authorization Server: 请求授权
2.Authorization → User: 是否同意
3.User → Authorization: 同意
4.Authorization → Client: 授权码
5.Client → Token Endpoint: 授权码换Token
6.Token Endpoint → Client: AccessToken
7.Client → Resource: AccessToken访问
8.Resource → Client: 验证返回
```

#### 20.2 JWT

```java
// 生成JWT
JwtBuilder builder = Jwts.builder()
    .setId(uuid).setSubject(user)
    .claim("role","admin")
    .signWith(key)
    .setExpiration(date);

// 验证
Jwts.parser().setSigningKey(key).parseClaimsJws(token);
```

#### 20.3 网关认证

```
网关统一认证流程：
1.Client → Gateway: 请求
2.Gateway → Auth Server: 验证Token
3.Auth → Gateway: OK返回用户信息
4.Gateway → 服务: 转发+Header用户信息
```

```java
// Spring Security + JWT
JwtAuthenticationFilter extends OncePerRequestFilter {
    // 验证JWT
    // 设置SecurityContext
}
```

#### 20.4 回答模板

> 服务间认证用OAuth2 + JWT。网关统一验证Token，转化Header带用户信息下游。JWT分三部分用HS256/RS256签名。只验签不解密（公钥发布）。gateway做统一认证入口。

---

## 第四章 分布式存储篇（中高级 ★★★★）

### 21. 分布式缓存？

#### 21.1 Redis Cluster

```properties
# redis.conf
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-migration-barrier 1

# JedisCluster
JedisCluster jedisCluster = new JedisCluster(addrs, timeout, pool);
```

```java
// 槽（Slot）
# 16384个槽
# CRC16(key) % 16384 = slot
# 每个master负责一部分slot
```

#### 21.2 缓存策略

```plaintext
缓存模式：
1.Cache-Aside
  - Read: Cache→miss→DB
  - Write: DB→Cache

2.Read-Through
  - 缓存自动加载DB

3.Write-Through
  - 同步写DB和Cache

4.Write-Behind
  - 异步写，稍后flush
```

```java
// CacheAside示例
public User getUser(Long id) {
    User user = redisTemplate.opsForValue().get(id);
    if (user == null) {
        user = db.getById(id);
        if (user != null) {
            redisTemplate.set(id, user);
        }
    }
    return user;
}
```

#### 21.3 回答模板

> Redis分布式缓存cluster：数据分16384个槽，每节点部分。主从同步，自动failover。缓存模式：旁路CacheAside最常用，读miss填充、写直写DB/异步写Cache。缓存穿透BloomFilter、缓存击穿互斥锁、缓存雪崩TTL+随机。

---

### 22. 分布式session？

#### 22.1 保存session

```java
// 1.Redis保存Session
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>

// 2.Spring Boot Config
spring.session.store-type=redis
spring.session.redis.namespace=app:session
```

#### 22.2 Token

```java
// JWT保存sessionless
// 结论
1.Token→Redis无效，因为每次请求重算
2.只用于验证用户3.用Token
```

#### 22.3 回答模板

> Session统一存Redis集群解决多节点共享。无状态用JWT Token（带签名无状态），Token存用户ID和角色，验签不解密只验证有效。JWT过期时间不宜太长，配合RefreshToken。

---

### 23. 分库分表？

#### 23.1 分片策略

```
Sharding Strategy：
- Range：时间范围
  按月份/地区分

- Hash取模：哈希后%N
  取模方式：hash(id)%N

- 映射表：Sharding Catalog
  路由表直接查

- 一致性哈希：consistent hashing
  动态增减节点
```

```java
// Shard-key选择
// 选择唯一且常用查询字段
// user_id: order.user_id
// time_range: create_time
```

#### 23.2 中间件

```
ShardingSphere简介：
1.ShardingSphere-JDBC
  Java Jar，Spring配置
  支持：Read/Write + Sharding

2.ShardingSphere-Proxy
  独立进程，MySQL/PostgreSQL协议
```

```java
// ShardingSphere-JDBC配置
@Bean
public DataSource dataSource() throws SQLException {
    HikariDataSource ds = new HikariDataSource();
    // 配置数据源
    ShardingSphereDataSourceFactory.createDataSource(
        createMap(ds), createRules(), props);
}
```

#### 23.3 回答模板

> 分库分表解决大数据量，ShardingKey选择高频查询字段。Range适合时间序列，Hash取模均衡。但ID不能自增、跨分片查询需聚合。ShardingSphere是分布式数据库中间件。

---

### 24. 分布式锁？

#### 24.1 Redis分布式锁

```java
// SETNX
Boolean lock = redisTemplate.opsForValue()
    .setIfAbsent(lockKey, "1", Duration.ofSeconds(30));

// Lua脚本解锁
String script = "if redis.call('get', KEYS[1]) == ARGV[1]" +
             " then return redis.call('del', KEYS[1]) else return 0 end";

// watchdog续期
RedissonClient.lock = redisson.getLock();
lock.lock(); // 自动续期
lock.tryLock(3, 10, TimeUnit.SECONDS);
```

#### 24.2 ZK分布式锁

```java
// 临时顺序节点
/path/lock
  /lock/0000000001 (最小获取锁)
/path/lock/0000000002
...

// Watch监听前一个
getChildren(path, new Watcher(){
    onChildChange(parent){}
});
```

#### 24.3 对比

| 特性 | Redis | ZK |
|------|------|-----|
| 性能 | 高 | 一般 |
| 可靠性 | 一般 | 高 |
| 复杂度 | 低 | 中 |
| Lua | 支持 | 不支持 |

#### 24.4 回答模板

> 分布式锁有Redis（SETNX+过期+Watchdog自动续期）和ZK（临时顺序+Watch）两种。Redis性能好，Redisson SDK封装完善。ZK可靠性高（有leader），选主。库存、限时优惠需分布式锁防超卖。

---

### 25. 消息队列？

#### 25.1 RabbitMQ

```java
// Exchange类型：
// Direct：完全匹配
// Fanout：广播
// Topic：*和#匹配
// Headers：匹配headers

// Queue
@RabbitListener(queues = "my.queue")
public void listen(String msg) {}

// 发送
rabbitTemplate.convertAndSend("exchange", "route.key", "msg");
```

#### 25.2 Kafka

```java
// Producer
producer.send(new ProducerRecord<>(topic, msg));

// Consumer
@KafkaListener(topics = "my-topic", groupId = "my-group")
public void listen(ConsumerRecord record) {}

// 顺序保证partition
// key进同一partition
// partition数不变
```

#### 25.3 RocketMQ

```java
// Producer
producer.send(new Message("topic", "tag", msg));
producer.send(message, (result, e) -> {});

// Consumer
@RocketMQListener(topic = "topic", consumerGroup = "group")
public void listen(Message msg) {}
```

#### 25.4 MQ选型对比

| MQ | Rabbit | Kafka | Rocket |
|---|-------|-------|-------|
| 延迟 | 低 | 中 | 低 |
| 吞吐 | 高 | 极高 | 高 |
| 有序 | 支持 | Partition | 支持 |
| 事务 | 支持 | 支持 | 事务 |
| 生态 | 大 | 大 | 阿里 |
| 适用 | 普通 | 日志/大数据 | 业务 |

#### 25.5 回答模板

> MQ选型：Rabbit（低延迟，丰富Exchange，通用业务）、Kafka（高吞吐，大数据日志）、Rocket（阿里业务，事务消息）。削峰填谷、异步、解耦、事务消息都用。Kafka日志收集性能最优。

---

## 第五章 架构设计篇（中高级 ★★★★）

### 26. 高可用架构设计？

#### 26.1 高可用多层级

```plaintext
高可用架构：
1. DNS：高可智能DNS，智能解析
2. LVS/KeepAlive：负载均衡主备
3. Nginx：多实例+respawn
4. 应用：多实例+滚动发布
5. DB：主从+半同步
6. Cache：Redis Cluster/Sentinel
7. MQ：集群
```

#### 26.2 异地多活

```plaintext
异地多活架构：
- 两地（上海+北京）
- 或三地（北京+上海+深圳）
- 流量分配DNS
- 数据同步
- 单元化
- 流量切换
```

#### 26.3 回答模板

> 高可用整体思路冗余：无单点，应用多实例+Nginx+LVS主备、数据库主从+半同步、缓存Cluster、MQ集群。异地多活：同AZ容灾+跨城容灾+DNS切换。设计：分层冗余+降级预案+监控告警。

---

### 27. 压测与性能优化？

#### 27.1 压测工具

```java
// JMeter
// Gatling
// Locust
// wrk

// ab - Apache Benchmark
ab -n 1000 -c 100 http://url

// wrk
wrk -t4 -c100 -d30s http://url
```

#### 27.2 性能指标

```plaintext
性能指标：
- QPS (每秒请求)
- TPS (每秒事务)
- RT (响应时间) P50/P95/P99
- PV (页面访问量)
- UV (独立访客)
- 并发用户

计算公式：
QPS = UV / RT (经验)
RT = 并发用户 / QPS
```

#### 27.3 优化方向

```java
// 优化：
// 1.连接池
// 2.缓存
// 3.异步化
// 4.压缩
// 5.减少请求
// 6.数据库优化
// 7.索引
// 8.预热
```

#### 27.4 回答模板

> 性能用JMeter/Gatling/Locust压测，关注P99 RT/QPS/并发。优化方向：缓存（Redis）、池化（DB/Http）、异步（消息）、压缩（GZIP）、合并（CSS Sprite）、预热。上线前压测找出瓶颈。

---

### 28. 分布式架构安全？

#### 28.1 常见攻击

```
分布式系统面临的攻击：
1.DDoS：流量攻击
2.SQL注入
3.XSS跨站脚本
4.CSRF跨站请求伪造
5.中间人攻击MITM
6.HTTP劫持
7.撞库/暴破
```

#### 28.2 安全措施

```java
// 防护措施：
1. WAF Web Application Firewall
2. IDS/IPS 入侵检测/防御系统
3. HTTPS TLS/SSL证书（全站HTTPS）
4. 请求签名防篡改
5. 接口限流防刷
6. 敏感数据脱敏/加密
```

#### 28.3 回答模板

> 安全：DDoS用CDN+WAF+黑洞路由，SQL/XSS用预编译防止，输入过滤。接口限流+验证码防刷。全站HTTPS+证书+TLS1.3防中间人。签名防请求篡改。密钥分开存KMS。

---

### 29. 服务网格 Service Mesh？

#### 29.1 Istio

```plaintext
Istio定义：
Service Mesh是服务间通讯基础设施
流量管理层与业务解耦

Istio架构：
- Control Plane: Pilot, Citadel, Galley
- Data Plane: Envoy Sidecar

功能：
- 流量管理：Routing, Retry, Timeout
-安全：mTLS, 授权
-观测：Telemetry, Tracing
```

#### 29.2 流量路由

```yaml
# VirtualService
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

#### 29.3 回答模板

> Service Mesh把流量管理、安全、可观测性与业务代码分离。Istio是当前最流行的Mesh，用Envoy做Sidecar代理。Service Mesh的好处是无侵入、Sidecar统一流量控制、mTLS加密、细粒度路由。K8s+Istio是云原生标配。

---

### 30. Kubernetes？

#### 30.1 Pod vs Deployment

```yaml
# Pod最小单元
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx

# Deploymet管理Pod
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

#### 30.2 Service

```yaml
# Service定义
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP/NodePort/LoadBalancer
```

#### 30.3 StatefulSet

```yaml
# StatefulSet（状态应用）
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ReadWriteOnce]
      resources:
        requests:
          storage: 10Gi
```

#### 30.4 回答模板

> K8s核心资源：Pod（容器集合）、Deployment（管理无状态Pod）、Service（网络抽象）、StatefulSet（有状态应用）、ConfigMap/Secret（配置）。K8s是容器编排，K8s+微服务=云原生架构。

---

## 附录：面试追问

1. **为什么微服务不用ES用于搜索？**
   - ES单实例瓶颈？数据量大ES性能下降？但ES可以用，支持集群+分片

2. **分布式锁实现选型？**
   - 推荐Redis（Redisson）或ZK。对性能要求高选Redis，选高可用选ZK

3. **数据一致性怎么办？**
   - 强一致用Seata AT/TCC，最终一致用MQ异步+Saga编排。根据业务选

4. **选Dubbo还是Spring Cloud?**
   - Dubbo性能高用gRPC，Spring Cloud生态全。性能敏感选Dubbo。

5. **微服务如何拆分到大小合适？**
   - 团队人数（2 pizza team）+代码行数（几百到几千）+业务边界清晰+满足单一职责。

---

## 参考资料

- 《分布式系统设计》Brian Foote
- 《微服务架构设计模式》Chris Richardson
- Spring Cloud官方文档
- Kubernetes官方文档
- Seata官方文档
- Etcd官方文档

---

> 整理by Claude Code | 分布式与微服务面试高频100问