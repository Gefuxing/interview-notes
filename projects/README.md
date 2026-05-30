# 项目场景题100问——真实业务场景深度指南

> 本文档面向项目开发学习者，从真实业务场景出发，包含高频面试场景题深度解析。
> 每个问题从「业务背景」→「问题分析」→「解决方案」→「代码实现」四个维度讲解。

---

## 第一章 数据处理场景（高频 ★★★★★）

### 1. 百万级数据Excel导出

#### 1.1 业务背景
系统需要将数据库中100万条记录导出为Excel供运营人员分析，内存只有200MB。

#### 1.2 问题分析
- 全量加载会OOM
- Excel文件过大无法打开
- HTTP请求超时
- 数据库查询慢

#### 1.3 解决方案
- 分页查询+流式写入(SXSSF)
- 拆分多个Excel文件
- 异步导出+邮件通知
- 使用CSV替代Excel

#### 1.4 代码实现
```java
// SXSSF流式导出（内存保留1000行）
SXSSFWorkbook workbook = new SXSSFWorkbook(1000);
Sheet sheet = workbook.createSheet();
int rowNum = 0;
for (int page = 0; ; page++) {
    List<Data> list = queryByPage(page, 5000);
    if (list.isEmpty()) break;
    for (Data d : list) {
        Row row = sheet.createRow(rowNum++);
        row.createCell(0).setCellValue(d.getId());
    }
    ((SXSSFSheet) sheet).flushRows(500); // 刷新到磁盘
}
workbook.write(out);
workbook.dispose();
```

---

### 2. 千万级数据分页查询优化

#### 2.1 业务背景
分页查询第100页时响应超过10秒，用户体验差。

#### 2.2 问题分析
- OFFSET越大，性能呈指数下降
- SELECT * 导致回表
- 缺少合适的复合索引

#### 2.3 解决方案
- 游标分页（基于ID）
- 索引覆盖
- 限制返回字段

#### 2.4 代码实现
```java
// 游标分页（性能最优）
List<User> searchByCursor(Long lastId, int size) {
    return mapper.selectByCursor(lastId, size);
}
// SQL: WHERE id > #{lastId} ORDER BY id LIMIT #{size}
```

---

### 3. 亿级用户共同好友查询

#### 3.1 业务背景
查询用户A和B的共同好友，平台活跃用户1亿。

#### 3.2 问题分析
- 数据量大，内存无法加载
- Set求交效率低
- 需要归一化算法

#### 3.3 解决方案
- Redis Bitmap位图存储
- BITOP AND求交集
- 分段处理大用户

#### 3.4 代码实现
```java
// Redis Bitmap方案
public Set<Long> getCommonFriends(long uid1, long uid2) {
    String key1 = "fr:" + uid1;
    String key2 = "fr:" + uid2;
    String resultKey = "result:" + uid1 + ":" + uid2;
    jedis.bitop(AND, resultKey, key1, key2);
    // 解析bitmap获取结果
    Set<Long> common = new HashSet<>();
    // 遍历解析...
    return common;
}
```

---

### 4. 实时积分排行榜

#### 4.1 业务背景
游戏需要实时榜单，展示前100名玩家。

#### 4.2 问题分析
- 更新频率高（每秒数千次）
- 查询延迟要求毫秒级
- 并发量高

#### 4.3 解决方案
- Redis ZSet有序集合
- 增量更新
- 本地缓存热点数据

#### 4.4 代码实现
```java
// ZSet实现
public void updateScore(String playerId, double score) {
    jedis.zadd("leaderboard", score, playerId);
}
public Set<String> getTopN(int n) {
    return jedis.zrevrange("leaderboard", 0, n - 1);
}
```

---

### 5. 连续签到统计

#### 5.1 业务背景
记录用户连续签到天数，断签后重新计算。

#### 5.2 问题分析
- 连续性判断逻辑复杂
- 跨月数据处理
- 防刷验证

#### 5.3 解决方案
- Redis Bitmap按天存储
- 游标遍历计算连续

#### 5.4 代码实现
```java
public void signIn(long userId) {
    String key = "sign:" + userId + ":" + getYearWeek();
    jedis.setBit(key, LocalDate.now().getDayOfYear(), true);
}
public int getContinousDays(long userId) {
    // 从今天往前遍历，连续断开则停止
}
```

---

### 6. 缓存击穿（热点Key）

#### 6.1 业务背景
热点Key失效瞬间，大量请求直达DB。

#### 6.2 问题分析
- 单一Key高并发
- 缓存空窗期
- 雪崩效应

#### 6.3 解决方案
- 互斥锁（SETNX）
- 逻辑过期
- 永不过期+后台异步更新

#### 6.4 代码实现
```java
// 互斥锁方案
String lockKey = "lock:" + key;
Boolean acquired = redis.setIfAbsent(lockKey, "1", 10, TimeUnit.SECONDS);
if (acquired) {
    try {
        value = redis.get(key);
        if (value == null) {
            value = db.query(key);
            redis.set(key, value, 30, TimeUnit.MINUTES);
        }
    } finally {
        redis.delete(lockKey);
    }
} else {
    Thread.sleep(100); return getValue(key);
}
```

---

### 7. 缓存雪崩

#### 7.1 业务背景
大量Key在同一时间过期。

#### 7.2 问题分析
- 同时失效导致DB压力骤增
- 缓存服务不可用

