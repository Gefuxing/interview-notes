# 设计模式100问——设计模式核心技术深度指南

> 本文档面向设计模式学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 概述与原则（高频 ★★★★★）

### 1. 什么是设计模式？

#### 1.1 设计模式定义

> 设计模式是针对软件开发中常见问题的可复用解决方案，是对经过验证的可行实践的结构化描述。

```plaintext
设计模式发展：
1994：GoF《设计模式》- 23种基础模式
1995：Smalltalk MVC 模式
2002：PoEAA（企业应用架构模式）
2009：DDD（领域驱动设计）
```

#### 1.2 三大类型

```plaintext
创建型模式（Creational）     → 关注对象创建
├── Singleton              单例
├── Factory Method        工厂方法
├── Abstract Factory     抽象工厂
├── Builder              建造者
└── Prototype            原型

结构型模式（Structural）   → 关注对象组合
├── Adapter              适配器
├── Bridge               桥接
├── Composite            组合
├── Decorator            装饰器
├── Facade               外观
├── Proxy                代理
└── Flyweight            享元

行为型模式（Behavioral） → 关注对象交互
├── Iterator             迭代器
├── Observer             观察者
├── Strategy             策略
├── Template Method     模板方法
├── Command             命令
├── State                状态
├── Chain of Responsibility 责任链
└── Visitor              访问者
```

#### 1.3 回答模板

> 设计模式是软件设计中常见问题的成熟解决方案。GoF的23种模式分为创建型（对象创建）、结构型（对象组合）、行为型（对象交互）三类。

---

### 2. SOLID 原则？

#### 2.1 五大原则

```
S - Single Responsibility Principle  单一职责
O - Open Closed Principle             开闭原则
L - Liskov Substitution Principle     里氏替换
I - Interface Segregation Principle  接口隔离
D - Dependency Inversion Principle   依赖倒置
```

#### 2.2 详解

```plaintext
1. 单一职责 SRP
   一个类只负责一项职责，容易维护

2. 开闭原则 OCP
   对扩展开放，对修改关闭

3. 里氏替换 LSP
   子类能够替换父类而不影响功能

4. 接口隔离 ISP
   使用多个专门的小接口优于一个大的接口

5. 依赖倒置 DIP
   依赖抽象而不是具体
```

#### 2.3 回答模板

> SOLID是面向对象设计的五大原则：单一职责（类职责单一）、开闭（扩展开放修改关闭）、里氏替换（子类替换父类）、接口隔离（小接口）、依赖倒置（依赖抽象）。

---

### 3. 设计模式的本质？

#### 3.1 核心思想

```plaintext
程序设计的核心：
1. 寻找变化点 → 封装变化点
2. 面向接口编程 → 依赖抽象
3. 组合优于继承 → 灵活组合
4. 职责单一 → SRP
```

#### 3.2 模式选择指引

```plaintext
需要对象创建时： 工厂、单例、建造者、原型
需要对象结构时： 适配器、装饰、代理、门面
需要对象行为时： 策略、模板、观察者、命令
需要对象组合时： 组合、迭代、享元
```

#### 3.3 回答模板

> 设计模式本质是围绕变化点封装、代码复用、职责分离。好的设计让系统更灵活、易扩展、好维护。

---

## 第二章 创建型模式（高频 ★★★★★）

### 4. 单例模式 Singleton？

#### 4.1饿汉式

```go
// Go实现（线程安全）
type Singleton struct {
    Data string
}

var inst *Singleton

func GetInstance() *Singleton {
    if inst == nil {
        inst = &Singleton{}
    }
    return inst
}

// 懒汉式
type LazySingleton struct {
    Data string
}

var lazy *LazySingleton
once.Do(func() {
    lazy = &LazySingleton{Data: "data"}
})

func GetLazy() *LazySingleton {
    return lazy
}
```

#### 4.2 双检锁

