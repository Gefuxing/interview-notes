# RabbitMQ 100问——消息队列核心技术深度指南

> 本文档面向消息队列学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 RabbitMQ基础（高频 ★★★★★）

### 1. 什么是RabbitMQ？

#### 1.1 定义

> RabbitMQ是实现了AMQP（Advanced Message Queuing Protocol）协议的消息中间件。

```plaintext
RabbitMQ核心特性：
- 消息中间件（Message Broker）
- MOM（Message Oriented Middleware）
- 支持多种协议（AMQP/STOMP/MQTT/HTTP）
- Erlang运行时
```

#### 1.2 回答模板

> RabbitMQ是采用Erlang编写、实现了AMQP协议的消息队列。用于应用间异步通信、解耦。

---

### 2. 消息队列的作用？

#### 2.1 核心价值

```
消息队列核心价值：
- 解耦（Dcoupling）- 保证系统独立性
- 削峰填谷（Peak Cutting）- 流量调控
- 异步处理（Async）- 提高系统吞吐
- 消息路由（Routing）- 灵活路由到消费者
```

#### 2.2 场景

```
典型场景：
- 订单处理
- 日志收集
- 任务分发
- 事件驱动
```

---

### 3. AMQP协议是什么？

#### 3.1 协议

```
AMQP（Advanced Message Queuing Protocol）：
- 消息传递，互联网标准
- 消息格式（标准的消息）
- 语义（Provider/Consumer）
- 安全性（SASL，TLS）
```

#### 3.2 回答模板

> AMQP是消息队列的互联网标准协议RabbitMQ支持的核心协议，定义了Producer/Broker/Consumer交互。

---

### 4. RabbitMQ架构组件？

#### 4.1 架构

```
RabbitMQ核心组件：
- Producer（生产者）
- Consumer（消费者）
- Broker（代理/服务器）
- Queue（队列）
- Exchange（交换机）
- Binding（绑定）
- Routing Key（路由键）
```

#### 4.2 图示

```plaintext
Producer → Exchange → Binding → Queue → Consumer
         (Router)  (Key)
```

---

### 5. 核心概念有哪些？

#### 5.1 名词

```
关键概念：
- Virtual Host (vhost)：隔离
- Connection：连接
- Channel：通道（多路复用）
- Exchange：交换机
- Queue：队列
- Binding：绑定
```

#### 5.2 回答模板

> RabbitMQ核心概念包括Producer/Exchange/Binding/Queue/Consumer。

---

### 6. RabbitMQ安装？

#### 6.1 Docker安装

```bash
docker run -d --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  -v rabbitmq_data:/var/lib/rabbitmq \
  rabbitmq:3-management
```

#### 6.2 RPM安装

```bash
# Erlang安装
rpm -Uvh https://packages.erlang-solutions.com/erlang/rhel/7/x86_64/esl-erlang_23.2.1-1~centos~7_amd64.rpm

# RabbitMQ安装
rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.8.7/rabbitmq-release-signing-key.asc
yum install rabbitmq-server-3.8.7
```

---

### 7. 管理界面使用？

#### 7.1 Web UI

```
默认端口：15672
默认账户：guest/guest（仅本地）
```

---

### 8. RabbitMQ Clients？

#### 8.1 各语言客户端

```
官方客户端：
- Java Client
- .NET Client
- Erlang Client
-，社区支持：Python/Ruby/Go/Node.js
```

#### 8.2 Java示例

```java
// 连接工厂
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
factory.setUsername("guest");
factory.setPassword("guest");

// 连接
Connection connection = factory.newConnection();

// 通道
Channel channel = connection.createChannel();
```

---

### 9. Exchange类型有哪些？

#### 9.1 四种Exchange