#### 7.3 解决方案
- 随机TTL
- 多级缓存
- 永不失效+主动刷新

#### 7.4 代码实现
```java
// 随机过期时间 10~30分钟
int expire = 10 + new Random().nextInt(20);
redis.setex(key, expire, value);
```

---

### 8. 缓存穿透

#### 8.1 业务背景
查询一个不存在的Key，每次都打DB。

#### 8.2 问题分析
- 恶意请求
- 查询不存在的数据

#### 8.3 解决方案
- 空值缓存
- BloomFilter过滤

#### 8.4 代码实现
```java
// 空值缓存+布隆过滤器
if (!bloomFilter.mightContain(key)) {
    return null; // 快速拦截
}
value = redis.get(key);
if (value == null) {
    value = db.query(key);
    redis.set(isNull?"NULL":value, 5, TimeUnit.MINUTES);
}
```

---

### 9. 库存超卖

#### 9.1 业务背景
秒杀活动100件，上万人抢购。

#### 9.2 问题分析
- 并发下库存扣减不准确
- 幻读问题

#### 9.3 解决方案
- 数据库乐观锁
- Redis原子扣减
- 消息队列串行化

#### 9.4 代码实现
```java
// Redis原子扣减
public boolean seckill(String productId) {
    Long stock = jedis.decr("stock:" + productId);
    if (stock < 0) {
        jedis.incr("stock:" + productId);
        return false;
    }
    return true;
}
```

---

### 10. 幂等性接口

#### 10.1 业务背景
用户重复提交提现申请。

#### 10.2 问题分析
- 网络抖动重试
- 恶意重复提交
- 回调多次

#### 10.3 解决方案
- 唯一标识（Token/UUID）
- 去重表（Redis/DB）
- 状态机校验

#### 10.4 代码实现
```java
public boolean process(String bizId) {
    if (redis.hasKey("processed:" + bizId)) {
        return false;
    }
    redis.setex("processed:" + bizId, 24 * 3600, "1");
    return doProcess(bizId);
}
```

---

## 第二章 高并发场景（高频 ★★★★★）

### 11. 秒杀系统设计

#### 11.1 业务背景
双11零点大促，瞬时并发100万QPS。

#### 11.2 架构设计
- 页面静态化CDN
- 验证码拦截
- 限流（Redis/Nginx）
- 削峰（MQ）
- 库存预热
- 熔断降级

#### 11.3 代码实现
```java
@PostMapping("/seckill")
public Result seckill(SeckillRequest req) {
    // 1. 验证码校验
    if (!verifyCaptcha(req.getCode())) {
        return Result.error("验证码错误");
    }
    // 2. IP限流
    if (rateLimiter.isBlocked(getIp())) {
        return Result.error("请稍后重试");
    }
    // 3. 库存校验
    if (decrementStock(req.getProductId())) {
        // 4. 写入队列
        mq.send(buildMsg(req));
        return Result.ok("排队中");
    }
    return Result.error("已售罄");
}
```

---

### 12. 抢单并发控制

#### 12.1 业务背景
1个Offer，100人抢，保证唯一性。

#### 12.2 解决方案
- SETNX原子操作
- Lua脚本保证原子性

#### 12.3 代码实现
```java
// Lua脚本原子操作
String script = "if redis.call('setnx', KEYS[1], ARGV[1]) == 1 then " +
              "  redis.call('expire', KEYS[1], 60) " +
              "  return 1 else return 0 end";
Long result = jedis.eval(script, 1, key, uuid);
```

---

### 13. 订单30分钟未支付关闭

#### 13.1 业务背景
订单创建30分钟后自动关闭库存回滚。

#### 13.2 解决方案
- 定时任务
- 延迟消息队列

#### 13.3 代码实现
```java
// 延迟消息
@RabbitListener(queues = "order.cancel")
public void cancelOrder(String orderId) {
    Order order = getOrder(orderId);
    if (order.isUnpaid()) {
        rollbackStock(order.getItems());
        closeOrder(orderId);
    }
}
// 发送延迟消息
mqp.convertAndSend("order.cancel", orderId, m -> m.setExpiration("1800000"));
```

---

### 14. 热点数据探测

#### 14.1 业务背景
实时发现热点商品，用于自动扩容。

#### 14.2 解决方案
- 滑动窗口统计
- 聚类算法

#### 14.3 代码实现
```java
public Map<String, Long> detectHotspots(List<String> keys) {
    for (String key : keys) {
        if (redis.zcard("hot:" + key) > THRESHOLD) {
            return hotspot(key);
        }
    }
}
```

---

### 15. 消息积压处理

#### 15.1 业务背景
消费者宕机，10万条消息积压。

#### 15.2 解决方案
- 临时扩容消费者
- 消息批量处理
- 跳过重试3次失败的消息

#### 15.3 代码实现
```java
@RabbitListener(concurrency = "20")
public void batchConsume(List<Message> msgs) {
    for (Message msg : msgs) {
        try { process(msg); ack(msg); }
        catch (Exception e) {
            if (retries(msg) >= 3) ack(msg); // 跳过
            else nack(msg, true);
        }
    }
}
```

---

### 16. 重复消息处理

#### 16.1 业务背景
ACK失败导致消息重复投递。

#### 16.2 解决方案
- 消息去重表
- 业务幂等