```java
// Java 双检锁 DCL
public class Singleton {
    private volatile static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

#### 4.3 回答模板

> Singleton单例确保一个类只有一个实例。Go可以once.Do实现线程安全。懒汉式延后创建，饿汉式类加载时创建。Spring IOC是容器管理单例。

---

### 5. 工厂方法模式？

#### 5.1 简单工厂

```go
type Api interface {
    Say() string
}

type ImplA struct {}
func (a *ImplA) Say() string { return "A" }

type ImplB struct {}
func (b *ImplB) Say() string { return "B" }

func NewApi(t string) Api {
    if t == "A" {
        return &ImplA{}
    }
    return &ImplB{}
}
```

#### 5.2 工厂方法

```go
// 工厂各自创建产品
type Factory interface {
    Create() Api
}

type FactoryA struct{}
func (f *FactoryA) Create() Api { return &ImplA{} }

type FactoryB struct{}
func (f *FactoryB) Create() Api { return &ImplB{} }
```

#### 5.3 抽象工厂

```go
// 抽象工厂创建一组相关产品
type AbstractFactory interface {
    CreateProductA() ProductA
    CreateProductB() ProductB
}
```

#### 5.4 回答模板

> 简单工厂一个工厂创建所有产品，工厂方法子类创建各产品，抽象工厂创建一组相关产品。JDBC是工厂方法应用的典型。

---

### 6. 建造者模式 Builder？

#### 6.1 定义

```go
type Builder interface {
    BuildPartA() Builder
    BuildPartB() Builder
    BuildPartC() Builder
    GetResult() interface{}
}

type Director struct {
    builder Builder
}

func NewDirector(builder Builder) *Director {
    return &Director{builder: builder}
}

func (d *Director) Construct() {
    d.builder.BuildPartA().BuildPartB().BuildPartC()
}
```

#### 6.2 实际应用

```go
// StringBuilder
sb := strings.Builder{}
sb.WriteString("hello")
sb.WriteString("world")

// HTTP Request
req := NewRequestBuilder().
    SetURL("http://example.com").
    SetMethod("POST").
    SetBody(data).
    Build()