```java
// Direct Exchange - 精确匹配
channel.exchangeDeclare("direct_logs", "direct", true);

// Topic Exchange - 模式匹配
channel.exchangeDeclare("topic_logs", "topic", true);

// Fanout Exchange - 广播
channel.exchangeDeclare("fanout_logs", "fanout", true);

// Headers Exchange - Header属性匹配
channel.exchangeDeclare("headers_logs", "headers", true);
```

#### 9.2 回答模板

> RabbitMQ四种交换机：Direct精确匹配、Topic模式匹配（*#）、Fanout广播、Headers头匹配。

---

### 10. Queue队列属性？

#### 10.1 队列特性

```java
// 声明队列
channel.queueDeclare("queue_name", durable, exclusive, autoDelete, arguments);
// 参数：
// durable：持久化，重启后存在
// exclusive：仅一个consumer，关闭后删除
// autoDelete：无consumer自动删除
// arguments：扩展参数(x-message-ttl, x-max-length等)
```

---

## 第二章 消息发布订阅（高频 ★★★★★）

### 11. 生产者发布消息？

#### 11.1 Basic Publish

```java
// 简单发布
byte[] messageBodyBytes = "Hello, World!".getBytes();
channel.basicPublish(exchangeName, routingKey, null, messageBodyBytes);
```

#### 11.2 属性设置

```java
// 设置属性
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
    .deliveryMode(2)  // 持久化
    .contentType("application/json")
    .priority(1)
    .build();

channel.basicPublish(exchangeName, routingKey, properties, messageBodyBytes);
```

---

### 12. 消费者订阅消息？

#### 12.1 Basic Consume

```java
// 自动确认
channel.basicConsume(queueName, true, consumer);

// 手动确认
channel.basicConsume(queueName, false, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag,
                       Envelope envelope,
                       AMQP.BasicProperties properties,
                       byte[] body) {
        String message = new String(body);
        System.out.println("Received: " + message);

        // 确认消息
        channel.basicAck(envelope.getDeliveryTag(), false);
    }
});
```

---

### 13. 消息确认机制ACK？

#### 13.1 ACK

```java
// 手动确认
channel.basicAck(envelope.getDeliveryTag(), false);

// 拒绝消息（丢弃）
channel.basicReject(envelope.getDeliveryTag(), false);

// 拒绝消息（重新入队）
channel.basicReject(envelope.getDeliveryTag(), true);
```

#### 13.2 回答模板

> 消息ACK确认机制：自动ACK/手动ACK。手动确保消息处理完后确认，防丢失。

---

### 14. 消息属性Properties？

#### 14.1 BasicProperties

```
消息属性：
- deliveryMode (1非持久，2持久)
- contentType (MIME类型)
- priority (优先级 0-9)
- expiration (TTL)
- messageId
- timestamp
- headers (自定义头)
```

---

### 15. 消息持久化？

#### 15.1 实现

```java
// 队列持久化
channel.queueDeclare("myQueue", true, false, false, null);

// 消息持久化
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
    .deliveryMode(2)
    .build();
channel.basicPublish("", "myQueue", properties, message);
```

---

### 16. Prefetch预取？

#### 16.1 流量控制

```java
// 预取数量（公平分发）
channel.basicQos(1, true);// prefetchCount=1

// 公平VS轮询
// prefetch=1: 公平分发（谁空闲给谁）
// prefetch=0: 轮询（不等待）
```

---

### 17. Consumer Tag？

#### 17.1 消费者标识

```java
// 指定consumerTag
channel.basicConsume(queueName, consumerTag: "my_consumer", callback);

// 主动获取consumerTag
String consumerTag = channel.basicConsume(queueName, callback);
```

---

### 18. Message反序列化？

#### 18.1 JSON处理

```java
// Jackson ObjectMapper
ObjectMapper mapper = new ObjectMapper();
Message message = mapper.readValue(json, Message.class);
```

---

### 19. 过期时间TTL？

#### 19.1 设置TTL

```java
// 队列设置TTL
Map<String, Object> args = new HashMap<>();
args.put("x-message-ttl", 1800000); // 30分钟
channel.queueDeclare("myQueue", true, false, false, args);

// 消息设置TTL
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
    .expiration("1800000")
    .build();
```