#### 16.3 代码实现
```java
if (redis.hasKey("msg:" + msgId)) {
    channel.basicAck(deliveryTag, false);
    return;
}
process(msg);
redis.setex("msg:" + msgId, 24*3600, "1");
```

---

### 17. 接口限流

#### 17.1 业务背景
防止恶意请求保护后端服务。

#### 17.2 解决方案
- 计数器限流
- 滑动窗口
- 令牌桶

#### 17.3 代码实现
```java
// 令牌桶
public boolean tryAcquire(String key) {
    Long count = redis.incr("rate:" + key);
    redis.expire(key, 60);
    return count <= MAX_PER_MINUTE;
}
```

---

### 18. 服务熔断降级

#### 18.1 业务背景
依赖服务超时不可用。

#### 18.2 解决方案
- CircuitBreaker
- 降级方法

#### 18.3 代码实现
```java
@CircuitBreaker(name = "remote", fallbackMethod = "fallback")
public String call() { return remoteCall(); }
public String fallback(Exception e) { return "降级结果"; }
```

---

### 19. 分布式锁

#### 19.1 业务背景
同一订单只能被一个worker处理。

#### 19.2 解决方案
- Redis SETNX
- Lua脚本保证原子解锁

#### 19.3 代码实现
```java
Boolean lock(String key, String val) {
    return redis.setIfAbsent(key, val, 30, TimeUnit.SECONDS);
}
void unlock(String key, String val) {
    // Lua保证只删除自己的锁
    jedis.eval("if redis.call('get',KEYS[1])==ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end",
              1, key, val);
}
```

---

### 20. 热点Key发现与保护

#### 20.1 业务背景
自动发现并保护热点数据。

#### 20.2 解决方案
- 频率统��算��
- 多级缓存保护

#### 20.3 代码实现
```java
public void hit(String key) {
    redis.incr("hit:" + key);
    if (hot(key)) {
        // 提升到本地缓存
        localCache.put(key, redis.get(key));
    }
}
```

---

## 第三章 数据一致性场景（高频 ★★★★★）

### 21. 跨库数据同步

#### 21.1 业务背景
订单库需要同步到数据仓库。

#### 21.2 解决方案
- Canal监听Binlog
- Debezium CDC
- 定时任务拉取

#### 21.3 代码实现
```java
@CanalListener(destination = "orders")
public void onEvent(Message msg) {
    Entry entry = msg.getEntries()[0];
    if (entry.getEntryType() == RO WDATA) {
        RowChange rowChange = RowChange.parseB(entry);
        handle(rowChange);
    }
}
```

---

### 22. 缓存与数据库一致性

#### 22.1 业务背景
修改DB后需要更新Cache。

#### 22.2 解决方案
- Cache-Aside模式
- 双删策略
- 延迟双删

#### 22.3 代码实现
```java
public void update(User user) {
    db.update(user);
    redis.delete("user:" + user.getId());
    // 延迟双删
    redis.async(() -> redis.delete("user:" + user.getId()));
}
```

---

### 23. 异构数据源查询

#### 23.1 业务背景
查询MySQL+ES合并结果。

#### 23.3 代码实现
```java
// 并行查询+合并
Future<List<A>> es = executor.submit(() -> es.search(q));
Future<List<A>> mysql = executor.submit(() -> db.query(q));
merge(es.get(), mysql.get());
```

---

### 24. 敏感数据脱敏

#### 24.1 业务背景
手机号、身份证等展示脱敏。

#### 24.2 代码实现
```java
public String maskPhone(String p) {
    return p == null ? null : p.substring(0,3)+"****"+p.substring(7);
}
public String maskIdCard(String id) {
    return id.substring(0,3)+"**********"+id.substring(id.length()-4);
}
```

---

### 25. 分布式事务最终一致性

#### 25.1 业务背景
创建订单->扣库存->记录日志需要事务。

#### 25.2 解决方案
- TCC模式
- 事务消息
- 本地消息表

#### 25.3 代码实现
```java
// 事务消息方案
@Transactional
public void createOrder(Order order) {
    dao.insert(order);
    mq.sendInTransaction(() -> {
        inventoryDao.deduct(order.getItems());
        logDao.record(order.getId());
    });
}
```

---

### 26. 对账务处理

#### 26.1 业务背景
第三方支付需要对账。

#### 26.2 解决方案
- 定时对账任务
- 差异文件生成

#### 26.3 代码实现
```java
@Scheduled(cron = "0 0 2 * * ?")
public void reconcile() {
    List<PayRecord> db = dao.listToday();
    List<PayRecord> remote = remoteApi.listToday();
    Map diff = findDiff(db, remote);
    saveDiff(diff);
}
```

---

### 27. ID号段连续性

#### 27.1 业务背景
订单号需要连续且唯一。

#### 27.2 解决方案
- 数据库号段
- Redis预生成
- Snowflake算法

#### 27.3 代码实现
```java
// 号段模式
public List<Long> getIdSection(int size) {
    Long max = redis.get("id:max");
    if (max == null) {
        max = dao.getMaxId();
        redis.set("id:max", max + size);
    }
    return LongStream.rangeClosed(max + 1, max + size).boxed().collect(Collectors.toList());
}
```

---

## 第四章 性能优化场景（中高级 ★★★★）

### 28. SQL慢查询优化

