# DDD 100问——领域驱动设计核心深度指南

> 本文档面向DDD学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 DDD基础（高频 ★★★★★）

### 1. 什么是DDD？

#### 1.1 定义

> DDD（Domain-Driven Design，领域驱动设计）是一种软件开发方法论，通过深入理解业务领域来指导软件设计。

```plaintext
DDD核心：
- 领域模型
- 通用语言（Ubiquitous Language）
- 限界上下文（Bounded Context）
- 战略设计 + 战术设计
```

#### 1.2 回答模板

> DDD是Eric Evans提出的领域建模方法，通过通用语言和限界上下文建立领域模型。核心是" software implementation must be deeply understood as to what is important in the business domain. "

---

### 2. 为什么需要DDD？

#### 2.1 传统开发问题

```plaintext
传统开发痛点：
- 需求和代码脱节
- 贫血模型
- 事务脚本
- 难以应对变化
- 代码腐败
```

#### 2.2 DDD价值

```plaintext
DDD的价值：
- 统一语言
- 领域模型
- 上下文隔离
- 演进式设计
- 高内聚低耦合
```

#### 2.3 回答模板

> 传统开发需求和代码分离，DDD统一语言，以领域模型为核心，面对复杂业务时更能应对变化。

---

### 3. 核心概念有哪些？

#### 3.1 战略设计

```plaintext
战略设计：
- 通用语言
- 限界上下文
- 核心域
- 支撑子域
- 通用子域
```

#### 3.2 战术设计

```plaintext
战术设计：
- 实体（Entity）
- 值对象（Value Object）
- 聚合根（Aggregate Root）
- 领域服务
- 领域事件
- 仓储（Repository）
```

#### 3.3 回答模板

> DDD核心概念：战略设计包括通用语言和限界上下文；战术设计包括实体、值对象、聚合根、领域服务、仓储。

---

### 4. 通用语言是什么？

#### 4.1 定义

> 通用语言（Ubiquitous Language）是业务人员和开发人员共同使用的语言，消除歧义。

```plaintext
通用语言来源：
- 业务需求文档
- 业务流程
- 业务规则
- 会议讨论
```

#### 4.2 如何创建

```plaintext
创建流程：
1. 领域专家和开发团队一起讨论
2. 识别业务术语
3. 建立词汇表
4. 代码中应用通用语言
```

#### 4.3 回答模板

> 通用语言是团队统一的业务术语，在需求、设计、代码、测试中一致使用，确保各方理解一致。

---

### 5. 限界上下文是什么？

#### 5.1 Bounded Context

> 限界上下文是领域模型的边界，在这个边界内领域模型是完整的。

```plaintext
限界上下文划分：
- 订单上下文
- 库存上下文
- 支付上下文
- 用户上下文
```

#### 5.2 上下文映射

```plaintext
映射关系：
- 防腐层（ACL）
- 合作伙伴
- 共享内核
- 供应者/消费者
- conformist
```

#### 5.3 回答模板

> 限界上下文划分系统边界。上游下游用防腐层隔离，独立演进。

---

### 6. 核心域和支撑子域？

#### 6.1 定义

```plaintext
核心域（Core Domain）：
系统的核心竞争力，最重要的业务

支撑子域（Supporting Subdomain）：
支撑核心域但不关键

通用子域（Generic Subdomain）：
通用解决方案
```

#### 6.2 选择原则

```plaintext
投资优先级：
核心域 > 支撑子域 > 通用子域
```

#### 6.3 回答模板

> 核心域是系统中最重要的部分，需要最多投入。支撑子域支撑核心功能。

---

### 7. 什么是领域模型？

#### 7.1 领域模型定义

> 领域模型是对业务领域概念的对象表示，包含业务规则和行为。

```java
// 领域模型
public class Order {
    private OrderId id;
    private List<OrderItem> items;
    private Money totalAmount;

    public void calculateTotal() {
        totalAmount = items.stream()
            .map(OrderItem::subtotal)
            .reduce(Money.ZERO, Money::add);
    }
}
```

#### 7.2 回答模板

> 领域模型是反映业务概念和规则的代码对象，具有行为，不仅仅是数据容器。

---

### 8. 贫血模型vs充血模型？

#### 8.1 贫血模型

```java
// 贫血模型（Anti-pattern）
public class Order {
    private String id;
    private BigDecimal amount;
    // getter/setter
    // 无业务逻辑
}

// Transaction Script
public class OrderService {
    public void calculate(Order order) {
        order.setAmount(order.getItems().stream()
            .map(i -> i.getPrice().multiply(i.getQty()))
            .reduce(ZERO, ZERO::add));
    }
}
```

#### 8.2 充血模型

```java
// 充血模型（DDD）
public class Order {
    private List<OrderItem> items;

    public Money calculateTotal() {
        return items.stream()
            .map(OrderItem::subtotal)
            .reduce(Money.ZERO, Money::add);
    }
}
```

#### 8.3 回答模板

> DDD推荐充血模型，业务逻辑在领域模型中，而不是Service类。贫血模型是DDD的反模式。

---

## 第二章 战术设计（高频 ★★★★★）

### 9. 实体Entity？

#### 9.1 定义

> 实体是有唯一标识的对象，其标识在整个生命周期内保持不变。

```java
@Entity
public class Order {
    @Id
    private OrderId id;
    private Customer customer;
    private List<OrderItem> items;

    // 同一ID即使属性变化也是同一个实体
}
```

#### 9.2 特征

```plaintext
实体特征：
- 有唯一标识
- 可以修改
- 标识不变
- 有生命周期
```

#### 9.3 回答模板

> 实体有唯一标识，用ID标识身份，可以用equals/id比较。即使属性变化也仍是同一个实体。