---

### 20. Priority优先级队列？

#### 20.1 优先级

```java
// 优先队列
Map<String, Object> args = new HashMap<>();
args.put("x-max-priority", 10); // 最高优先级
channel.queueDeclare("priorityQueue", true, false, false, args);

// 消息加优先级
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
    .priority(5)
    .build();
```

---

## 第三章 Exchange路由（高频 ★★★★★）

### 21. Direct Exchange？

#### 21.1 精确匹配

```java
// 交换机绑定
channel.exchangeDeclare("direct_exchange", "direct", true);
channel.queueDeclare("error_queue", false, false, false, null);
channel.queueDeclare("info_queue", false, false, false, null);

// 绑定
channel.queueBind("error_queue", "direct_exchange", "error");
channel.queueBind("info_queue", "direct_exchange", "info");
```

---

### 22. Fanout Exchange？

#### 22.1 广播

```java
// Fanout（忽略routingkey，全部发送）
channel.exchangeDeclare("logs_fanout", "fanout", true);

// 所有绑定队列都能收到消息
```

---

### 23. Topic Exchange？

#### 23.1 模式匹配

```java
// Topic规则:
// * 匹配一个单词
// # 匹配0或多个单词

// 绑定
channel.queueBind("order_queue", "topic_exchange", "order.#");
channel.queueBind("create_queue", "topic_exchange", "order.created");
channel.queueBind("all_queue", "topic_exchange", "#");
```

#### 23.2 示例

```
Routing Key: order.created.save
匹配队列：
- order.# ✅ (order.anything)
- order.created ✅
- *.created ✅
- # anything
```

---

### 24. Headers Exchange？

#### 24.1 Header匹配

```java
// Headers交换机
channel.exchangeDeclare("headers_exchange", "headers", true);

// 绑定（x-match: all/any）
Map<String, Object> args = new HashMap<>();
args.put("x-match", "all"); // all全匹配，any任一匹配
args.put("type", "order");
channel.queueBind("queue", "headers_exchange", "", args);

// 发送
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .headers(headers)
    .build();
```

---

### 25. Alternate Exchanges备用交换？

#### 25.1 AE

```java
// 声明AE
Map<String, Object> args = new HashMap<>();
args.put("alternate-exchange", "ae_exchange");
channel.exchangeDeclare("main_exchange", "direct", true, args);
```

---

## 第四章 集群高可用（高频 ★★★★★）

### 26. RabbitMQ集群？

#### 26.1 集群配置

```bash
# 服务启动自动集群
rabbitmqctl set_cluster_name mycluster

# 停止应用
rabbitmqctl stop_app

# 重置节点
rabbitmqctl reset

# 加入集群
rabbitmqctl join_cluster <cluster_node> @<host>

# 启动应用
rabbitmqctl start_app
```

#### 26.2 查看集群状态

```bash
rabbitmqctl cluster_status
```

---

### 27. 镜像队列？

#### 27.1 Mirror Queue

```java
// 策略参数
rabbitmqctl set_policy ha-myqueue "^myqueue" \
  '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'

// ha-mode：
// - all（所有节点）
// - exactly（指定数量）
// - nodes（指定节点）
```

---

### 28. 负载均衡Load Balance？

#### 28.1 LB方案

```
负载均衡方案：
- HAProxy
- Nginx TCP Proxy
- AWS ALB
```

#### 28.2 HAProxy示例

```bash
listen rabbitmq
    bind 0.0.0.0:5672
    mode tcp
    balance roundrobin
    server node1 node1:5672 check inter 5000 rise 2 fall 3
    server node2 node2:5672 check inter 5000 rise 2 fall 3
    server node3 node3:5672 check inter 5000 rise 2 fall 3
```

---

### 29. Feder联邦队列？

#### 29.1 跨集群同步