#### 28.1 业务背景
某API响应超过5秒。

#### 28.2 解决方案
- EXPLAIN分析
- 添加索引
- 优化SQL

#### 28.3 代码实现
```sql
-- 添加复合索引
ALTER TABLE orders ADD INDEX idx_user_status (user_id, status);
-- 避免全表扫描
SELECT id, name FROM orders WHERE user_id = ? AND status = ?
```

---

### 29. 大数据量Join

#### 29.1 业务背景
两张百万级表Join性能差。

#### 29.2 解决方案
- 预先计算
- 缓存中间结果
- 拆分为两次查询

#### 29.3 代码实现
```java
List<User> users = db.query("SELECT * FROM users WHERE ...");
Map deptMap = db.queryAsMap("SELECT id, name FROM dept");
users.forEach(u -> u.setDeptName(deptMap.get(u.getDeptId())));
```

---

### 30. 接口响应时间优化

#### 30.1 业务背景
商品详情页RT超过1秒。

#### 30.2 解决方案
- 不查SELECT *
- 减少JOIN
- 异步加载非核心信息

#### 30.3 代码实现
```java
// Promise并行查询 CompletableFuture
CompletableFuture<Product> p = CompletableFuture.supplyAsync(() -> getProduct(id));
CompletableFuture<List<Comment>> c = CompletableFuture.supplyAsync(() -> getComments(id));
CompletableFuture<List<Similar>> s = CompletableFuture.supplyAsync(() -> getSimilar(id));
Product result = p.get();
result.setComments(c.join());
result.setSimilarProducts(s.join());
```

---

### 31. 分页性能优化

#### 31.1 业务背景
百万级数据分页很慢。

#### 31.2 解决方案
- 游标分页
- 索引覆盖

#### 31.3 代码实现
```java
// 游标分页 比 OFFSET快100倍
List items = dao.select(key, lastId, 20);
// SQL: WHERE key=? AND id>? ORDER BY id LIMIT 20
```

---

### 32. 大表DDL在线变更

#### 32.1 业务��景
大表添加索引不能锁表。

#### 32.2 解决方案
- 使用Percona Toolkit
- Online DDL
- 复制临时表

#### 32.3 代码实现
```bash
# MySQL 8.0 Online DDL
ALTER TABLE large_table ADD INDEX idx_name(name), ALGORITHM=INPLACE, LOCK=NONE;
```

---

### 33. JVM调优

#### 33.1 业务背景
Full GC频繁。

#### 33.2 解决方案
- G1 GC
- 合理堆大小
- 排查内存泄漏

#### 33.3 代码实现
```bash
# JVM参数
-Xms4g -Xmx4g -XX:+UseG1GC -XX:MaxGCPauseMillis=200
```

---

### 34. 连接池优化

#### 34.1 业务背景
高并发时连接耗尽。

#### 34.2 解决方案
- 合理配置连接数
- 检测连接泄漏

#### 34.3 代码实现
```yaml
# Druid配置
spring.datasource.druid.max-active: 20
validation-query: SELECT 1
```

---

### 35. CPU高排查

#### 35.1 业务背景
服务CPU 100%。

#### 35.2 解决方案
- top定位进程
- jstack排查线程

#### 35.3 代码实现
```bash
# 排查步骤
top -H -p pid
jstack pid > j.stack
# 找到CPU高线程
```

---

### 36. 数据库连接池打满

#### 36.1 业务背景
Too many connections。

#### 36.2 解决方案
- 增大连接池
- 减少慢SQL
- 加超时

#### 36.3 代码实现
```java
HikariConfig config = new HikariConfig();
config.setMaximumPoolSize(50);
config.setMinimumIdle(10);
config.setConnectionTimeout(30000);
```

---

## 第五章 架构设计场景（中高级 ★★★★）

### 37. 微服务拆分

#### 37.1 业务背景
单体应用200万行代码，需要拆分。

#### 37.2 拆分原则
- 业务边界
- 团队边界
- 独立演进

#### 37.3 代码实现
```java
// 按业务拆分服务
// order-service 订单服务
// product-service 商品服务
// user-service 用户服务
```

---

### 38. 注册发现

#### 38.1 业务背景
服务需要动态感知上下线。

#### 38.2 解决方案
- Nacos/Consul/Eureka

#### 38.3 代码实现
```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: nacos:8848
```

---

### 39. 配置中心

#### 39.1 业务背景
多环境配置管理困难。

#### 39.2 解决方案
- Nacos/Apollo

#### 39.3 代码实现
```yaml
spring:
  cloud:
    nacos:
      config:
        server-addr: nacos:8848
        group: DEFAULT_GROUP
        refresh-enabled: true
```

---

### 40. API网关设计

#### 40.1 业务背景
统一入口需要鉴权、限流。

#### 40.2 解决方案
- Spring Cloud Gateway
- Kong/Nginx

#### 40.3 代码实现
```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("user", r -> r.path("/user/**")
            .filters(f -> f.stripPrefix(1).addRequestHeader("X-Gateway", "true"))
            .uri("lb://user-service"))
        .build();
}
```

---

### 41. 分布式ID生成

#### 41.1 业务背景
分库分表需要唯一ID。

#### 41.2 解决方案
- Snowflake
- UUID
- 数据库号段