```

#### 6.3 回答模板

> 建造者模式分步骤创建复杂对象，产品是逐步构建出来的。StringBuilder、RequestBuilder是常见应用。链式调用Builder。

---

### 7. 原型模式 Prototype？

#### 7.1 定义

```go
// Go中的原型模式
func Clone(src interface{}) interface{} {
    // 深拷贝
    data, _ := json.Marshal(src)
    clone := reflect.New(reflect.TypeOf(src).Elem()).Interface()
    json.Unmarshal(data, clone)
    return clone
}
```

#### 7.2 应用

```go
// prototype
func (p *ConcretePrototype) Clone() *ConcretePrototype {
    clone := *p
    return &clone
}
```

#### 7.3 回答模板

> 原型模式通过复制现有对象创建新对象。Java用Cloneable接口，Go可用序列化实现深拷贝。文案复制是典型应用。

---

## 第三章 结构型模式（高频 ★★★★★）

### 8. 适配器模式 Adapter？

#### 8.1 定义

```go
// Target接口
type Target interface {
    Request() string

// Adaptee (不兼容)
type Adaptee struct{}
func (a *Adaptee) SpecificRequest() string {
    return "specific"
}

// Adapter
type Adapter struct {
    Adaptee *Adaptee
}

func (a *Adapter) Request() string {
    return a.Adaptee.SpecificRequest()
}
```

#### 8.2 使用场景

```plaintext
适配器使用场景：
- 旧代码接口不兼容
- 第三方库的整合
- 标准接口与遗留系统
```

#### 8.3 回答模板

> 适配器让不兼容的接口能够一起工作。生活中类似的电源适配器。Wrapper模式之一。

---

### 9. 装饰器模式 Decorator？

#### 9.1 定义

```go
type Component interface {
    Operation() string
}

type ConcreteComponent struct{}
func (c *ConcreteComponent) Operation() string {
    return "ConcreteComponent"
}

type Decorator struct {
    component Component
}
func (d *Decorator) Operation() string {
    return d.component.Operation()
}

type ConcreteDecorator struct {
    Decorator
}

func (d *ConcreteDecorator) Operation() string {
    return d.component.Operation() + "+Decorated"
}
```

#### 9.2 实际应用

```go
// io.Reader包装
reader := bufio.NewReader(file)
reader = zip.NewReader(reader, nil)
reader = gzip.NewReader(reader)
```

#### 9.3 回答模板

> Decorator在不改变原类的情况下动态添加功能。Java的I/O streams、Go的middleware拦截器都用Decorator。AOP是装饰器的应用。

---

### 10. 代理模式 Proxy？

#### 10.1 定义

```go
type Subject interface {
    Request() string
}

type RealSubject struct{}
func (r *RealSubject) Request() string {
    return "RealSubject"
}

type Proxy struct {
    real *RealSubject
}

func (p *Proxy) Request() string {
    // 权限控制
    // 延迟加载
    if p.real == nil {
        p.real = &RealSubject{}
    }
    return p.real.Request()
}
```

#### 10.2 分类

```plaintext
代理类型：
- 远程代理 Remote
- 虚拟代理 Virtual
- 保护代理 Protection
- 智能引用 Smart Reference
- 缓存代理 Cache
```

#### 10.3 回答模板

> 代理模式和Decorator都包装对象，但目的不同：Decorator添功能，Proxy控制访问。远程调用、延迟加载是代理经典应用。

---

### 11. 外观模式 Facade？

#### 11.1 定义

```go
type SubsystemA struct{}
func (s *SubsystemA) Method1() string { return "A1" }
func (s *SubsystemA) Method2() string { return "A2" }

type SubsystemB struct{}
func (s *SubsystemB) Method1() string { return "B1" }


type Facade struct {
    a *SubsystemA
    b *SubsystemB
}

func (f *Facade) Operation() string {
    return f.a.Method1() + f.a.Method2() + f.b.Method1()
}
```

#### 11.2 回答模板

> Facade为子系统提供统一入口，简化调用。JDBC DriverManager是典型的外观。解耦是核心目标。

---

### 12. 组合模式 Composite？

#### 12.1 定义

```go
type Component interface {
    Operation() string
    Add(c Component)
    Remove(c Component)
}

type Leaf struct {
    Name string
}
func (l *Leaf) Operation() string { return l.Name }
func (l *Leaf) Add(c Component) {}

type Composite struct {
    children []Component
    Name    string
}
func (c *Composite) Operation() string {
    result := c.Name + "("
    for _, child := range c.children {
        result += child.Operation() + ","
    }
    return result + ")"
}
func (c *Composite) Add(comp Component) {
    c.children = append(c.children, comp)
}
```

#### 12.2 应用

```
组合模式在：
文件和文件夹
组织架构
UI组件
菜单
```

#### 12.3 回答模板

> Composite将对象组织成树形结构表示。部分-整体层次。文件系统、组织架构、UI组件��是��型应用。

---

### 13. 桥接模式 Bridge？

#### 13.1 定义

```go
// Implementor
type Device interface {
    IsEnabled() bool
    Enable()
    Disable()
    GetVolume() int
    SetVolume(vol int)
}

// Concrete Implementors
type TV struct {
    on  bool
    vol int
}
func (t *TV) IsEnabled() bool { return t.on }
func (t *TV) Enable()        { t.on = true }
func (t *TV) Disable()       { t.on = false }
func (t *TV) GetVolume() int { return t.vol }
func (t *TV) SetVolume(v int) { t.vol = v }

// Abstraction
type RemoteControl struct {
    device Device
}
func (r *RemoteControl) TogglePower() {
    if r.device.IsEnabled() {
        r.device.Disable()
    } else {
        r.device.Enable()
    }
}
```

#### 13.2 回答模板

> Bridge将抽象部分和实现部分分离，使得两者可以独立变化。JDBC是桥接实现、Go的接口也是桥接。

---

## 第四章 行为型模式（高频 ★★★★★）

### 14. 观察者模式 Observer？

#### 14.1 定义

```go
type Observer interface {
    Update(msg string)
}

type Subject struct {
    observers []Observer
}

func (s *Subject) Attach(o Observer) {
    s.observers = append(s.observers, o)
}

func (s *Subject) Notify(msg string) {
    for _, o := range s.observers {
        o.Update(msg)
    }
}
```

#### 14.2 应用

```go
// Event Bus
// Listener
// Message Queue

// 观察者在：
- 事件系统
- MVC（Model-View-Controller）
- 发布订阅
- GUI响应
```

#### 14.3 回答模板

> Observer定义对象间一对多依赖关系，当对象改变时自动通知所有依赖对象。消息队列、EventBus、Listener是常见应用。

---

### 15. 策略模式 Strategy？

#### 15.1 定义

```go
type Strategy interface {
    Execute(data interface{})
}

type ConcreteStrategyA struct{}
func (s *ConcreteStrategyA) Execute(data interface{}) {
    fmt.Println("Strategy A:", data)
}

type Context struct {
    strategy Strategy
}

func (c *Context) SetStrategy(s Strategy) {
    c.strategy = s
}
func (c *Context) Execute(data interface{}) {
    c.strategy.Execute(data)
}
```

#### 15.2 应用

```go
// 支付策略
strategies := map[string]Strategy{
    "alipay": &AlipayStrategy{},
    "wechat": &WechatStrategy{},
}

payment := PaymentContext{strategy: strategies[method]}
payment.pay(amount)
```

#### 15.3 回答模板

> Strategy定义算法家族并相互替换。支付方式、排序算法、 Compression压缩方式是常用场景。减少if-else。

---

### 16. 模板方法模式 Template Method？

#### 16.1 定义

```go
type AbstractClass interface {
    TemplateMethod()
    Step1()
    Step2()
}

type ConcreteClass struct{}

func (c *ConcreteClass) TemplateMethod() {
    c.Step1()
    c.Step2()
}
func (c *ConcreteClass) Step1() {}
func (c *ConcreteClass) Step2() {}
```

#### 16.2 应用

```go
// HTTP处理
type HTTPHandler struct{}

func (h *HTTPHandler) Handle() {
    h.parseRequest()
    h.authenticate()
    h.route()
    h.execute()
    h.logResponse()
}

// parseRequest, authenticate等可被覆盖
```

#### 16.3 回答模板

> Template Method定义算法骨架，将具体步骤延迟到子类。Spring的JDBC、JMS是典型应用。

---

### 17. 命令模式 Command？

#### 17.1 定义

```go
type Command interface {
    Execute()
    Undo()
}

type Light struct{ On bool }

type LightOnCommand struct {
    light *Light
}
func (c *LightOnCommand) Execute() {
    c.light.On = true
}
func (c *LightOnCommand) Undo() {
    c.light.On = false
}

type Invoker struct {
    history []Command
}

func (i *Invoker) Execute(cmd Command) {
    cmd.Execute()
    i.history = append(i.history, cmd)
}
```

#### 17.2 应用

```
命令行、撤销功能：
- 文本编辑器撤销
- 浏览器前进后退
- 数据库事务回滚
- 任务队列
```

#### 17.3 回答模板

> Command将请求封装为对象，便于记录、执行、撤销。MacroCommand复合命令实现批量操作。宏命令是Command应用。

---

### 18. 迭代器模式 Iterator？

#### 18.1 定义

```go
type Iterator interface {
    HasNext() bool
    Next() interface{}
}

type Container interface {
    GetIterator() Iterator
}

type ArrayIterator struct {
    index int
    data  []interface{}
}
func (i *ArrayIterator) HasNext() bool {
    return i.index < len(i.data)
}
func (i *ArrayIterator) Next() interface{} {
    item := i.data[i.index]
    i.index++
    return item
}
```

#### 18.2 标准库应用

```go
range可以遍历切片
for index, element := range slice {}

// map遍历（顺序随机）：
for key, value := range m {}

// channel遍历：
for item := range ch {}

// Go的标准库iterator（Go 1.23+）
for iter := itercollection.Iterator(); iter.Next(); {
    fmt.Println(iter.Value())
}
```

#### 18.3 回答模板

> Iterator提供方法顺序访问集合元素。Java用Iterator，Go用for range。Go 1.23引入标准Iterator接口。

---

### 19. 状态���式 State？

#### 19.1 定义

```go
type State interface {
    Handle(c *Context)
}

type ConcreteStateA struct{}
func (s *ConcreteStateA) Handle(c *Context) {
    fmt.Println("State A handling, switch to B")
    c.state = &ConcreteStateB{}
}

type Context struct {
    state State
}

func (c *Context) Request() {
    c.state.Handle(c)
}
```

#### 19.2 vs Strategy

```
状态模式：状态对象决定行为
Strategy模式：context决定使用哪种策略

状态模式：
- 状态转换自动进行
- 状态子类知道下一个状态

Strategy模式：
- 传入具体策略
- 策略间相互独立
```

#### 19.3 回答模板

> State将对象的行为封装为不同状态，内部状态转换。适合状态机、业务流转换。State和Strategy易混淆，注意区分。

---

### 20. 责任链模式 Chain of Responsibility？

#### 20.1 定义

```go
type Handler interface {
    SetNext(h Handler) Handler
    Handle(req interface{})
}

type BaseHandler struct {
    next Handler
}

func (h *BaseHandler) SetNext(handler Handler) Handler {
    h.next = handler
    return handler
}

func (h *BaseHandler) DefaultHandle(req interface{}) {
    fmt.Println("Default handled")
    if h.next != nil {
        h.next.Handle(req)
    }
}
```

#### 20.2 应用

```
责任链：
- Java Filter
- Netty Pipeline
- Koa/Egg.js middleware
- dubbo Filter
```

#### 20.3 回答模板

> Chain Responsibility将请求沿着链传递，直到被某个处理器处理。过滤器和中间件是典型应用。

---

## 第五章 设计与重构（中高级 ★★★★）

### 21. 为什么需要设计模式？

#### 21.1 核心价值

```
设计模式的价值：
- 提高代码复用
- 减少代码维护成本
- 提高代码可读性和可理解性
- 让系统更灵活易扩展
- 提高团队协作效率
```

#### 21.2 过度设计的警示

```plaintext
过度设计的问题：
- 为未需求预留设计
- 大类拆分过于细分
- 引入不必要的灵活性
- 违反YAGNI（You Aren't Gonna Need It）
- 抽象层次过深
```

#### 21.3 回答模板

> 设计模式解决实际问题，不是银弹。过度设计和不做设计都有问题，简单系统用简单做法。遇到重复代码、难以扩展时再考虑设计模式。

---

### 22. 何时选择设计模式？

#### 22.1 简单指标

```
问题模式 → 解决方案
- 有且只有一个实例 → Singleton单例
- 创建有共同祖先的产品 → 工厂
- 同一接口不同实现 → Strategy策略
- 需要对象逐步构造 → Builder建造者
- 对象有多种状态 → State状态
- 响应式更新 → Observer观察者
```

#### 22.2 何时用

```
- 重构代码时：发现重复代码模式、难以扩展
- 设计代码时：未来可能变化点、复杂业务场景
- Code Review时：发现 antipattern
- 发现 bug时：从问题反推模式
```

#### 22.3 回答模板

> 设计模式在遇到以下两类问题时使用：1）发现系统有重复结构和重复代码；2）需要为未来变化预留灵活性。先写简单实现，有必要时Refactor。

---

### 23. 如何防止过度设计？

#### 23.1 指导原则

```
简单原则：
1. MVP（Minimum Viable Product）- 先做小做简单
2. YAGNI - 不做未来可能不需要的
3. KISS - 保持简单
4. DRY - 不要重复

演进原则：
- 第一次不做
- 第二次容忍（先写代码，未来 Refactor）
- 第三次改进（识别模式后重构）
```

#### 23.2 指标监控

```
代码复杂度指标：
- 行数  L <  200
- 方法圈复杂度 < 10
- 类依赖  < 5
- 继承深度  < 2
```

#### 23.3 回答模板

> 记住KISS和YAGNI。把握适度设计，不要为未来不存在的需求买单。代码腐化后再Refactor是正常的演进���略���

---

### 24. 设计模式 VS 编程范式？

#### 24.1 关系

```
设计模式 vs 编程范式：

OOP面向对象：
- 所有GoF模式都基于OOP

FP函数式：
-  closures = Strategy
-  map/filter/reduce = Iterator
-  Event sourcing = CQRS

DDD领域驱动：
-  Entity = Prototype
-  Repository =  Factory
-  Aggregate = Composite
-  Domain Events = Observer
```

#### 24.2 回答模板

> 许多设计模式在不同编程范式中都有体现，但OOp是主要基础。DDD也借鉴了很多设计模式的概念。

---

### 25. 设计模式与反模式？

#### 25.1 常见反模式

```
Anti-Patterns 反模式
- God Class（上帝类）过大、职责众多
- 多次重复Copy Paste代码
- 强行使用Singleton作为全局变量
- Service Locator 反模式
- Anemia Domain Model（贫血领域模型）
```

#### 25.2 修复代码异味

```
代码异味标志：
- 过长方法
- 过多参数
- 重复代码
- 冗余类
- 循环依赖
```

建议通过Code Review和Linter发现异味。

---

### 26. 23种设计模式速查表

| 模式 | 核心思想 | 适用场景 |
|-------|----------|----------|
| 单例 | 全局唯一的实例 | 全局配置、连接池 |
| 工厂 | 创建对象 | 类复杂创建逻辑 |
| 抽象工厂 | ���列产品创建 | 跨平台UI |
| Builder | 步骤创建 | 对象复杂构造 |
| Prototype | 复制 | 动态克隆 |
| 适配器 | 接口转换 | 整合遗留代码 |
| 代理 | 访问控制 | 延迟加载、权限 |
| 装饰 | 动态添加功能 | 包装类、cache |
| 外观 | 统一入口 | 多层架构 |
| 组合 | 树形结构 | 级联操作 |
| 桥接 | 分离抽象实现 | 多平台 |
| 观察者 | 事件通知 | 消息系统 |
| 策略 | 算法互换 | 业务规则 |
| 命令 | 请求封装 | 撤销、重做 |
| 状态 | 状态转换 | 业务流程 |
| 模板 | 算法框架 | 框架 |
| 迭代器 | 顺序访问 | 集合遍历 |
| 责任链 | 传递处理 | 过滤链 |

---

## 附录：面试追问

1. **项目哪个地方用了什么设计模式？为什么？）
   解释：Spring和主流框架中的典型应用

2. **Singleton有什么问题？）
   答：隐藏依赖、不能抽象、对GC不友好、测试困难

3. **为什么不推荐反射？）
   答：运行期消耗、没有编译检查

4. **Go如何实现设计模式？）
   答：duck typing、functional选项、结构体嵌入

5. **DDD和设计模式？）
   答：Entity/VO/Repostory与设计模式对应

---

## 参考资料

- 《Head First设计模式》
- 《GoF设计模式》
- 《重构改善既有代码的设计》

---

> 整理by Claude Code | 设计模式面试高频100问