```bash
# 定义上游
rabbitmqctl set_policy federate-upstream \
  '{"uri":"amqp://user:password@upstream.server","expires":3600000}' \
  --apply-to exchanges

# 或者直接配置federation
```

---

### 30. Shovel迁移工具？

#### 30.1 消息转发

```bash
# 动态Shovel
rabbitmqctl shovel declare myshovel \
  --source-protocol amqp --source-address "source-queue" \
  --destination-protocol amqp --destination-address "dest-queue"
```

---

### 31. 集群脑裂问题？

#### 31.1 脑裂处理

```
脑裂Split-brain：
- 少数服从多数
- 手动确认主节点
rabbitmqctl forget_cluster_node <node_name>
```

---

### 32. Quorum Queue？

#### 32.1 仲裁队列

```java
// Quorum Queue（5.0+）
// 复制因子为3
// 多数写入保证持久
Queue queue = Channel.queueDeclarePassive("quorum.queue");
```

---

### 33. Stream流式消息？

#### 33.1 RabbitMQ Streams

```
Streams功能：
- 消息持久保存
- Consumers间消息共享
- 回溯消费
- 消息日志
```

---

### 34. 高可用模式？

#### 34.1 HA模式

```
HA选择：
- 普通队列 + 负载均衡
- 镜像队列 + 负载均衡
- 仲裁队列 + 负载均衡
```

---

### 35. 故障转移？

#### 35.1 自动Failover

```
故障转移：
- 副本集会自动选举
- 不影响已发送消息确认
- 消费者需要重连
```

---

## 第五章 消息安全与管理（高频 ★★★★★）

### 36. 用户权限管理？

#### 36.1 权限配置

```bash
# 创建用户
rabbitmqctl add_user admin password

# 设置权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

# 分配角色
rabbitmqctl set_user_tags admin administrator
```

---

### 37. Virtual Host隔离？

#### 37.1 vhost隔离

```bash
# 创建vhost
rabbitmqctl add_vhost myvhost

# 授权
rabbitmqctl set_permissions -p myvhost user ".*" ".*" ".*"
```

---

### 38. TLS/SSL加密？

#### 38.1 TLS配置

```yaml
# rabbitmq.conf配置
listeners.ssl.default = 5671
ssl_options.cacertfile = /path/to/cacert.pem
ssl_options.certfile = /path/to/cert.pem
ssl_options.keyfile = /path/to/key.pem
ssl_options.verify = verify_peer
ssl_options.fail_if_no_peer_cert = true
```

---

### 39. 用户管理API？

#### 39.1 HTTP API

```bash
# 查看用户
curl -u guest:guest http://localhost:15672/api/users

# 创建用户
curl -u guest:guest -X PUT -H "content-type:application/json" \
  http://localhost:15672/api/users/%2F/newuser \
  -d '{"password":"password","tags":["administrator"]}'
```

---

### 40. 访问控制ACL？

#### 40.1 权限定义

```
Configure：配置（写/读）
Write：写入（写）
Read：读取（读）
```

---

## 第六章 应用集成（高频 ★★★★★）

### 41. Spring AMQP集成？

#### 41.1 Spring Boot

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 41.2 配置

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

#### 41.3 生产者

```java
rabbitTemplate.convertAndSend("exchange", "routing.key", message);
```

#### 41.4 消费者

```java
@RabbitListener(queues = "queue.name")
public void handle(Message message) {
    // 处理消息
}
```

---

### 42. Spring Cloud Stream？

#### 42.1 Binder

```java
// 定义 Binder
@EnableBinding(Source.class)
// 配置
spring.cloud.stream.bindings.output.destination=my-exchange
spring.cloud.stream.bindings.input.group=my-group
```

---

### 43. 延迟队列实现？

#### 43.1 延迟消息