#### 41.3 代码实现
```java
// Snowflake
long id = ((timestamp - EPOCH) << 22) |
         (machineId << 12) |
         sequence.incrementAndGet();
```

---

### 42. 分库分表设计

#### 42.1 业务背景
单库无法支撑数据量。

#### 42.2 解决方案
- 垂直拆分
- 水平拆分
- ShardingSphere/MyCat

#### 42.3 代码实现
```java
// ShardingSphere
ShardingRuleConfiguration rule = new ShardingRuleConfiguration();
rule.getTableRule().add(createTableRule());
rule.setDefaultDatabaseShardingStrategyModulo();
```

---

### 43. 服务调用超时

#### 43.1 业务背景
RPC调用超时设置不合理。

#### 43.2 解决方案
- 超时设置
- 重试策略
- 熔断降级

#### 43.3 代码实现
```java
@FeignClient(value = "service", timeout = 5000, retries = 2)
public interface ProductClient {
    @GetMapping("/product/{id}")
    Product getProduct(@PathVariable("id") Long id);
}
```

---

### 44. 灰度发布

#### 44.1 业务背景
新版本需要小流量验证。

#### 44.2 解决方案
- 权重流量切分
- UserId路由

#### 44.3 代码实现
```yaml
# Nginx权重
upstream backend {
    server v1 weight=90;
    server v2 weight=10;
}
```

---

### 45. 同城多活

#### 45.1 业务背景
需要容灾能力。

#### 45.2 解决方案
- 同城双活
- 跨机房同步

#### 45.3 代码实现
```java
// 双写+主从
@PostPersist
public void publish(Event event) {
    mq.send("samecity.event", event); // 跨机房
    mq.send("local.event", event); // 本机房
}
```

---

### 46. 数据库读写分离

#### 46.1 业务背景
读多写少需要分散压力。

#### 46.2 解决方案
- 主从复制
- 读写分离

#### 46.3 代码实现
```yaml
spring:
  shardingsphere:
    datasource:
      ds0: ds_master
      ds1: ds_slave
    rules:
      replication:
       mode: MASTER_SLAVE
       master-data-source-name: ds0
       slave-data-source-names:
          - ds1
```

---

### 47. 热点账户转账

#### 47.1 业务背景
大V账户并发转账安全。

#### 47.2 解决方案
- Redis分布式锁
- 乐观锁

#### 47.3 代码实现
```java
// Redis锁+乐观锁
redis.watch(balanceKey);
int balance = Integer.parseInt(redis.get(balanceKey));
if (balance < amount) throw new Exception("余额不足");
redis.multi();
redis.decrBy(balanceKey, amount);
redis.exec();
```

---

### 48. 限时优惠抢购

#### 48.1 业务背景
限定时间段的优惠活动。

#### 48.2 解决方案
- 动态配置秒杀时间
- 验证码拦截

#### 48.3 代码实现
```java
if (!inTimeRange(now(), config.getStartTime(), config.getEndTime())) {
    return error("活动未开始");
}
```

---

### 49. 数据归档

#### 49. 业务背景
冷热数据分离，降低成本。

#### 49.2 解决方案
- 按时间归档
- 迁移到OSS/HDFS

#### 49.3 代码实现
```java
@Scheduled(cron = "0 0 3 * * ?")
public void archive() {
    List old = dao.selectCreatedBefore(LocalDateTime.now().minusMonths(6));
    for (Data d : old) {
        oss.upload(d);
        dao.delete(d.getId());
    }
}
```

---

### 50. 敏感日志脱敏

#### 50.1 业务背景
日志不能记录敏感信息。

#### 50.2 解决方案
- 统一脱敏组件
- logback规则

#### 50.3 代码实现
```java
// 统一脱敏器
public class DesensitizeUtil {
    public static String phone(String s) {
        return s == null ? null : s.substring(0,3)+"****"+s.substring(7);
    }
}
```

---

## 第六章 业务场景综合（中高级 ★★★★）

### 51. 短链系统设计

#### 51.1 业务背景
将长URL转换为短URL。

#### 51.2 解决方案
- ID自增+62进治
- 哈希+碰撞检测

#### 51.3 代码实现
```java
public String genShortUrl(String longUrl) {
    long id = redis.incr("short:counter");
    String shortCode = Base62.encode(id); // 0-9a-zA-Z
    redis.setex("short:" + shortCode, 7*24*3600, longUrl);
    return "https://s.url/" + shortCode;
}
```

---

### 52. 延迟队列设计

#### 52.1 业务背景
订单超时自动取消。

#### 52.2 解决方案
- RabbitMQ TTL+DLX
- Redis过期键
- 定时任务扫描

#### 52.3 代码实现
```java
// RabbitMQ延迟队列
@RabbitListener(queues = "delay.order")
public void handleDelay(Order order) {
    if (!paid(order.getId())) {
        cancel(order.getId());
    }
}
// 发送时设置TTL
mqp.convertAndSend("delay.order", orderId, m ->
    m.setExpiration("1800000")); // 30分钟
```

---

### 53. 消息推送系统

#### 53.1 业务背景
APP推送+短信+邮件。

#### 53.2 解决方案
- 极光/个推
- 多通道重试

#### 53.3 代码实现
```java
public void push(User user, String content) {
    // APP
    jpush.push(user.getId(), content);
    // 短信备用
    if (user.getPhone() != null) {
        sms.send(user.getPhone(), content);
    }
}
```