---

### 10. 值对象Value Object？

#### 10.1 定义

> 值对象是没有唯一标识的不可变对象，用equals判断相等。

```java
// Money是值对象
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    // 无ID，无setter，不可变
}
```

#### 10.2 特征

```plaintext
值对象特征：
- 无唯一标识
- 不可变
- equals基于属性
- 可以安全共享
```

#### 10.3 回答模板

> 值对象是不可变的，没有ID，用属性值判断相等。Money/Address是典型值对象。

---

### 11. Aggregate聚合根？

#### 11.1 定义

> 聚合根是聚合的根实体，负责维护聚合内部的一致性。

```java
// Order是聚合根
public class Order { // Aggregate Root
    private OrderId id;
    private List<OrderItem> items; // 内部实体

    public void addItem(Product product, int qty) {
        // 聚合根控制内部一致性
        if (items.size() > MAX_ITEMS) {
            throw new BusinessException("超出最大商品数");
        }
        items.add(new OrderItem(product, qty));
    }

    // 外部只能通过聚合根修改内部
}
```

#### 11.2 设计原则

```plaintext
聚合原则：
- 聚合根控制所有访问
- 引用其他实体通过ID
- 聚合边界清晰
- 事务一致性边界
```

#### 11.3 回答模板

> 聚合根是领域模型的边界，所有修改通过聚合根，内部一致性由聚��根保证。

---

### 12. 领域服务Domain Service？

#### 12.1 定义

> 当业务逻辑不属于实体或值对象时，使用领域服务。

```java
public class PricingService {

    public Money calculateDiscount(Order order, Customer customer) {
        // 跨多个实体的业务逻辑
        if (customer.isVIP() && order.totalAmount().gt(1000)) {
            return order.totalAmount().multiply(0.1);
        }
        return Money.ZERO;
    }
}
```

#### 12.2 与实体方法的区别

```plaintext
领域服务 vs 实体方法：
- 领域服务：多实体协作
- 实体方法：单实体行为
```

#### 12.3 回答模板

> 领域服务处理跨实体的业务逻辑。当行为不属于任何实体或值对象时使用。

---

### 13. 领域事件Domain Event？

#### 13.1 定义

> 领域事件是领域内发生的事情，用于解耦和事件溯源。

```java
// 领域事件
public class OrderCreatedEvent {
    private OrderId orderId;
    private Instant occurredAt;
    private CustomerId customerId;
}
```

#### 13.2 应用场景

```java
// 触发事件
public class Order {
    public void submit() {
        // 业务逻辑
        DomainEvents.publish(new OrderCreatedEvent(this.id, now(), customerId()));
    }
}
```

#### 13.3 回答模板

> 领域事件用于解耦和异步处理。OrderPlaced、PaymentReceived是典型事件。

---

### 14. 仓储Repository？

#### 14.1 定义

> 仓储是领域模型和持久化之间的抽象。

```java
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    void save(Order order);
    void delete(OrderId id);
}
```

```java
// JPA实现
@Repository
public class JpaOrderRepository implements OrderRepository {
    @Override
    public Optional<Order> findById(OrderId id) {
        return repository.findById(id);
    }
}
```

#### 14.2 回答模板

> 仓储是持久化抽象，一个聚合根对应一个仓储。用DAO/DAL是另一种叫法。

---

### 15. 工厂Factory？

#### 15.1 定义

> 工厂负责创建复杂的领域对象。

```java
public class OrderFactory {
    public Order create(Customer customer, List<CartItem> items) {
        Order order = new Order(customer);
        items.forEach(item -> order.addItem(item.product(), item.qty()));
        return order;
    }
}
```

#### 15.2 回答模板

> 工厂用于创建复杂对象。简单对象用构造函数即可。

---

### 16. 应用服务Application Service？

#### 16.1 定义

> 应用服务协调领域对象，是用例的入口。

```java
@Service
public class OrderApplicationService {

    @Transactional
    public CreateOrderResponse createOrder(CreateOrderCommand cmd) {
        // 获取command对象
        Customer customer = customerRepo.findById(cmd.customerId());
        // 委托给领域模型
        Order order = orderFactory.create(customer, cmd.items());
        order.place();
        // 持久化
        orderRepo.save(order);
        // 触发事件
        DomainEvents.publish(new OrderPlacedEvent(order.id()));
        return new CreateOrderResponse(order.id());
    }
}
```

#### 16.2 回答模板

> 应用服务是用例层，协调领域对象，处理事务。一个用例一个方法。

---

### 17. CQRS命令查询职责分离？

#### 17.1 定义

> CQRS将读和写分离，使用不同的模型。

```java
// 命令模型（写）
public class Order {
    // 业务逻辑丰富
    public void place() { ... }
}

// 查询模型（读）
public class OrderView {
    // 渲染友好
    String customerName;
    List<ItemView> items;
    String status;
}
```

#### 17.2 实现

```java
// 事件溯源存储
// 命令端写入EventStore
eventStore.append(orderEvents);

// ��询��读取
orderViews = eventStore.project(OrderView.class);
```

#### 17.3 回答模板

> CQRS分离读写模型，写模型关注业务，读模型关注展示。可以事件溯源实现。

---

### 18. DDD分层架构？

#### 18.1 分层

```plaintext
DDD分层：
- Interface    接口层（API）
- Application 应用层（AppService）
- Domain      领域层（Model+Service）
- Infra       基础设施层（Repo实现）
```

#### 18.2 代码组织

```java
// Domain
com.example.domain.model.Order
com.example.domain.service.PricingService
com.example.domain.repository.OrderRepository

// Application
com.example.application.OrderService

// Infrastructure
com.example.infrastructure.persistence.JpaOrderRepository

// Interface
com.example.api.OrderController
```