```java
// 方式1：TTL + DLX
// 声明死信交换机
channel.exchangeDeclare("dlx_exchange", "direct", true);
// 声明队列绑定DLX
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx_exchange");
args.put("x-message-ttl", 1800000);
channel.queueDeclare("delay_queue", true, false, false, args);

// 方式2：插件rabbitmq_delayed_message_exchange
Map<String, Object> args = new HashMap<>();
args.put("x-delayed-type", "direct");
```

---

### 44. 死信队列DLX？

#### 44.1 Dead Letter

```java
// 死信
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dead_exchange");
args.put("x-dead-letter-routing-key", "dead.key");
channel.queueDeclare("myqueue", true, false, false, args);
```

---

### 45. 消息追踪？

#### 45.1 Firehose

```bash
# 开启Firehose
rabbitmqctl trace_on

# 或者使用rabbitmq_tracing插件
rabbitmqctl rabbitmq_traces_on
rabbitmqctl set_global_parameter name=rabbitmq_tracing pattern=json ...
```

---

### 46. 消息可靠性？

#### 46.1 可靠消息

```
可靠性保障：
- 消息持久化（durable deliveryMode）
- 交换机和队列持久化
- 确认机制（publisher confirms）
- 消费者手动ACK
- 事务（不推荐，性能差）
```

#### 46.2 Publisher Confirms

```java
// 开启confirm
channel.confirmSelect();

// 等待确认
channel.waitForConfirmsOrDie();

// 异步confirm
channel.addConfirmListener((seq, ack) -> {
    // ack成功
}, (seq, nack) -> {
    // 失败处理
});
```

---

### 47. 确保消息不丢失？

#### 47.1 丢失场景

```
消息丢失场景：
1. Producer到Broker丢失
2. Exchange到Queue丢失
3. Queue本身丢失

措施：
- publisher confirms + 持久化 + 镜像队列 +ACK确认 + 重试机制
```

---

### 48. Spring Retry重试？

#### 48.1 Retry Template

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true
          initial-interval: 3000
          max-attempts: 3
          multiplier: 2.0
```

---

### 49. 消息积压处理？

#### 49.1 积压解决

```plaintext
积压处理：
- 增加Consumer
- 增加预取prefetch
- 消息转换处理
- 暂时不ACK保证消费

预警：
- 监控队列积压数量
- 告警
```

---

### 50. 消息顺序性？

#### 50.1 顺序保证

```
消息顺序：
- 单线程发送
- 单分区内
- 消息加sequenceID
- 业务侧排序重组
```

## 第七章 运维与监控（中高级 ★★★★）

### 51. 监控插件management？

#### 51.1 监控

```bash
# 启劢插件
rabbitmq-plugins enable rabbitmq_management
rabbitmq-plugins enable rabbitmq_prometheus
```

#### 51.2 监控API

```bash
# 查看队列
curl -u guest:guest http://localhost:15672/api/queues

# overview
curl -u guest:guest http://localhost:15672/api/overview
```

---

### 52. Prometheus监控？

#### 52.1 监控

```yaml
# docker-compose.yml
- RABBITMQ_DEFAULT_GUES_USER: guest
- RABBITMQ_DEFAULT_PASS: guest
# prometheus配置
rabbitmq_plugns enable rabbitmq_prometheus
```

---

### 53. 日志分析？

#### 53.1 日志位置

```
日志位置：
- 日志文件/var/log/rabbitmq/
- 日志级别rabbitmq.conf配置
- 日志轮转log_rotation
```

---

### 54. 性能压测？

#### 54.1 Benchmark

```bash
# perfTest
# rabbitmq-perf-test 工具
java -jar perfTest.jar \
  --uri amqp://localhost \
  --producers 10 \
  --consumers 10 \
  --rate-limit 50000