---

### 54. 搜索建议

#### 54.1 业务背景
搜索框自动补全提示。

#### 54.2 解决方案
- Trie树
- 异步构建

#### 54.3 代码实现
```java
// Trie树前缀匹配
public List<String> suggest(String prefix) {
    return trie.startsWith(prefix, 10);
}
```

---

### 55. Feed流设计

#### 55.1 业务背景
朋友圈信息流。

#### 55.2 解决方案
- Pull模式（微博）
- Push模式（朋友圈）
- 混合模式

#### 55.3 代码实现
```java
// Push模式
public void post(User user, Feed feed) {
    db.save(feed);
    // 推送到粉丝
    for (Long followerId : user.getFollowers()) {
        redis.lpush("feed:" + followerId, feed.getId());
    }
}
```

---

### 56. 评论系统设计

#### 56.1 业务背景
文章评论+楼中楼。

#### 56.2 解决方案
- 嵌套评论
- 抽象评论树

#### 56.3 代码实现
```java
public List<Comment> getComments(Long articleId) {
    List<Comment> all = commentDao.selectByArticle(articleId);
    return buildTree(all);
}
```

---

### 57. 点赞系统设计

#### 57.1 业务背景
帖子点赞+实时显示。

#### 57.2 解决方案
- Redis Set存储点赞
- 异步持久化

#### 57.3 代码实现
```java
public boolean like(Long userId, Long postId) {
    Long result = redis.sadd("like:" + postId, userId);
    if (result == 1) {
        mq.send("like.event", postId); // 异步持久化
    }
    return result == 1;
}
```

---

### 58. 排行榜设计

#### 58.1业务背景
榜单+实时更新。

#### 58.2 解决方案
- Redis ZSet
- 定时持久化

#### 58.3 代码实现
```java
public List<Player> getRankList(int size) {
    Set ranked = redis.zrevrange("rank", 0, size - 1);
    return parse(ranked);
}
```

---

### 59. 抽奖系统设计

#### 59.1 业务背景
限时抽奖活动。

#### 59.2 解决方案
- 奖池预抽/实时抽
- 幂等验证
- 防刷

#### 59.3 代码实现
```java
public DrawResult draw(Long userId, String activityId) {
    // 检查是否已抽
    if (redis.exists("draw:" + activityId + ":" + userId)) {
        return new DrawResult(-1, "已抽过奖");
    }
    // 抽奖逻辑
    Prize prize = lottery.draw();
    redis.set("draw:" + activityId + ":" + userId, prize.getId());
    return prize;
}
```

---

### 60. 支付系统设计

#### 60.1 业务背景
第三方支付回调。

#### 60.2 解决方案
- 验签
- 幂等
- 异步通知

#### 60.3 代码实现
```java
@PostMapping("/notify")
public CallResult notify(NotifyRequest req) {
    // 1. 验签
    if (!verifySign(req)) {
        return CallResult.fail("签名失败");
    }
    // 2. 幂等
    if (processed(req.getTradeNo())) {
        return CallResult.ok("success");
    }
    // 3. 处理
    processPaySuccess(req.getTradeNo());
    return CallResult.ok("success");
}
```

---

### 61. 发货系统设计

#### 61.1 业务背景
订单自动发货。

#### 61.2 解决方案
- 异步发货
- 重试机制

#### 61.3 代码实现
```java
@RabbitListener(queues = "order.paid")
public void handlePaid(Order order) {
    // 1. 远程调用ERP
    ShipResult result = erp.ship(order.getItems());
    if (result.isSuccess()) {
        order.setStatus(SHIPPED);
    } else {
        retry(order); // 重试
    }
}
```

---

### 62. 取消订单退款

#### 62.1 业务背景
用户取消订单后退款。

#### 62.2 解决方案
- 原路退回
- 记录流水

#### 62.3 代码实现
```java
public void refund(Order order) {
    Payment payment = order.getPayment();
    // 原路退回
    if (payment.getType() == WECHAT) {
        wechatRefund.refund(payment.getTradeNo(), order.getAmount());
    }
    // 记录流水
    refundFlowDao.save(buildRefundFlow(order));
}
```

---

### 63. 积分系统设计

#### 63.1 业务背景
用户签到送积分，消费抵扣。

#### 63.2 代码实现
```java
public void signIn(Long userId) {
    // 签到积分
    redis.zadd("sign:score", userId, System.currentTimeMillis());
    // 连续签到加成
    int days = continuousDays(userId);
    int points = Math.min(days * 2, 10);
    addPoints(userId, points);
}
```

---

### 64. 黑名单系统

#### 64.1 业务背景
恶意用户拉黑。

#### 64.2 代码实现
```java
public boolean isBlack(Long userId) {
    return redis.sismember("black:set", userId);
}
```

---

### 65. IP限流防刷

#### 65.1 业务背景
单IP请求频率限制。

#### 65.2 代码实现
```java
public boolean isLimited(String ip) {
    String key = "req:ip:" + ip;
    Long count = redis.incr(key);
    if (count == 1) redis.expire(key, 60);
    return count > 100;
}
```

---

### 66. 验证码系统

#### 66.1 业务背景
短信验证码/图片验证码。