#### 18.3 回答模板

> DDD经典四层：Interface → Application → Domain → Infrastructure。各层有各层职责。

---

### 19. 防腐层ACL？

#### 19.1 定义

> 防腐层（Anti-Corruption Layer）隔离外部系统的影响。

```java
// ACL转换器
public class ExternalCustomerAdapter implements CustomerRepository {

    private final ExternalCustomerClient client;

    @Override
    public Optional<Customer> findById(CustomerId id) {
        ExternalCustomerDto dto = client.fetch(id);
        return Optional.of(mapper.toDomain(dto));
    }
}
```

#### 19.2 回答模板

> 防腐层隔离外部系统，转化外部模型为本系统模型。防外部代码腐败。

---

### 20. 领域驱动设计的一般流程？

#### 20.1 实施流程

```plaintext
DDD实施：
1. 识别核心域
2. 建立通用语言
3. 划分限界上下文
4. 识别实体和值对象
5. 定义聚合和聚合根
6. 设计领域服务
7. 实现Repository
8. 编写Application Service
```

#### 20.2 回答模板

> DDD实施从识别核心域开始，逐步建立领域模型。不是一次性设计，是演进式的。

---

## 第三章 事件驱动（高频 ★★★★★）

### 21. 事件溯源Event Sourcing？

#### 21.1 定义

> 事件溯源通过存储领域事件而非状态来重建对象。

```java
// 事件存储
public class EventStore {
    public void append(DomainEvent event) {
        store.append(event);
    }

    public List<DomainEvent> getEvents(AggregateId id) {
        return store.eventsFor(id);
    }
}
```

#### 21.2 重建对象

```java
// 从事件重建
public Order reconstruct(List<DomainEvent> events) {
    return events.reduce(Order::handle);
}
```

#### 21.3 回答模板

> 事件溯源存储事件序列，重放事件重建状态。优势是完整审计溯源。

---

### 22. Saga模式？

#### 22.1 定义

> Saga通过一系列本地事务实现跨服务一致性。

```java
public class PlaceOrderSaga {

    public void execute(PlaceOrderCommand cmd) {
        // 1. 创建订单
        orderRepo.save(new Order(cmd));

        // 2. 预留库存
        try {
            inventoryService.reserve(cmd.items());
        } catch (InventoryException e) {
            // compensation
            orderRepo.cancel(cmd.orderId());
            throw e;
        }

        // 3. 扣款
        paymentService.charge(cmd.customerId(), cmd.amount());
    }
}
```

#### 22.2 补偿事务

```plaintext
Saga补偿：
- 反向操作undo
- 每个步骤有对应的Compensation
```

#### 22.3 回答模板

> Saga处理分布式事务，通过补偿事务实现最终一致性。适合跨服务长流程。

---

### 23. Event Bus事件总线？

#### 23.1 定义

> Event Bus用于发布和订阅领域事件。

```java
public interface EventBus {
    void publish(DomainEvent event);
    void subscribe(Class<? extends DomainEvent> event, EventHandler handler);
}
```

#### 23.2 回答模板

> Event Bus发布订阅式处理事件，实现松耦合。

---

### 24. 消息队列和DDD的关系？

#### 24.1 集成

```java
// 发送到消息队列
@Application
public class OrderService {
    @Autowired
    private EventPublisher publisher;

    public void placeOrder(Order order) {
        order.place();
        publisher.publish(new OrderPlacedEvent(order.id()));
    }
}

// 消费者
@Component
public class InventoryConsumer {
    @RabbitListener(queue = "order.placed")
    public void handle(OrderPlacedEvent event) {
        // 处理
    }
}
```

#### 24.2 回答模板

> RabbitMQ/Kafka集成DDD做异步处理和解耦，生产用消息队列是必须的。

---

## 第四章 微服务和DDD（中高级 ★★★★）

### 25. DDD和微服务的关系？

#### 25.1 映射关系

```plaintext
DDD 限界上下文 ≈ 微服务
单个微服务 = 1+限界上下文
边界清晰的服务 = 良好的限界上下文
```

#### 25.2 回答模板

> 限界上下文是微服务的边界。一个微服务可以包含多个聚合。

---

### 26. 每个服务应有独立的数据库？

#### 26.1 数据库隔离

```plaintext
微服务数据库：
- 订单服务 → 订单DB
- 用户服务 → 用户DB
- 库存服务 → 库存DB
- 不共享表
```

#### 26.2 回答模板

> 每个微服务有独立数据库，表不共享。跨服务通过API/事件通信。

---

### 27. API设计？

#### 27.1 REST API

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @PostMapping
    public ResponseEntity<Order> create(@RequestBody @Valid CreateOrderReq req) {
        return ok(orderService.create(req));
    }
}
```

#### 27.2 回答模板

> 微服务用REST API暴露能力。HTTP + JSON是标准。

---

### 28. 服务间通信？

#### 28.1 同步调用

```java
@Autowired
private CustomerClient customerClient;