```

---

### 55. 配置文件详解？

#### 55.1 rabbitmq.conf

```ini
# 配置文件格式(advanced)
[
  {rabbit, [
    {tcp_listeners, [5672]},
    {ssl_listeners, []},
    {default_vhost, <<"/">>},
    {default_user, <<"guest">>},
    {default_pass, <<"guest">>},
    {vm_memory_high_watermark, 0.4},
    {disk_free_limit, 50000000}
  ]},
  {rabbitmq_management, [
    {listener, [{port, 15672}]}
  ]}
].
```

---

### 56. 内存告警配置？

#### 56.1 内存

```ini
# 阈值配置
vm_memory_high_watermark.relative = 0.6
vm_memory_high_watermark_paging_ratio = 0.75
```

---

### 57. 磁盘告警？

#### 57.1 磁盘

```ini
# 磁盘阈值
disk_free_limit.absolute = 50MB
disk_free_limit.relative = 1.0
```

---

### 58. Mnesia数据库？

#### 58.1 后端存储

```
Mnesia：
- Erlang原生DB
- 元数据存储
- 集群信息
- 用户权限
- 等等
```

---

### 59. 日志轮转？

#### 59.1 rotate

```ini
rabbitmq.conf:log.file.rotation
rabbitmqctl rotate_logs
```

---

### 60. 管理命令？

#### 60.1 常用命令

```bash
# 用户管理
rabbitmqctl add_user admin 123456
rabbitmqctl set_permissions admin ".*" ".*" ".*"
rabbitmqctl set_user_tags admin administrator

# 队列操作
rabbitmqctl list_queues name messages consumers
rabbitmqctl list_exchanges

# 健康检查
rabbitmqctl ping
rabbitmqctl status
```

---

## 第八章 协议与扩展（中高级 ★★★★）

### 61. MQTT协议支持？

#### 61.1 MQTT

```bash
# 启劢MQTT
rabbitmq-plugins enable rabbitmq_mqtt

# 配置
tcp_listeners.start，应为5672， MQTT端口1883
```

---

### 62. STOMP协议？

#### 62.1 STOMP

```bash
rabbitmq-plugins enable rabbitmq_stomp
# 默认端口61613
```

---

### 63. Web-STOMP（WebSocket）？

#### 63.1 Web STOMP

```bash
rabbitmq-plugins enable rabbitmq_web_stomp
# 端口15674
```

---

### 64. AMQP 1.0支持？

#### 64.1 AMQP 1.0

```bash
# 启劢插件
rabbitmq-plugins enable rabbitmq_amqp1_0
rabbitmq-plugins enable rabbitmk_amqp1_0_sasl_client
```

---

### 65. HTTP API？

#### 65.1 HTTP

```bash
# API端点
http://localhost:15672/api/
http://localhost:15672/api/definitions
```

---

### 66. Shovel动态转发？

#### 66.1 Dynamic Shovel

```bash
rabbitmqctl shovel_declare name=shovel \
  src_uri=amqp://server dst_uri=amqp://client \
  src_queue=sourcequeue
```

---

### 67. Federation联邦？

#### 67.1 跨集群

```bash
# 定义上游
rabbitmqctl set_policy federate-upstream upstream1 \
  "amqp://upstream-server" ...

# 应用
rabbitmqctl set_policy federate exchange1
```

---

### 68. 插件开发？

#### 68.1 插件扩展

```
RabbitMQ Plugin:
- 自定义Exchange Type
- 自定义Validator
- 自定義Message保存
```

---

### 69. Erlang运行时？

#### 69.1 ERTS

```
Erlang特点：
- 软实时
- 分布式
- Hot Code Loading
- OTP框架
```

---

### 70. 集群协议升级？

#### 70.1 Protocol

```bash
# 协议检测
rabbitmqctl diag_report
rabbitmq-diagnostics listeners
```

---

### 71. Erlang版本兼容性？

#### 71.1 版本

```
Erlang和RabbitMQ对应关系：需要查官方文档版本矩阵
```

---

### 72. 性能调优？

#### 72.1 性能参数

```ini
# 网络
frame_max = 131072
heartbeat = 60