#### 66.2 代码实现
```java
public String sendCode(String phone) {
    String code = String.format("%06d", new Random().nextInt(999999));
    // 5分钟有效
    redis.setex("code:" + phone, 300, code);
    sms.send(phone, code);
    return code;
}
```

---

### 67. 单点登录

#### 67.1 业务背景
多系统SSO。

#### 67.2 代码实现
```java
public String login(String user, String pass) {
    if (validate(user, pass)) {
        String token = UUID.randomUUID().toString();
        redis.setex("token:" + token, 7*24*3600, user);
        return token;
    }
    return null;
}
```

---

### 68. 图片裁剪上传

#### 68.1 业务背景
头像裁剪上传。

#### 68.2 代码实现
```java
public String uploadAndCrop(MultipartFile file, int x, int y, int w, int h) {
    BufferedImage img = ImageIO.read(file.getInputStream());
    BufferedImage cropped = img.getSubimage(x, y, w, h);
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    ImageIO.write(cropped, "PNG", out);
    return oss.upload(out.toByteArray());
}
```

---

### 69. 文件批量上传

#### 69.1 业务背景
大文件分片上传。

#### 69.2 代码实现
```java
@PostMapping("/chunk")
public void uploadChunk(MultipartFile file, int chunk, int chunks, String fileId) {
    String path = "temp/" + fileId + "/" + chunk;
    oss.upload(path, file.getBytes());
    if (chunk == chunks - 1) {
        mergeChunks(fileId, chunks); // 合并
    }
}
```

---

### 70. 数据导出模板

#### 70.1 业务背景
批量导入数据模板。

#### 70.2 代码实现
```java
public Workbook exportTemplate() {
    Workbook wb = new Workbook();
    Sheet s = wb.createSheet("导入模板");
    Row r = s.createRow(0);
    r.createCell(0).setCellValue("名称(必填)");
    r.createCell(1).setCellValue("数量(必填)");
    return wb;
}
```

---

### 71. 定时任务幂等

#### 71.1 业务背景
防止定时任务重复执行。

#### 71.2 代码实现
```java
@Scheduled(cron = "0 0 2 * * ?")
public void dailyTask() {
    String lockKey = "task:daily:" + LocalDate.now();
    if (redis.setIfAbsent(lockKey, "1", 3*3600)) {
        doTask();
    }
}
```

---

### 72. 异步任务编排

#### 72.1 业务背景
多个异步任务完成后汇总结果。

#### 72.2 代码实现
```java
CompletableFuture<String> f1 = supplyAsync(() -> step1());
CompletableFuture<String> f2 = supplyAsync(() -> step2());
CompletableFuture<String> f3 = supplyAsync(() -> step3());
allOf(f1, f2, f3).thenApply(v -> merge(f1.join(), f2.join(), f3.join()));
```

---

### 73. 热点大Key

#### 73.1 业务背景
List长度几百万。

#### 73.2 解决方案
- 分批存储
- Hash结构

#### 70.3 代码实现
```java
public void add(String bigKey, String value) {
    long idx = redis.incr("idx:" + bigKey);
    redis.hset(bigKey, String.valueOf(idx), value);
}
```

---

### 74. 数据库死锁

#### 74.1 业务背景
并发更新导致死锁。

#### 74.2 解决方案
- 统一更新顺序
- 降低隔离级别

#### 74.3 代码实现
```java
// 按主键顺序更新，避免AB-BA死锁
update set balance=balance-? where id=? and balance>=?;
```

---

### 75. 数据漂移

#### 75.1 业务背景
数据与实际不符。

#### 75.2 解决方案
- 定时对账
- 修复任务

#### 76.3 代码实现
```java
@Scheduled(cron = "0 0 4 * * ?")
public void check() {
    List<Long> diff = findDiffBetweenDbAndCache();
    for (Long id : diff) {
        fix(id);
    }
}
```

---

### 76. 重复点击

#### 76. 业务背景
防止按钮重复点击。

#### 76.2 代码实现
```java
public void disableButton(Model model) {
    model.addAttribute("disabled", true);
}
@InitBinder
public void initBinder(WebDataBinder binder) {
    binder.setDisallowedFields("disabled");
}
```

---

### 77. 参数校验

#### 77. 业务背景
参数合法性校验。

#### 77.2 代码实现
```java
@Validated
public class Param {
    @NotBlank(message = "用户名不能为空")
    private String username;
    @Min(0) @Max(150)
    private Integer age;
}
```

---

### 78. 异常统一处理

#### 78.1 业务背景
统一返回值。

#### 78.2 代码实现
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    public Result handle(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }
}
```

---

### 79. 空值处理

#### 79.1 业务背景
Optional空值处理。

#### 79.2 代码实现
```java
return Optional.ofNullable(user)
    .map(User::getName)
    .orElse("匿名用户");