public Order createOrder(CreateOrderReq req) {
    Customer customer = customerClient.findById(req.customerId());
    // ...
}
```

#### 28.2 异步通信

```java
// 事件驱动
@RabbitListener
public void handle(OrderCreatedEvent event) {
    notificationService.send(event.customerId(), "Order created");
}
```

#### 28.3 回答模板

> 服务间通信：同步用Feign/RestTemplate，异步用消息队列。优先异步解耦。

---

### 29. 分布式事务处理？

#### 29.1 方案

```plaintext
分布式事务：
- 2PC/XA（不推荐，性能差）
- Saga（补偿模式）
- TCC（Try-Confirm-Cancel）
- 最终一致性 + 重试
```

#### 29.2的回答模板

> 分布式事务避免用2PC。Saga、TCC、最终一致是常用方案。

---

### 30. 如何划分微服务边界？

#### 30.1 划分原则

```plaintext
边界划分：
- 单一职责
- 限界上下文
- 业务能力
- 团队
```

#### 30.2 回答模板

> 微服务边界参考限界上下文和业务能力。不是越小越好，适合最重要。

---

## 第五章 建模实践（中高级 ★★★★）

### 31. 事件风暴Event Storming？

#### 31.1 定义

> 事件风暴通过识别领域事件来建模。

```plaintext
Event Storming流程：
1. 识别领域中发生了什么（事件）
2. 谁触发了这些事件（命令）
3. 什么实体参与了（实体）
4. 在哪里划分界限（边界）
```

#### 31.2 步骤

```markdown
1. Event: 订单已下单 → 命令: 下单 → 实体: Order
2. Event: 库存已预留 → 命令: 预留 → 实体: Inventory
3. Event: 支付完成 → 命令: 支付 → 实体: Payment
```

#### 31.3 回答模板

> 事件风暴是DDD建模工作坊，识别事件→命令→实体→聚合。Alberto Brandolini发明。

---

### 32. 领域模型重构时机？

#### 32.1 重构信号

```plaintext
需要重构：
- 模型膨胀（太多方法）
- 频繁修改同一个对象
- 利斯科夫替代
- 重复代码
```

#### 32.2 回答模板

> 重构发生在模型开始腐化时。敏捷迭代，允许推翻模型。

---

### 33. DDD落地难点？

#### 33.1 挑战

```plaintext
落地难点：
- 组织壁垒
- 语言统一困难
- 过度设计
- 技术债务
- 学习成本
```

#### 33.2 回答模板

> DDD最大难点是统一语言，业务和研发理解常常不一致。循序渐进。

---

### 34. 领域模型vs数据模型？

#### 34.1 区别

| 方面 | 领域模型 | 数据模型 |
|------|----------|--------|
| 目的 | 业务表达 | 数据存储 |
| 关注点 | 行为 | 属性 |
| 结构 | 聚合 | 表 |
| 变化 | 业务驱动 | 产品驱动 |

#### 34.2 回答模板

> 领域模型反映业务，数据模型反映存储。两者可以映射不必完全一致。

---

### 35. 模型教化？

#### 35.1 定义

> 模型教化让代码成为通用语言的载体。

```java
// 业务语言体现在代码
if (order.isOverdue()) { // OK
vs
if (now().compareTo(order.getCreateTime() + 30days) > 0) // NO
```

#### 35.2 回答模板

> 代码应使用业务语言，使通用语言成为代码的一部分。

---

### 36. 泛洪问题？

#### 36.1 问题

> 泛洪是指添加太多不必要的内容。

```plaintext
过度建模：
- 过细的聚合
- 过深的继承
- 过抽象的基类
```

#### 36.2 回答模板

> DDD应渐进式设计。不要为未来设计，YAGNI原则。

---

### 37. 贫血模型的根源？

#### 37.1 原因

```plaintext
贫血模型：
- ORM影响
- 事务脚本习惯
- 团队不了解DDD
- 时间压力
```

#### 37.2 解决

```java
// 行为移到领域模型
public class Order {
    public void addItem(Product p, int qty) { ... }
}
```

---

### 38. 如何发现隐藏的概念？

#### 38.1 识别

```plaintext
隐藏概念：
- 命名不一致
- 重复逻辑
- 隐含的规则
- 可变的业务规则
```

#### 38.2 发现方法

```markdown
1. 与业务专家对话
2. 分析现有代码和文档
3. 寻找重复的模式
4. 业务规则变化点
```

---

### 39. 大多数DDD项目失败的原因？

#### 39.1 失败原因

```plaintext
失败原因：
- 只有技术团队参与
- 缺乏业务专家
- 过度设计
- 急于求成
- 没有持续
```

#### 39.2 成功要素

```plaintext
成功要素：
- 业务+技术一起
- 高层支持
- 小步快跑
- 回馈迭代
```

---

### 40. DDD代码组织结构？

#### 40.1 推荐的包结构

```java
com.company.{module}
├── domain/
│   ├── model/
│   │   ├── {aggregate}/
│   │   │   ├── {AggregateRoot}.java
│   │   │   ├── {Entity}.java
│   │   │   └── {ValueObject}.java
│   │   ├── service/
│   │   └── event/
│   ├── application/
│   ├── infrastructure/
│   └── interface/
```

#### 40.2 回答模板

> DDD代码按领域分层组织。模块→层→聚合→类。

---

## 第六章 实践案例（中高级 ★★★★）

### 41. 电商领域建模示例？

#### 41.1 核心域识别

```plaintext
电商核心域：
- 订单域（Core）
- 支付域（Core）
- 库存域（Support）
- 用户域（Support）
- 商品域（Support）
```

#### 41.2 聚合设计

```java
// 订单聚合
public class Order {
    private OrderId id;
    private List<OrderLine> lines;
    private Money total;
    private OrderStatus status;
}

// 订单行
public class OrderLine {
    private ProductId productId;
    private int quantity;
    private Money price;
}

// 值对象
public class Money { }
```

#### 41.3 回答模板

> 电商DDD：Core是订单/支付，支撑是用户/库存/物流。Order是核心聚合。

---

### 42. 订单聚合的状态机？

#### 42.1 Order状态

```java
public enum OrderStatus {
    DRAFT,          // 草稿
    SUBMITTED,      // 已提交
    PAID,          // 已支付
    PROCESSING,    // 处理中
    SHIPPED,       // 已发货
    DELIVERED,    // 已送达
    CANCELLED      // 已取消
}
```

#### 42.2 状态转移

```java
public class Order {
    public void submit() {
        if (status != DRAFT) throw new IllegalStateException();
        this.status = SUBMITTED;
    }

    public void pay(PaymentInfo payment) {
        if (status != SUBMITTED) throw new IllegalStateException();
        // 支付逻辑
        this.status = PAID;
    }
}
```

#### 42.3 回答模板

> 订单状态机用枚举+状态转换方法控制。每种状态只有合法的操作。

---

### 43. 如何实现促销优惠计算？

#### 43.1 策略模式

```java
public interface PromotionCalculator {
    Money calculate(Order order);
}

public class VipDiscount implements PromotionCalculator {
    @Override
    public Money calculate(Order order) {
        if (order.customer().isVIP()) {
            return order.subtotal().multiply(0.1);
        }
        return Money.ZERO;
    }
}
```

#### 43.2 回答模板

> 促销规则用策略模式是可变的业务规则，应该抽取为独立的领域服务。

---

### 44. 库存领域模型？

#### 44.1 SKU和Stock

```java
// SKU实体
public class Sku {
    private SkuId id;
    private String code;
    private String name;

    // 扣减库存
    public void deduct(int quantity) {
        if (available < quantity) {
            throw new OutOfStockException();
        }
        this.available -= quantity;
    }
}

// 库存聚合
public class Stock {
    private List<SkuStock> stocks;
}
```

#### 44.2 回答模板

> SKU是商品编码，Stock是具体库存。扣减库存要检查库存是否足够。

---

### 45. 支付领域模型？

#### 45.1 Payment

```java
public class Payment {
    private PaymentId id;
    private OrderId orderId;
    private Money amount;
    private PaymentMethod method;
    private PaymentStatus status;

    public void pay() {
        if (!canPay()) throw new IllegalStateException();
        // 调用支付网关
        this.status = SUCCESS;
    }
}
```

#### 45.2 回答模板

> Payment是支付聚合根，处理支付逻辑。外部支付网关通过防腐层调用。

---

### 46. 用户模型设计？

#### 46.1 User聚合

```java
public class User {
    private UserId id;
    private Username username;
    private Password password;
    private Email email;
    private List<Role> roles;

    public void assignRole(Role role) {
        this.roles.add(role);
    }
}
```

#### 46.2 回答模板

> 用户聚合包含用户名、密码、邮箱、角色。密码要加密存储。

---

### 47. 优惠券模型？

#### 47.1 Coupon聚合

```java
public class Coupon {
    private CouponId id;
    private String code;
    private DiscountType type; // FIXED/PERCENT
    private Money value;
    private DateRange validity;

    public boolean isApplicable(Order order) {
        return validity.contains(now()) &&
               !used &&
               meetsCondition(order);
    }
}
```

#### 47.2 回答模板

> 优惠券：优惠码、折扣类型、金额/比例、有效期、使用条件。

---

### 48. 购物车模型？

#### 48.1 Cart聚合

```java
public class Cart {
    private CartId id;
    private CustomerId customerId;
    private List<CartItem> items;

    public void addItem(Product product, int qty) {
        // 检查库存
        // 检查限购
        items.add(new CartItem(product, qty));
    }
}
```

#### 48.2 回答模板

>  Cart是临时购物车，会员登录才有。结算时转换为订单。

---

### 49. 物流领域模型？

#### 49.1 Logistics

```java
public class Logistics {
    private LogisticsId id;
    private OrderId orderId;
    private DeliveryInfo deliveryInfo;
    private TrackingNumber trackingNo;
    private DeliveryStatus status;
}
```

#### 49.2 回答模板

> 物流记录配送信息：收件人、地址、快递单号、状态。

---

### 50. 评论/评分模型？

#### 50.1 Review

```java
public class Review {
    private ReviewId id;
    private OrderId orderId;
    private ProductId productId;
    private int rating; // 1-5
    private String comment;
    private List<ReviewImage> images;
    private ReviewStatus status;
}
```

#### 50.2 回答模板

> Review评分：商品评价、评分、图片。防刷评要用订单关联。

---

## 第七章 DDD进阶（中高级 ★★★★）

### 51. 六边形架构HEX？

#### 51.1 定义

```plaintext
六边形架构：
- 核心是领域
- 左边是驱动（输入）
- 右边是被驱动（输出）
```

#### 51.2 代码

```java
// 端口（接口）
public interface OrderRepository {
    void save(Order order);
}

// 适配器（实现）
JpaOrderRepository implements OrderRepository
```

#### 51.3 回答模板

> 六边形架构：核心领域通过端口与外部Adapter隔离。好测试、易扩展。

---

### 52. 洋葱架构Onion Architecture？

#### 52.1 结构

```plaintext
Onion：
- Core Domain （中心）
- Application（内层）
- Infrastructure（外层）
```

#### 52.2 回答模板

> 洋葱架构和六边形类似，强调依赖方向只能是向内。

---

### 53. 整洁架构Clean Architecture？

#### 53.1 层

```plaintext
Clean Architecture：
- Entities      实体
- Use Cases    用例
- Interface    接口
- Frameworks   框架
```

#### 53.2 回答模板

> Clean Architecture是Robert Martin提出，和DDD分层对应。

---

### 54. CQRS具体实现？

#### 54.1 代码

```java
// 命令端
public class CreateOrderCommand {
    public void execute(Order order) {
        order.validate();
        orderRepository.save(order);
    }
}

// 查询端
public interface OrderQueryService {
    OrderDTO findById(OrderId id);
    List<OrderListDTO> findByCustomer(CustomerId cid);
}
```

#### 54.2 回答模板

> CQRS分离读写。写端处理Command，读端处理Query。

---

### 55. DDD和设计模式的关系？

#### 55.1 常用设计模式

```plaintext
DDD常用模式：
- Factory  创建复杂对象
- Builder 创建复杂对象
- Strategy 业务规则
- State    状态机
- Observer 领域事件
- Repository 数据持久化
```

#### 55.2 回答模板

> DDD运用经典设计模式。更强调领域概念的表达。

---

### 56. DDD和事务脚本混用？

#### 56.1 混用场景

```plaintext
简单CRUD：
- 用Transaction Script
- 报表查询
- 批量操作
```

#### 56.2 回答模板

> DDD不一定全用。对于简单场景，Transaction Script更轻量。

---

### 57. 测试驱动DDD？

#### 57.1 ATDD

```java
@Test
public void should_create_order_when_items_valid() {
    given_customer_has_valid_cart();
    when_create_order();
    then_order_created();
    then_event_published(OrderCreatedEvent);
}
```

#### 57.2 回答模板

> 测试优先ATDD，写出测试再写实现。测试即文档。

---

### 58. 领域模型测试？

#### 58.1 单元测试

```java
@Test
public void submit_should_throw_when_status_not_draft() {
    Order order = new Order();
    order.submit(); // status = SUBMITTED

    assertThrows(IllegalStateException.class, () -> order.submit());
}
```

#### 58.2 回答模板

> 领域模型测试：验证业务规则和行为。mock外部依赖。

---

### 59. 集成测试？

#### 59.1 集成

```java
@SpringBootTest
@Test
public void should_persist_and_retrieve_order() {
    Order order = new Order(customer, items);
    repository.save(order);

    Order found = repository.findById(order.id());
    assertThat(found).isEqualTo(order);
}
```

#### 59.2 回答模板

> 集成测试用真实DB，验证Repository实现。

---

### 60. EDA事件驱动架构？

#### 60.1 事件驱动

```plaintext
EDA:
- 事件发布
- 事件路由
- 事件消费
```

#### 60.2 回答模板

> 事件驱动EDA松耦合。DDD领域事件是EDA的一种形式。

---

## 第八章 工程实践（高级 ★★★）

### 61. DDD技术选型？

#### 61.1 Java栈

```plaintext
DDD Java技术栈：
- Java 17+
- Spring Boot 3
- Spring Data JPA/DDD
- Doma2
- Axon Framework
```

#### 61.2 回答模板

> Java DDD：AxonFramework支持CQRS/ES。SpringBoot+DDD最常见。

---

### 62. Spring DDD支持？

#### 62.1 使用

```java
@Aggregate
public class Order {
    @AggregateIdentifier
    private OrderId id;
}
```

#### 62.2 回答模板

> Spring Modulith提供DDD支持。也可以纯手写。

---

### 63. 领域事件发布方式？

#### 63.1 方式

```java
// ApplicationEventPublisher
@Autowired
private ApplicationEventPublisher publisher;

public void place() {
    status = PLACED;
    publisher.publishEvent(new OrderPlacedEvent(id));
}
```

#### 63.2 回答模板

> Spring ApplicationEventPublisher发布领域事件。

---

### 64. Repository实现选择？

#### 64.1 JPA

```java
@Repository
public class JpaOrderRepository implements OrderRepository {
    @PersistenceContext
    private EntityManager em;
}
```

#### 64.2 回答模板

> JPA Entity Manager是标准实现。复杂查询用QueryDSL。

---

### 65. DDD和Spring Data？

#### 65.1 结合

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, OrderId> {
}
```

#### 65.2 回答模板

> Spring Data JPA+DDD一个Repository接口继承JpaRepository。

---

### 66. 聚合持久化策略？

#### 66.1 Unit of Work

```java
@Transactional
public class OrderService {
    @Autowired
    private OrderRepository repository;

    public void create() {
        Order order = new Order(...);
        repository.save(order);
    }
}
```

#### 66.2 回答模板

> 一个事务对应一个聚合。跨聚合用Saga。

---

### 67. 如何保证聚合一致性？

#### 67.1 原则

```plaintext
一致性保证：
- 一个事务只修改一个聚合
- 强制业务规则在聚合根
- 跨聚合引用ID
```

#### 67.2 回答模板

> 一个事务修改一个聚合根。外部只能通过ID引用。

---

### 68. 大纲领域建模方法？

#### 68.1 EventStorming步骤

```markdown
1. 收集所有Domain Event
2. 添加Command（触发事件的动作）
3. 识别Actor（触发者）
4. 识别Aggregate
5. 定义Bounded Context
```

#### 68.2 回答模板

> 事件风暴是DDD快速建模方法论。用便利贴工作坊。

---

### 69. 如何处理分布式DDD？

#### 69.1 数据模型

```json
// 订单聚合数据
{
  "orderId": "uuid",
  "lines": [...],
  "status": "SUBMITTED",
  "version": 1
}
```

#### 69.2 回答模板

> 分布式DDD序列化JSON存储。版本号用于乐观锁。

---

### 70. DDD和GraphQL？

#### 70.1 schema

```graphql
type Order {
  id: ID!
  lines: [OrderLine!]!
  total: Money!
  status: OrderStatus!
}

type Query {
  order(id: ID!): Order
  orders(customerId: ID!): [Order!]!
}
```

#### 70.2 回答模板

> GraphQL是API层一种实现，聚合 facets 可以映射为GraphQL types。

---

## 第九章 架构演进（中高级 ★★★★）

### 71. 从单体到DDD的迁移策略？

#### 71.1 绞杀模式

```plaintext
迁移策略：
1. 识别核心域
2. 逐步迁移
3. 实现ACL防腐层
4. 替换旧服务
```

#### 71.2 回答模板

> 迁移DDD用Strangler Fig模式：外面用ACL包裹旧服务，内部建新服务。

---

### 72. 如何验证DDD模型的质量？

#### 72.1 反腐

```plaintext
检验模型：
- 是否反映了业务语言
- 是否是高内聚
- 是否是稳定的
- 是否方便测试
```

#### 72.2 回答模板

> 好的DDD模型：业务语言自然、无所不在的setter、行为清晰、高内聚。

---

### 73. DDD和契约式设计？

#### 73.1 Specification模式

```java
public class OrderSpecification {
    public static Specification<Order> isOverdue() {
        return (root, query, cb) ->
            cb.lessThan(root.get("createdAt"), now().minusDays(30));
    }
}
```

#### 73.2 回答模板

> Specification模式描述业务规则，可以复用和组合。

---

### 74. 不变性Invariant实现？

#### 74.1 Enforcement

```java
public class Order {
    private OrderStatus status;

    public void changeItemQuantity(int index, int delta) {
        // Enforce Invariants
        if (status != DRAFT)
            throw new IllegalStateException("只能在草稿状态修改");
        // 校验
        items.get(index).increment(delta);
    }
}
```

#### 74.2 回答模板

> 不变性通过方法内强制校验保证。任何状态变化经过方法。

---

### 75. DDD中的告警警告设计？

#### 75.1 Domain Warning

```java
public class Order {
    private List<String> warnings;

    public void submit() {
        if (warnings.hasLowStock()) {
            warnings.add("Warning: Low Stock");
        }
    }
}
```

#### 75.2 回答模板

> 警告和错误分开。Warning是允许通过的，但需要记录。

---

### 76. 延迟加载在DDD中的应用？

#### 76.1 延迟加载

```java
public class Order {
    @OneToMany(fetch = LAZY)
    private List<OrderItem> items;

    public BigDecimal getTotalForce() {
        return items.stream() // 触发lazy
            .map(OrderItem::subtotal)
            .reduce(ZERO, Money::add;
    }
}
```

#### 76.2 回答模板

> 关联用LAZY，调用时加载。避免N+1问题。

---

### 77. 多态关联处理？

#### 77.1 处理方式

```java
// 多态引用
public abstract class Billing {
    // 引用抽象
    protected PaymentMethod method;
}

public class CreditCard extends PaymentMethod {}
public class Alipay extends PaymentMethod {}
```

#### 77.2 回答模板

> 多态用继承树建模。JPA用TABLE_PER_CLASS。

---

### 78. DDD处理时间相关概念？

#### 78.1 时间段

```java
public class TimePeriod {
    private Instant start;
    private Instant end;

    public boolean contains(Instant moment) {
        return moment.isAfter(start) && moment.isBefore(end);
    }
}
```

#### 78.2 回答模板

> 活动时间用TimePeriod值对象。活动是时间段。

---

### 79. 复杂业务规则的优先级处理？

#### 79.1 优先级

```java
public class PricingService {
    public Money calculate(Order order) {
        // 优先级1: 会员等级折扣
        // 优先级2: 活动折扣
        // 优先级3: 优惠券
        // ...
    }
}
```

#### 79.2 回答模板

> 规则按优��级��序应用。用策略模式使规则可配置。

---

### 80. DDD处理批处理？

#### 80.1 Batch

```java
@Service
public class BatchOrderService {
    public void settleExpiredOrders() {
        orders.findByStatusOverdue().forEach(Order::settle);
    }
}
```

#### 80.2 领域事件配合定时任务

```java
@Scheduled(cron = "0 0 2 * * ?")
public void dailySettle() {
    // 每日2点结算
}
```

---

## 第十章 最佳实践与反模式（中高级 ★★★★）

### 81. 常见的DDD反模式？

#### 81.1 Anti-patterns

```plaintext
反模式：
- 贫血模型（Anemic Domain Model）
- 分布式单体（Distributed Monolith）
- 过度抽象
- 泛洪建模
- Everything as aggregate
```

#### 81.2 回答模板

> DDD常见反模式：贫血模型、把所有东西都作为聚合、过度设计。

---

### 82. DDD适合什么样的项目？

#### 82.1 适合

```plaintext
DDD适合：
- 复杂业务逻辑
- 业务频繁变化
- 大型企业应用
- 团队规模较大
```

#### 82.2 不适合

```plaintext
DDD不适合：
- 简单CRUD
- 报表
- 小型工具
- 业务稳定
```

#### 82.3 回答模板

> DDD用于复杂业务，不是银弹。简单的系统不要DDD。

---

### 83. DDD落地的团队要求？

#### 83.1 团队构成

```plaintext
DDD团队：
- 至少一个熟悉业务的
- 至少一个DDD经验的
- 业务专家参与（关键）
```

#### 83.2 回答模板

> DDD成功的关键是业务+技术一起。纯技术团队没用。

---

### 84. 怎样快速写出好的DDD代码？

#### 84.1 实践

```plaintext
快速上手：
1. 识别核心业务
2. 画简单的领域模型
3. 找一个聚合实验
4. 用充血模型
5. 写测试
```

#### 84.2 回答模板

> DDD渐进式落地。先小范围实验，成功后再推广。

---

### 85. DDD建模工具？

#### 85.1 工具

```plaintext
建模工具：
- Miro
- Lucidchart
- Visual Paradigm
- ProcessOn
```

#### 85.2 回答模板

> 领域模型画图工具很重要，推荐Miro或ProcessOn。

---

### 86. 领域事件的版本控制？

#### 86.1 版本

```java
public class OrderPlacedEvent {
    private static final int VERSION = 2;
    // V1字段
    // V2增加字段
}
```

#### 86.2 回答模板

> 事件有版本号，用upcaster处理字段变化。

---

### 87. DDD如何处理文件上传下载？

#### 87.1 文件处理

```java
// 引用文件
public class OrderAttachment {
    private String fileId; // 文件存储ID
    private String fileName;
    private FileType type;
}
```

#### 87.2 回答模板

> DDD中不存储文件内容，而是存文件引用（ID指向OSS）。

---

### 88. 日志追踪和DDD？

#### 88.1 Trace ID

```java
MDC.put("traceId", Context.get().getTraceId());
logger.info("Order placed: {}", order.id());
```

#### 88.2 回答模板

> 跨服务用traceId追踪。MDC存traceId，日志聚合查。

---

### 89. DDD和日志聚合？

#### 89.1 ELK集成

```java
public class Order {
    public void submit() {
        logger.info("Order submitted: {}", id);
    }
}
```

#### 89.2 回答模板

> 结构化JSON日志存ES，聚合ELK stack。

---

### 90. 领域特定语言DSL？

#### 90.1 类库

```java
// 流式API
Order.create()
    .withItems(items)
    .withCustomer(customer)
    .applyDiscount(discount)
    .place();
```

#### 90.2 回答模板

> DSL让代码更像业务语言。流式Builder实现。

---

### 91. DDD处理并发？

#### 91.1 乐观锁

```java
@Version
private Long version;

public void change() {
    if (version != expected)
        throw new OptimisticLockException();
}
```

#### 91.2 回���模板

> 聚合用@Version乐观锁，并发更新抛出异常。

---

### 92. 领域模型和性能优化？

#### 92.1 缓存

```java
@Service
public class ProductCache {
    @Cacheable("products")
    public Product findById(ProductId id) {
        return repository.findById(id);
    }
}
```

#### 92.2 回答模板

> 热点数据缓存。DDD模型本身不加缓存，由repository层处理。

---

### 93. 领域事件丢失处理？

#### 93.1 补偿

```java
// 定时检查未处理事件
@Scheduled(fixedDelay = 60000)
public void checkUnpublished() {
    events.findUnpublished().forEach(EventBus::publish);
}
```

#### 93.2 回答模板

> 事件发布失败存数据库，定时重试投递。

---

### 94. DDD处理幂等性？

#### 94.1 Idempotent

```java
public class PaymentService {
    @Transactional
    public void process(PaymentCommand cmd) {
        // 幂等检查
        if (idempotentService.checkProcessed(cmd.paymentId())) {
            return;
        }
        // 处理
    }
}
```

#### 94.2 回答模板

> 外部请求幂等用唯一ID检查，防止重复处理。

---

### 95. DDD处理外部API异常？

#### 95.1 异常处理

```java
public class ExternalCallAdapter {
    public OrderStatus getStatus(String orderNo) {
        try {
            return client.getStatus(orderNo);
        } catch (Exception e) {
            // 降级处理
            return OrderStatus.UNKNOWN;
        }
    }
}
```

#### 95.2 回答模板

> 外部调用用Adapter包装，降级处理，异常不要扩散。

---

### 96. DDD代码review清单？

#### 96.1 代码review

```markdown
DDD代码review：
- [ ] 充血模型？有业务逻辑？
- [ ] 唯一ID标识？
- [ ] 不变性保证？
- [ ] 通用语言？
- [ ] 聚合边界清晰？
```

#### 96.2 回答模板

> 关键点：行为在Model、无所不在的setter、清晰的通用语言。

---

### 97. DDD和响应式编程？

#### 97.1 Reactive

```java
public Mono<Order> placeOrder(OrderCommand cmd) {
    return Mono.fromCallable(() -> domain.place(cmd))
              .subscribeOn(Schedulers.boundedElastic());
}
```

#### 97.2 回答模板

> 响应式和DDD正交。DDD负责领域，Reactive负责异步。

---

### 98. DDD的多租户处理？

#### 98.1 Tenant

```java
public class TenantContext {
    static ThreadLocal<String> current = new ThreadLocal<>();
}
```

#### 98.2 回答模板

> 多租户用TenantContext隔离，数据用TenantID过滤。

---

### 99. 领域模型版本迁移？

#### 99.1 Migration

```java
@Mapper(unmappedTargetPolicy = IGNORE)
public class OrderMapper {
    // V1 -> V2
}
```

#### 99.2 回答模板

> 模型升级用Migration脚本，数据要兼容转换。

---

### 100. DDD学习资源推荐？

#### 100.1 Book推荐

```plaintext
DDD书籍：
- 《Domain-Driven Design》- Eric Evans（经典）
- 《Implementing Domain-Driven Design》- Vaughn Vernon
- 《Domain-Driven Design Distilled》- Vaughn Vernon
- 《IDDD Guide》
```

#### 100.2 在线资源

```plaintext
在线：
- DDD Crew
- Va Vernon的IDDD articles
- Spring Blog DDD
- Microsoft eShopOnDDD
```

#### 100.3 回答模板

> 必读Eric Evan的蓝皮书。Spring PetClinic是很好的参考实现。

---

## 附录：面试追问

1. **项目中DDD如何落地？**
   答：选择核心域→建立通用语言→划分限界上下文→建模

2. **DDD和微服务的关系？**
   答：限界上下文≈微服务

3. **DDD的挑战是什么？**
   答：统一业务语言最难

4. **什么时候用DDD？**
   答：复杂业务

---

## 参考资料

- 《Domain-Driven Design》- Eric Evans
- 《Implementing Domain-Driven Design》- Vaughn Vernon
- 《Domain-Driven Design Distilled》

---

> 整理by Claude Code | DDD面试高频100问