# 连接
channel_max = 2048
```

---

### 73. 大规模连接优化？

#### 73.1 连接

```
连接优化：
- Multiplexing多路复用
- connection per thread
- 连接池复用
```

---

### 74. 磁盘or内存队列？

#### 74.1 Disk vs Memory

```
区别：
- disk：持久化但慢
- memory：快但重启丢失(除非persistence)
- hybrid：内存+溢出到磁盘
```

---

### 75. 消息格式转换？

#### 75.1 Format Transform

```
协议转化：
- AMQP-STOMP
- AMQP- MQTT
- 消息体JSON/Protobuf
```

---

## 第九章 最佳实践（中高级 ★★★★）

### 76. 确认发送成功？

#### 76.1 Confirm

```java
// 发送确认
channel.confirmSelect();
channel.waitForConfirmsOrDie();

// 批量确认
channel.waitForConfirmsOrDie(5000);
```

---

### 77. 重复消息处理？

#### 77.1 去重

```plaintext
重复消息处理：
- 幂等性消费
- 唯一消息ID
- 数据库唯一约束
- Set数据結果
```

---

### 78. 事务消息？

#### 78.1 事务

```java
// 使用事务（不推荐）
try {
    channel.txSelect();
    channel.basicPublish(exchange, routingKey, null, body);
    channel.txCommit();
} catch (Exception e) {
    channel.txRollback();
}
```

---

### 79. 幂等性消费？

#### 79.1 Idempotent

```java
// 数据库乐观锁/分布式锁
// Redis Set NX
// 唯一MessageID
```

---

### 80. 消息积压监控？

#### 80.1 Monitor

```bash
rabbitmqctl measure_queue_rates queue_name
rabbitmq-perf-test -rate rate-limit
```

---

### 81. 优雅停机？

#### 81.1 Graceful Shutdown

```bash
# 平滑关闭
rabbitmqctl shutdown

# 强制关闭
kill -15 $(cat /var/lib/rabbitmq/mnesia/hostname.pid)
```

---

### 82. 滚动升级？

#### 82.1 Upgrade

```
升级步骤1：
- 顺序逐节点升级
- 升级顺序：次节点→主节点
- 验证集群状态
```

---

### 83. 全局唯一ID生成？

#### 83.1 ID Generator

```java
// 方式
// - Snowflake
// - Twitter snowflake-impl
// - 业务内生ID
```

---

### 84. 多租户实现？

#### 84.1 Multi-Tenant

```
多租户实现：
- 隔离VHost
- 隔离用户
- Vhost级别的resource controls
```

---

### 85. 服务质量QOS？

#### 85.1 QOS

```java
// 公平分发
channel.basicQos(1); // prefetchCount=1

// 消息堆积
// 上游流控
// 限流
```

---

### 86. 消费失败处理？

#### 86.1 Failure Handling

```java
try {
    // 业务处理

    // 手动ACK
    channel.basicAck(envelope.getDeliveryTag(), false);
} catch (Exception e) {
    // 拒绝并重入队
    channel.basicReject(envelope.getDeliveryTag(), true);
    // 或拒绝丢弃
    // channel.basicReject(envelope.getDeliveryTag(), false);
}
```

---

### 87. 顺序消费实现？

#### 87.1 Sequential Consumption

```
实现方式：
// 消息加sequenceID
// 消费者单线程分发
// 消息排序缓存
```

---

### 88. 并行消费策略？

#### 88.1 Parallel Consumption

```
并行消费：
- 单队列多消费（竞争）+ prefetch控制
- 多消费实例（无冲突）+ 分片
- 消费者分组
```

---

### 89. 消息优先级实现？

#### 89.1 Priority Queue

```java
channel.queueDeclare("priority_queue_name", true, false, false,
    Map.of("x-max-priority", 10));
channel.basicPublish("", "priority_queue_name",
    MessageProperties.PRIORITY, message.getBytes());