```

---

### 80. 线程池拒绝策略

#### 80.1 业务背景
高并发下的拒绝策略。

#### 80.2 代码实现
```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10, 50, 60, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(1000),
    r -> new Thread(r, "pool-" + r.hashCode()),
    (r, exe) -> {
        // 拒绝策略：降级返回
        r.run();
    }
);
```

---

### 81. 循环引用问题

#### 81.1 业务背景
JSON序列化死循环。

#### 81.2 代码实现
```java
// 添加忽略注解
@JsonIgnore
private User user;
```

---

### 82. 日期格式化

#### 82.1 业务背景
前后端日期格式统一。

#### 82.2 代码实现
```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
private Date createTime;
```

---

### 83. 防止SQL注入

#### 83. 业务背景
参数化查询。

#### 83. 验证实现
```java
// 使用Parameterized
PreparedStatement ps = conn.prepareStatement("SELECT * FROM user WHERE name = ?");
ps.setString(1, name);
```

---

### 84. XSS攻击防御

#### 84.1 业务背景
特殊字符过滤。

#### 84.2 代码实现
```java
public static String stripXSS(String value) {
    if (value == null) return null;
    return value.replaceAll("<", "&lt;").replaceAll(">", "&gt;");
}
```

---

### 85. CSRF防攻击

#### 85. 业务背景
表单重复提交。

#### 85.2 代码实现
```java
// Token校验
@PostMapping("/submit")
public Result submit(@RequestParam("_token") String token) {
    if (!verifyToken(token)) return error("请刷新页面");
    return ok();
}
```

---

### 86. 文件上传安全

#### 86.1 业务背景
防止上传恶意文件。

#### 86.2 代码实现
```java
// 白名单校验
String ext = FilenameUtils.getExtension(file.getOriginalFilename());
if (!ALLOWED_EXT.contains(ext.toLowerCase())) {
    return error("不允许的文件类型");
}
```

---

### 87. 敏感信息脱敏

#### 87. 业务背景
日志敏感信息。

#### 87.2 代码实现
```java
// 自定义Layout输出脱敏
layout.addPropertyConverter("pwd", p -> "******");
```

---

### 88. 接口幂等

#### 88. 业务背景
防重复提交。

#### 88.2 代码实现
```java
public boolean submit_once(String param) {
    return redis.opsForValue().setIfAbsent(param, "1", 300, TimeUnit.SECONDS);
}
```

---

### 89. 全局ID唯一性

#### 89. 业务背景
分布式唯一ID。

#### 89.2 代码实现
```java
// Snowflake + 业务前缀
public long genId(String bizCode) {
    return (System.currentTimeMillis() << 17) |
           (machineId << 12) |
           (sequence.getAndIncrement() & 0xFFF);
}
```

---

### 90. 数据库连接泄漏

#### 90. 业务背景
连接未关闭。

#### 90.2 代码实现
```java
// Spring管理自动Close
try (Connection conn = ds.getConnection()) {
    // 使用
} // 自动关闭
```

---

### 91. 事务超时

#### 91. 业务背景
长事务风险。

#### 91.2 代码实现
```java
@Transactional(timeout = 5) // 5秒超时
public void doSomething() { }
```

---

### 92. 并发更新丢失

#### 92. 业务背景
并发修改覆盖。

#### 92.2 代码实现
```java
// 乐观锁版本
update user set name=?, version=version+1 where id=? and version=?
```

---

### 93. 批量插入优化

#### 93. 业务背景
批量插入性能。

#### 93.2 代码实现
```java
// 批量插入
jdbc.batch("INSERT INTO user VALUES (?,?,?)", beans);
```

---

### 94. N+1问题

#### 94. 业务背景
循环查库。

#### 94.2 代码实现
```java
// 使用JOIN
@Query("JOIN FETCH")
List<User> findAllWithRoles();
```

---

### 95. MQ消息丢失

#### 95. 业务背景
发��失败。

#### 95.2 代码实现
```java
// 发送确认
rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
    if (!ack) {
        // 补偿发送
        resend(correlationData.getId());
    }
});
```

---

### 96. Redis数据倾斜

#### 96. 业务背景
某个Key访问特别高。

#### 96.2 代码实现
```java
// 分散到多个Key
for (int i = 0; i < 10; i++) {
    String key = "hot:" + HashUtil.hash(userId) % 10;
    redis.hset(key, userId, value);
}
```

---

### 97. 高并发抢红包

#### 97. 业务背景
群红包随机金额。

#### 97.2 代码实现
```java
// 随机分金额算法
public List<Integer> splitRedPacket(double amount, int count) {
    List<Integer> amounts = new ArrayList<>();
    double left = amount;
    for (int i = 0; i < count - 1; i++) {
        int range = (int) (left / (count - i) * 2);
        int money = new Random().nextInt(range);
        amounts.add(money);
        left -= money;
    }
    amounts.add((int) left);
    return amounts;
}
```

---

### 98. 活动防刷

#### 98. 业务背景
同接口同参数限制。

#### 98.2 代码实现
```java
@RateLimiter(key = "activity:" + userId + ":" + productId, limit = 1, time = 60)
public Result act() { return do(); }
```

---

### 99. 导出下载中文文件名

#### 99. 业务背景
下载文件名乱码。

#### 99.2 代码实现
```java
response.setHeader("Content-Disposition",
    "attachment;filename=" + URLEncoder.encode(filename, "UTF-8"));
```

---

### 100. 全局日志链路

#### 100. 业务背景
请求全链路追踪。

#### 100.2 代码实现
```java
@RequiredArgsConstructor
public class LogInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler) {
        String traceId = req.getHeader("X-Trace-Id");
        if (traceId == null) {
            traceId = UUID.randomUUID().toString();
        }
        MDC.put("traceId", traceId);
        resp.setHeader("X-Trace-Id", traceId);
        return true;
    }
}
```

---

> 整理by Claude Code | 项目场景题面试100问