```

---

### 90. 生产环境调优参数？

#### 90.1 Production Tuning

```
生产环境配置：
- 预读取数量合理配比(prefetch_count)
- 消息处理超时ACK_timeout
- 连接池大小channel pool size
- 心跳heartbeat参数

# 关键参数
net_ticktime = 15
tcp_listen.backlog = 128
```

---

## 第十章 架构集成（中高 ★★★★）

### 91. RabbitMQ vs Kafka？

#### 91.1 对比

| 特性 | RabbitMQ | Kafka |
|------|---------|-------|
| 协议 | AMQP | 自定义协议 |
| 架构 | Queue + Exchange | 分区+Segment |
| 顺序保证 | 单队列顺序 | 分区顺序 |
| 消息保留 | ACK删除 | log保留 |
| 性能 | 消息级 | 批处理 |
| 复杂度 | 中 | 低 |
| 适用场景 | 队列任务 | 日志流 |

#### 91.2 选择场景

```
选择RabbitMQ：
- 任务队列
- 请求响应
- RPC
- 复杂路由

选择Kafka：
- 日志聚合
- 事件流
- CDC
- 大数据
```

---

### 92. 与Spring Boot集成？

#### 92.1 Spring Boot Consumer

```java
// 消费者
@Component
@RabbitListener(queuesToDeclare = @Queue("my.queue"))
public class Consumer {

    @RabbitHandler
    public void handle(String message) {
        System.out.println(message);
    }
}
```

---

### 93. 与Spring Cloud Netflix集成？

#### 93.1 Spring Cloud

```java
// 生产者
@Autowired
private RabbitTemplate rabbitTemplate;

// 发送
rabbitTemplate.convertAndSend("exchange", "routingKey", object);
```

---

### 94. 分布式事务？

#### 94.1 Transaction

```
分布式事务方案：
- TC+TCC (Try/Confirm/Cancel)
- Saga模式
- 消息最终一致性
- 事务消息半消息
```

---

### 95. 和业务系统结合？

#### 95.1 业务集成

```
典型集成：
- 订单系统：订单后异步通知
- 库存系统：库存解耦
- 支付系统：回调通知
- 物流系统：状态推送
```

---

### 96. 限流实现？

#### 96.1 Rate Limiting

```
限流方案：
- 生产端限流：Semaphore
- 消费端限流：prefetch
- 限流算法：Token Bucket
```

---

### 97. 监控告警指标？

#### 97.1 Metrics

```bash
监控指标：
- 队列消息数量
- 消息生产速率
- 消费延迟lag
- 连接数
- 文件描述符使用率
```

---

### 98. 性能压测指标？

#### 98.1 Benchmark

```
性能指标参考：
- 延迟 < 1ms
- 吞吐量 10K+/s
- CPU < 80%
- 内存使用稳定
```

---

### 99. 日志采集方案？

#### 99.1 Log Collection

```
架构：
- Filebeat→Kafka→Logstash→ES
- RabbitMQ可以作消息中间层
- 消费者消费后存入ES
```

---

### 100. 学习资源？

#### 100.1 Resources

```plaintext
官方文档：
- http://www.rabbitmq.com
- RabbitMQ Tutorials
- Java Client Guide
- Erlang Client
```

---

## 附录：常见面试问题

1. **RabbitMQ如何保证消息不丢失？**
   答：持久化+ACK+镜像+确认机制

2. **RabbitMQ集群如何高可用？**
   答：镜像队列+HAProxy

3. **消息顺序如何保证？**
   答：单队列+分组处理+业务顺序

4. **如何处理消息重复？**
   答：业务幂等性+唯一ID去重

5. **RabbitMQ和Kafka怎么选？**
   答：复杂路由→RabbitMQ，大数据流→Kafka

---

## 参考资料

- RabbitMQ Official Documentation
- RabbitMQ Tutorials
- Spring AMQP Documentation
- 《RabbitMQ in Depth》

---

> 整理by Claude Code | RabbitMQ面试高频100问