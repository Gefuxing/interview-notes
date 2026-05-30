# JVM夺命连环100问——JVM核心技术深度指南

> 本文档面向JVM学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是JVM？为什么要学JVM？

#### 1.1 JVM定义

> JVM（Java Virtual Machine，Java虚拟机）是执行Java字节码的虚拟机，是Java"一次编写，到处运行"的核心。它不仅是Java语言的运行容器，也是Android、Kotlin、Scala等语言的运行平台。

```plaintext
JVM架构：
┌─────────────────────────────────────────┐
│           Java源程序                    │
├─────────────────────────────────────────┤
│         javac 编译器                    │
├─────────────────────────────────────────┤
│         .class 字节码                   │
├─────────────────────────────────────────┤
│          JVM虚拟机                     │
├─────────────────────────────────────────┤
│    即时编译器(JIT) / 解释器              │
├─────────────────────────────────────────┤
│          本地机器码                     │
└─────────────────────────────────────────┘
```

#### 1.2 JVM核心作用

```plaintext
内存管理：
- 自动内存分配
- 垃圾回收
- 避免内存泄漏

平台无关：
- Write Once, Run Anywhere
- 字节码 anywhere 运行

安全沙箱：
- 权限控制
- 代码沙箱
```

#### 1.3 回答模板

> JVM是Java虚拟机，执行Java字节码实现"一次编译，到处运行"。它负责内存管理（自动分配和GC）、字节码执行（JIT编译优化）、平台无关性。是后端开发理解性能问题、排错优化的基础。

---

### 2. JVM内存结构？

#### 2.1 JVM内存区域

```plaintext
运行时数据区：
┌────────────────────────────────────┐
│          方法区（Metaspace）          │ ← 类信息、常量、静态变量
├────────────────────────────────────┤
│              Java堆（Heap）          │ ← 对象实例、数组（GC主要区域）
├────────────────────────────────────┤
│           虚拟机栈（VM Stack）       │ ← 方法调用、局部变量
├────────────────────────────────────┤
│           本地方法栈（Native）        │ ← Native方法调用
├────────────────────────────────────┤
│           PC寄存器（Program Counter）  │ ← 当前字节码行号
├────────────────────────────────────┤
│           执行引擎                   │ ← 解释器+JIT
└───────────────────────────────��────┘
```

#### 2.2 各区域作用

| 区域 | 线程私有 | 作用 |
|------|----------|------|
| PC寄存器 | 是 | 记录执行位置 |
| 虚拟机栈 | 是 | 方法帧栈 |
| 本地方法栈 | 是 | Native方法 |
| 堆 | 否 | 对象实例 |
| 方法区 | 否 | 类元数据 |

#### 2.3 回答模板

> JVM内存分为线程私有（PC寄存器、虚拟机栈、本地方法栈）和线程共享（堆、方法区）。堆是GC主要区域，存放对象实例；方法区存类信息、常量、静态变量。Java 8后方法区改为Metaspace用本地内存。

---

### 3. 什么是字节码？

#### 3.1 字节码定义

> 字节码（Bytecode）是JVM执行的中间二进制格式，由Java编译器生成。每条指令是1字节（0-255），所以叫字节码。

```plaintext
字节码示例（Hello World）：
// Java源码
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}

// 关键字节码：
iconst_1      // 常数1入栈
istore_1      // 存入局部变量1
getstatic #2  // 获取静态字段
iload_1       // 加载局部变量
invokevirtual #3// 调用方法
return       // 返回
```

#### 3.2 字节码指令分类

```plaintext
加载存储：iload, istore, aload, astore
操作数栈：dup, swap, pop
控制转移：ifeq, ifne, goto
方法调用：invokevirtual, invokespecial, invokestatic
对象创建：new, putsfield, getfield
```

#### 3.3 回答模板

> 字节码是JVM执行的二进制指令格式，由javac生成。每条指令1字节共约200种。可以用javap -c查看。通过字节码可理解底层执行、分析性能问题、学习各种语法糖的实现原理。

---

### 4. 类加载器？

#### 4.1 类加载器层级

```plaintext
JVM类加载器：
┌─────────────────────────────────────┐
│ Bootstrap ClassLoader (C++)          │ ← 加载JAVA_HOME/jre/lib
├─────────────────────────────────────┤
│ Extension ClassLoader                │ ← 加载JAVA_HOME/jre/ext
├─────────────────────────────────────┤
│ Application ClassLoader            │ ← 加载classpath
├─────────────────────────────────────┤
│ 自定义ClassLoader                   │
└─────────────────────────────────────┘
```

#### 4.2 双亲委派模型

```java
// ClassLoader源码简化
protected Class<?> loadClass(String name, boolean resolve) {
    // 首先委派父类加载
    Class<?> c = parent.loadClass(name, false);
    if (c == null) {
        // 父类无法加载，自己尝试
        c = findClass(name);
    }
    return c;
}
```

#### 4.3 自定义类加载器

```java
// 自定义类加载器
public class MyClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String path = name.replace('.', '/') + ".class";
        byte[] bytes = Files.readAllBytes(Paths.get(path));
        return defineClass(name, bytes, 0, bytes.length);
    }
}
```

#### 4.4 回答模板

> JVM有启动类加载器、扩展类加载器、应用类加载器三层。双亲委派是父类优先加载，确保类的唯一性和安全性。自定义类加载器可以打破双亲、实现热部署、模块化加载等高级功能。

---

### 5. 类加载过程？

#### 5.1 加载三阶段

```plaintext
1. 加载（Loading）
   → 通过类的全限定名获取二进制字节流
   → 转化为方法区的运行时数据结构
   → 生成Class对��

2. 链接（Linking）
   → 验证：格式验证、语义验证、字节码验证
   → 准备：为类变量分配内存、设置初始值
   → 解析：符号引用→直接引用

3. 初始化（Initialization）
   → <clinit>()方法执行
   → 静态赋值语句和静态块执行
   → 父类先于子类
```

#### 5.2 触发初始化

```java
// 主动引用（触发初始化）
new Object();
Class.forName("Test");
调用静态方法或静态字段（非final修饰）
```

```java
// 被动引用（不触发）
- 访问静态final常量
- 子类访问父类静态字段
- 数组new Object[10]
```

#### 5.3 回答模板

> 类加载分加载、链接、初始化三阶段。链接包括验证、准备（分配内存）、解析（符号引用转直接引用）；初始化执行<clinit>方法。被动引用不会触发初始化如访问final常量、子类访问父类static。

---

### 6. 什么是反射？

#### 6.1 反射定义

> 反射（Reflection）是Java在运行时动态获取类信息和调用对象方法的能力。

```java
// 获取Class对象
Class<?> clazz = Class.forName("com.test.User");
Class<?> clazz = User.class;
Class<?> clazz = user.getClass();

// 创建实例
Object obj = clazz.newInstance(); // 已废弃

// 获取方法
Method method = clazz.getMethod("doSomething", String.class);
method.invoke(obj, "param");

// 获取字段
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);
field.set(obj, "value");
```

#### 6.2 反射代价

```plaintext
反射的性能代价（约3-20倍慢）：
- 运行时解析
- 无法JIT优化
- Security Manager检查
- 暴力访问accessible=true

优化手段：
- 缓存Class/Method对象
- 调用次数少用反射
- java.lang.reflect.Proxy或cglib
```

#### 6.3 回答模板

> 反射是运行时获取类信息、调用方法的机制，用于Spring Boot、ORM框架等。反射可以绕过泛型检查（但仍存于字节码）、访问private成员。反射有性能代价，约比正常调用慢3-20倍。Spring大量使用反射所以启动慢。

---

### 7. 什么是泛型擦除？

#### 7.1 类型擦除

```java
// 源代码
List<String> list = new ArrayList<>();
list.add("hello");
// 编译后字节码实际是：
List list = new ArrayList();
list.add("hello");
// 长城bridge方法兼容：
public Object get(int i) { return (String) list[i]; }
```

```plaintext
Java泛型是编译期检查
运行时擦除为Object或上限

擦除规则：
- 有上限：<T extends Number> → Number
- 无上限：<T> → Object
- 通配符：<? extends Number> → Number
```

#### 7.2 Bridge方法

```java
// 源码
class Node<T> {
    public T get() { return null; }
}
class StringNode extends Node<String> {
    public String get() { return "s"; }
}

// 编译后实际生成桥接方法：
class StringNode extends Node {
    public String get() { return "s"; }
    public Object get() { return (Object) get(); } // 桥接方法
}
```

#### 7.3 回答模板

> Java泛型是编译期检查，运行时擦除为Object或上限类型。这是兼容Java 1.5之前代码的历史选择。好处是兼容旧代码，代价是无法泛型信息运行时获取、无法创建泛型数组。Bridge方法是用来保持类型兼容的。

---

### 8. JDK vs JRE vs JVM？

#### 8.1 区别

```plaintext
JVM（Java Virtual Machine）
└── 运行Java字节码的核心组件

JRE（Java Runtime Environment）
└── JVM + 核心类库（lib/rt.jar等）
└── 不含javac编译器

JDK（Java Development Kit）
└── JRE + 开发工具（javac、jdb、jar等）
└── 含完整开发环境
```

#### 8.2 安装目录结构

```plaintext
JDK安装目录：
├── bin/        # javac��java���jar等
├── jre/        # JRE
├── lib/        # 工具类库、源码
├── include/    # Native头文件
└── ...
```

#### 8.3 回答模板

> JVM是运行字节码的虚拟机；JRE是运行环境=JVM+核心类库；JDK是开发环境=JRE+javac等开发工具。开发需要JDK，生产运行只需要JRE。Docker镜像通常只装JRE减小体积（约减去一半）。

---

### 9. 浅拷贝和深拷贝？

#### 9.1 拷贝概念

```java
// 浅拷贝（Shallow Copy）
// 复制引用
Object clone() throws CloneNotSupportedException {
    Object obj = this.clone(); // Object默认是浅拷贝
    return obj;
}

// 深拷贝（Deep Copy）
// 复制对象及引用的所有对象
Object deepClone() throws Exception {
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(bos);
    oos.writeObject(this);
    ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
    ObjectInputStream ois = new ObjectInputStream(bis);
    return ois.readObject();
}
```

#### 9.2 实现方式

```java
// Cloneable
class Person implements Cloneable {
    private String name;
    private Date birthday;

    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            person.birthday = (Date) birthday.clone(); // 深拷贝
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}
```

#### 9.3 回答模板

> 浅拷贝只复制引用（地址），对象还是同一份；深拷贝递归复制所有引用对象。要正确实现深拷贝需要递归clone所有引用成员，或用序列化。集合的浅拷贝指元素引用相同，深拷贝需逐元素深拷贝。

---

### 10. String/StringBuilder/StringBuffer？

#### 10.1 区别

| 特性 | String | StringBuilder | StringBuffer |
|------|--------|--------------|-------------|
| 线程安全 | 不可变 | 不安全 | 安全 |
| 性能 | 低（每次新对象） | 高 | 中（有synchronize） |
| 存储 | char[] final | char[] | char[] synchronized |

#### 10.2 String常量池

```java
String s1 = "hello";       // 常量池
String s2 = "hello";       // 常量池同一对象
String s3 = new String("hello"); // 堆对象
s1 == s2  // true
s1 == s3   // false
```

```java
// intern方法
String s = new String("hello").intern(); // 放入常量池
// 常量池有则返回已有的引用
```

#### 10.3 回答模板

> String是不可变对象，每次操作产生新String所以性能低，有常量池优化相同字面量；StringBuilder是非线程安全的高性能拼接；StringBuffer是线程安全的有synchronize但性能低。Java 9后String内部用byte[]更省内存。

---

## 第二章 垃圾回收篇（高频 ★★★★★）

### 11. 什么是垃圾回收？

#### 11.1 GC定义

> 垃圾回收（Garbage Collection，GC）是JVM自动回收不再使用的对象释放内存的机制，是Java内存管理区别于C++手动管理的重要特征。

```plaintext
GC解决的问题：
- 内存分配
- 内存回收
- 避免内存泄漏
- 悬挂引用（野指针）
```

#### 11.2 GC算法演进

```plaintext
1960: 引用计数（Reference Counting）
  - 简单但无法处理循环引用
1984: 标记-清除（Mark-Sweep）
  - Stop-the-world
1988: 标记-压缩（Mark-Compact）
  - 解决碎片问题但时间长
1990: 复制算法（Copying）
  - 存活对象少时高效（Young区）
2001: 分代算法（Generational）
  - 综合利用各种算法当今主流
```

#### 11.3 回答模板

> 垃圾回收是JVM自动回收无用对象释放内存的机制，避免手动free的野指针问题。基本算法有引用计数（已被淘汰）、标记清除、标记压缩、复制、分代等。当今主流是分代算法+各种GC组合。

---

### 12. 如何判断对象可回收？

#### 12.1 引用计数法

```plaintext
引用计数（Reference Counting）：
- 对象每被引用+1计数
- 引用失效-1计数
- 计数为0可回收
- 无法处理循环引用！
// A引用B，B引用A，计数永远不为0
```

#### 12.2 可达性分析

```java
// GC Roots
public class GCRoot {
    static GCRoot staticObj;      // 静态变量
    final static GCRoot finalObj;   // final static
    GCRoot methodObj;             // 方法中对象

    public static void main(String[] args) {
        // 栈中对象引用
        GCRoot localObj = new GCRoot();

        // localObj是Root但引用的对象
        // 如果不再被Roots引用，可回收
    }
}
```

```plaintext
GC Roots包括：
- 虚拟机栈引用对象
- 方法区static属性对象
- 方法区final属性对象
- 本地方法栈JNI引用对象
- 同步锁等待的对象
- JVM内部对象
```

#### 12.3 回答模板

> 判断对象可回收有两种方法：1）引用计数但无法处理循环引用已淘汰；2）可达性分析从GC Roots向下遍历，能到达的就是存活。GC Roots包括栈中对象、static/final常量、锁对象等。

---

### 13. 什么是Minor GC��Major GC？

#### 13.1 分代假说

```plaintext
Generational Hypothesis（分代假说）：
- 大多数对象朝生夕灭
- 熬过越多次GC越难死
- 新生对象在Young区
- 老对象在Old区
```

#### 13.2 分代垃圾回收

```plaintext
Young区（Minor GC）：
- 标记-复制算法
- 存活对象少，效率高
- STW时间短
- 对象晋升阈值：age>15（默认）

Old区（Major/Full GC）：
- 标记-压缩/标记-清除
- 对象多，时间长
- 比Minor GC慢10倍以上
```

#### 13.3 回答模板

> 基于分代假说，新生成对象在Young区用Minor GC快速回收（标记-复制），老对象用Major/Full GC。对象年龄>15晋升Old区。Full GC比Minor慢10倍以上，优化要尽量让对象在Young区消亡减少Full GC频率。

---

### 14. GC收集器有哪些？

#### 14.1 收集器对比

| 收集器 | 区域 | 算法 | 特点 |
|--------|------|------|------|
| Serial | Young | 复制 | 单线程stop-the-world |
| ParNew | Young | 复制 | Serial多线程版 |
| Parallel Scavenge | Young | 复制 | 吞吐量优先 |
| Serial Old | Old | 标记-压缩 | 单线程 |
| CMS | Old | 标记-清除 | 并发、低停顿 |
| G1 | Mixed | 复制+标记 | 并发、可预测停顿 |
| ZGC | All | 着色指针 | 并发TB级 |
| Shenandoah | All | 转发指针 | 并发TB级 |
| Epsilon | 无GC | 空转 | 测试用 |

#### 14.2 GC组合

```plaintext
常见组合：
Serial + Serial Old（-client）
ParNew + CMS（早期web）
Parallel Scavenge + Parallel Old（吞吐量优先）
G1（现在默认）
ZGC/Shenandoah（大内存低延迟）
```

```bash
# 指定GC
-XX:+UseSerialGC
-XX:+UseParNewGC
-XX:+UseParallelGC
-XX:+UseConcMarkSweepGC
-XX:+UseG1GC
-XX:+UseZGC
```

#### 14.3 回答模板

> GC收集器有Serial（单线程）、ParNew（多线程）、Parallel（吞吐量优先）、CMS（并发低停顿）、G1（目前默认）、ZGC/Shanandoah（TB级低延迟）等。不同收集器适合不同场景，G1是目前默认的收集器。

---

### 15. 什么是STW？

#### 15.1 Stop-The-World

```plaintext
Stop-The-World（STW）：
- GC时暂停所有应用线程
- 只GC线程运行
- 由分代/算法选择

STW原因：
- 标记需要全遍历
- 引用可能在变需要安全点
- 复制需要地址一致
```

#### 15.2 安全点和安全区

```plaintext
Safe Point（安全点）：
- 代码指令序列的特定点
- 可以暂停GC
- 方法调用、循环、异常等

Safe Region（安全区）：
- 一段代码区域
- 区域内引用不变
- 可以GC
```

#### 15.3 回答模板

> STW是GC时暂停所有应用线程只GC线程运行的现象，几乎所有GC都有STW只是时间长短。安全点是代码中能安全停GC的指定点，循环末尾、方法返回处等都是安全点。

---

### 16. CMS收集器？

#### 16.1 CMS工作流程

```plaintext
CMS（Concurrent Mark Sweep）：
1. 初始标记（Initial Mark）
   → STW，标记GC Roots直接引用的对象

2. 并发标记（Concurrent Mark）
   → 用户线程运行，并发遍历引用链

3. 重新标记（Remark）
   → STW，修正并发标记期间变化
   → 标记增量更新

4. 并发清除（Concurrent Sweep）
   → 用户线程运行，清除垃圾
```

#### 16.2 CMS问题

```plaintext
1. CPU敏感
   - 并发占用CPU
   - 线程数影响

2. 浮动垃圾
   - 并发期间产生的垃圾本次不清理
   - 下次GC

3. 内存碎片
   - 标记-清除有碎片
   - -XX:CMSInitiatingOccupancyFraction触发full gc
```

#### 16.3 回答模板

> CMS是并发低停顿收集器，工作流程：初始标记→并发标记→重新标记→并发清除。有三个问题：CPU敏感、浮动垃圾、内存碎片。需要定期Full GC整理碎片。

---

### 17. G1收集器？

#### 17.1 G1特���

```plaintext
G1（Garbage First）：
- 面向服务端的GC
- 标记-复制+标记-压缩
- 可预测停顿时间
- Region分区管理
- Remembered Set记忆集
```

```plaintext
与CMS区别：
- 分代：G1仍然分代但不连续
- 内存：G1用Region
- 并发：小young+concurrent mark
- 可预测停顿：-XX:MaxGCPauseMillis=200
```

#### 17.2 工作流程

```plaintext
G1流程：
1. 年轻代收集（Young GC）
   → 收集年轻代Region
   → 复制到Survivor

2. 混合收集（Mixed GC）
   → 年轻代 + 部分老年代
   → -XX:G1MixedGCLiveThresholdPercent

3. Full GC（后备）
   → 转移失败触发
   → 单线程
```

#### 17.3 回答模板

> G1是JDK 9后的默认GC，用Region分区管理，把堆划分为多个相等大小Region。可通过MaxGCPauseMillis设定停顿目标自适应。是面向服务端低延迟的GC，配合-XX:MaxGCPauseMillis使用。

---

### 18.ZGC和Shenandoah？

#### 18.1 ZGC

```plaintext
ZGC（JDK 11+）：
- 并发TB级（TB以上内存）
- 利用着色指针（Colored Pointers）
- 染色位记录对象GC状态
- 停顿时间<10ms
- 不分代但保留了分代思想
```

```bash
# 启用ZGC
-XX:+UseZGC -XX:ConcGCThreads=4
```

#### 18.2 Shenandoah

```plaintext
Shenandoah（JDK 12+）：
- 类似ZGC但不用着色指针
- 用转发指针（Forwarding Pointer）
- JDK 8也有Backport版
- 停顿时间<1ms

区别：
- ZGC需要JDK版本
- Shenanogan对JDK版本要求宽松
- 技术细节略有不同
```

#### 18.3 回答模板

> ZGC和Shenandoah是TB级以上内存的低延迟GC，停顿时间毫秒甚至亚毫秒级。ZGC用着色指针需要JDK 11+，ShenandoahJDK 12+或JDK 8的Backport版。大内存低延迟场景考虑这两个GC。

---

### 19. 对象头包含什么？

#### 19.1 对象头结构

```plaintext
32位JVM：
┌────────────┬──────────┬────────────┐
│  Mark Word │  Klass   │ 数组长度  │
└────────────┴──────────┴────────────┘
◄── 32bits ──►◄── 32bits ──►◄─32bits─►

64位JVM：
┌────────────────┬────────────────┬────────────┐
│    Mark Word   │    Klass      │  数组长度 │
└────────────────┴────────────────┴────────────┘
◄────── 64bits ────────►◄── 64bits ──►◄64bits─►
```

#### 19.2 Mark Word内容

```plaintext
Mark Word内容（64位）：
1. 未锁定：
   hash:31bits + age:4bits + biased:1bit + lock:2bits

2. 偏向锁：
   thread:54bits + epoch:2bits + age:4bits + bias:1bit + lock:2bits

3. 轻量锁：
   pointer to lock record:62bits + lock:2bits

4. 重量锁：
   pointer to monitor:62bits + lock:2bits

5. GC标记：
   hash:31bits + age:4bits + lock:2bits
```

#### 19.3 回答模板

> 对象头包含Mark Word（锁状态、hash、GC分代年龄）、类型指针（指向Klass元数据）、数组长度（有数组时）。Mark Word存储锁状态信息，偏向锁/轻量锁/重量锁信息在这里。

---

### 20. 什么是偏向锁？

#### 20.1 偏向锁原理

```plaintext
偏向锁（Biased Lock）：
- 首次获取锁时CAS将线程ID写入Mark
- 之后该线程进入同步块无需任何同步
- 撤销需要STW

优点：无开销
缺点：有竞争时需要撤销
```

#### 20.2 偏向锁膨胀

```plaintext
偏向锁→轻量锁膨胀：
1. 撤销偏向锁
2. 在当前线程栈创建Lock Record
3. CAS尝试把Mark改成Lock Record
4. 成功获锁，失败则自旋
```

#### 20.3 关闭偏向锁

```bash
# 关闭偏向锁（高并发建议）
-XX:-UseBiasedLocking

# JDK 15后默认关闭
```

#### 20.4 回答模板

> 偏向锁是让首个获取锁的线程以后进入同步块无需同步的开销，轻量锁是自旋锁。偏向锁撤销需要STW，所以高并发场景经常关闭。JDK 15后默认禁用。

---

## 第三章 性能调优篇（高频 ★★★★★）

### 21. 常见JVM参数？

#### 21.1 堆内存参数

```bash
# 初始和最大
-Xms512m           # 初始堆
-Xmx2048m          # 最大堆
-Xmn256m           # Young区大小

# New区比例
-XX:NewRatio=2     # Old/New=2
-XX:SurvivorRatio=8 # Eden/Survivor=8
```

#### 21.2 GC参数

```bash
# 选择GC
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseConcMarkSweepGC
-XX:+UseG1GC
-XX:+UseZGC

# 打印
-XX:+PrintGCDetails
-Xloggc:gc.log

# Full GC条件
-XX:+UseAdaptiveSizePolicy
-XX:CMSInitiatingOccupancyFraction=80
```

#### 21.3 其他参数

```bash
# 方法区
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m

# 线程栈
-Xss1m             # 1MB栈

# JIT
-XX:CompileThreshold=10000
-XX:ReservedCodeCacheSize=240m
```

#### 21.4 回答模板

> JVM参数分几类：堆内存（-Xms/-Xmx/-Xmn）、GC选择（-XX:+UseG1GC等）、打印日志（-XX:+PrintGCDetails）、方法区（-XX:MetaspaceSize）。生产环境要合理设置堆大小和GC参数。

---

### 22. 如何排查线上问题？

#### 22.1 CPU 100%

```bash
# top查看进程
top -p <pid>

# jstack查看线程
jstack <pid> | head -50

# 按CPU排序的线程
top -Hp <pid>
printf "%x\n" <tid>
jstack <pid> | grep <thread hex>
```

#### 22.2 内存问题

```bash
# 堆内存
jmap -heap <pid>
jmap -histo <pid> | head -20
jmap -dump:format=b,file=heap.hprof <pid>

# 查看对象
jcmd <pid> GC.class_histogram
```

#### 22.3 GC问题

```bash
# 查看GC日志
jstat -gcutil <pid> 1000

# Jang Profiler
async-profiler
Arthas
```

#### 22.4 回答模板

> 线上问题用top找进程、jstack dump线程堆栈（-Hp转线程视图）、jmap查堆（-heap看概况、-histo看对象、-dump生成dump）。Arthas/Alibaba开源的arthas是排查利器。

---

### 23. jstat怎么用？

#### 23.1 jstat命令

```bash
# GC统计
jstat -gcutil <pid> 1000

# 类加载统计
jstat -class <pid>

# JIT编译统计
jstat -compiler <pid>

# 查看GC原因
jstat -gccapacity <pid>
```

#### 23.2 输出说明

```plaintext
S0C S1C S0U S1U EC EU OC OU MC MU YGC YGCT FGC FGCT CGC CGCT GCT
12096.0 12096.0 0.0 0.0 98304.0 48223.0 696320.0 0.0 44800.0 43092.0 3 0.037 0 0.000 0 0.000 37

S0C：Survivor0容量
S0U：Survivor0使用
EC：Eden容量
EU：Eden使用
OC：Old容量
OU：Old使用
YGC：年轻代GC次数
YGCT：年轻代GC时间
FGC：Full GC次数
FGCT：Full GC时间
```

#### 23.3 回答模板

> jstat -gcutil 1000 每秒输出GC统计，S0/S1/Eden/Old使用情况，YGC/FGC次数和时间。关注FGC是否频繁（应该很少）、OGC使用是否正常（>80%有风险）。JVM专业面试考点。

---

### 24. jmap怎么用？

#### 24.1 jmap命令

```bash
# 堆摘要
jmap -heap <pid>

# 直方图
jmap -histo <pid> | head -20

# 生成dump
jmap -dump:format=b,file=heap.hprof <pid>

# finalize队列
jmap -finalizerinfo <pid>
```

#### 24.2 MAT分析dump

```plaintext
Memory Analyzer分析：
- 1. File → Open Heap Dump
- 2. Leak Suspects Report
- 3. Top Components查看大对象
- 4. OQL查询

常用OQL：
SELECT * FROM String s WHERE s.@retainedHeapSize > 1000
SELECT * FROM char[] c WHERE c.@length > 1000
```

#### 24.3 回答模板

> jmap -heap看堆概要、-histo看对象数量和内存占用、-dump生成dump用MAT分析OOM原因。MAT可以分析内存泄漏suspect、出 Retained Heap大的对象。

---

### 25. 什么是卡表（Card Table）？

#### 25.1 Card Table作用

```plaintext
Card Table是用于Minor GC优化minor区引用的数据结构。

为什么需要？
- Young GC要扫描Old区对Young的引用
- 直接扫描整个Old区太慢

方案：
- Card Table把Old区分成512字节块
- 只需扫描dirty块
```

#### 25.2 工作原理

```plaintext
Dirty机制：
1. 引用变化时（在putfield等）
   → 把card标记为dirty

2. Minor GC时
   → 只扫描dirty cards
   → 其他的不用扫

写屏障（Write Barrier）：
- store指令后执行
- 改变引用时设置card dirty
```

#### 25.3 回答模板

> Card Table是Minor GC的优化，用512字节卡片记录Old区引用变化，只需扫描dirty卡片不用扫全Old区。写屏障在引用改变时设置dirty。Remembered Set是G1的概念类似但更精细。

---

### 26. Metaspace是什么？

#### 26.1 Metaspace定义

```java
// Java 8之前：PermGen
// -XX:PermSize=256m -XX:MaxPermSize=256m

// Java 8之后：Metaspace
// -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m
```

```plaintext
Metaspace vs PermGen：
- 位置：Metaspace用本地内存
- 管理：Metaspace按类需分配
- 压缩：Metaspace不需要压缩
- ���动���长：可以auto increase
```

#### 26.2 Metaspace参数

```bash
# 大小
-XX:MetaspaceSize=256m        # 初始
-XX:MaxMetaspaceSize=512m    # 最大

# 压缩类指针（JDK 8默认）
-XX:+UseCompressedClassMeta

# 类空间
-XX:CompressedClassSpaceSize=256m
```

#### 26.3 回答模板

> Java 8把PermGen替换为Metaspace用本地内存，不再有溢出问题。MetaspaceSize是触发Full GC的阈值，设太小会不停Full GC。元空间存储类的元信息（类名、方法信息等）和类加载器字典。

---

### 27. JIT编译器？

#### 27.1 JIT定义

> JIT（Just-In-Time）编译器是JVM把热点字节码编译为本地机器码的即时编译器，是Java快于Python/JS的关键。

```plaintext
JIT编译触发：
- 方法调用计数器
- 回边计数器（循环体）

热点检测：
- 方法调用>10000次
- 循环体>10000次
```

#### 27.2 分层编译

```plaintext
Tiered Compilation：
- Tier 0：解释执行
- Tier 1：C1轻量编译
- Tier 2：C1完整编译
- Tier 3：C2Server编译（最终优化）

# 启用分层
-XX:+TieredCompilation
```

#### 27.3 回答模板

> JIT把热点代码编译成本地机器码加速执行。分层编译有Tier 1解释→Tier 2C1→Tier 3C2Server渐进优化，C2做激进优化（内联、分支预测等）生成高质量代码。-XX:+TieredCompilation启用。

---

### 28. 内联优化？

#### 28.1 方法内联

```java
// 内联前
int add(int a, int b) { return a + b; }
int compute() { return add(1, 2); }

// 内联后
int compute() { return 1 + 2; }
```

为什么内联：
- 消除调用开销（栈帧创建、参数传递）
- 解开优化空间

内联条件：
- 方法体不太大
- 非虚方法（或只有一个实现
```

#### 28.2 虚调用内联

```java
// 虚方法不能直接内联
// 但可以 speculatively inline
if (obj.getClass() == Dog.class) {
    // 假设是Dog，当狗来内联编译
    // 如果假设错了deoptimize
}
```

#### 28.3 其他优化

```plaintext
JIT激进优化：
- 方法内联
- 常量折叠
- 公共子表达式消除
- 循环展开
- 向量化
- 锁 elision
```

#### 28.4 回答模板

> 内联是JIT最重要的优化，消除方法调用开销。内联条件是热点+小型方法。虚方法可以speculatively内联（假设类型，错了deopt）。-XX:PrintInlining可看内联日志。

---

### 29. 逃逸分析？

#### 29.1 逃逸分析

```java
// 逃逸：对象传出去
return sb.toString(); // StringBuilder逃逸了

// 不逃逸：只在方法内用
StringBuilder sb = new StringBuilder();
sb.append("a");
sb.append("b");
return sb.toString();
```

```plaintext
分析对象是否逃逸：
- 逃逸到方法外 → 堆分配
- 不逃逸 → 栈分配/标量分解

标量分解：
- 不用对象，用其fields分别存栈
```

#### 29.2 逃逸分析优化

```plaintext
1. 栈分配（Stack Allocation）
   - 不逃逸→栈上分配
   - 方法结束自动回收，无GC

2. 同步消除（Lock Elision）
   - 不逃逸到别的线程
   - 锁可以消除

3. 标量替换（Scalar Replacement）
   - 对象解开存fields
   - 用寄存器
```

#### 29.3 回答模板

> 逃逸分析判断对象是否会逃逸到方法外。不逃逸可以做：栈上分配（无GC）、同步消除、标量替换（用寄存器不用堆）。-XX:+PrintEscapeAnalysis看分析结果。

---

### 30. 对象内存布局？

#### 30.1 对象结构

```plaintext
对象在堆的布局：
┌──────────────────────────────────────┐
│ 对象头（Object Header）                │
│  MarkWord(64bits) | Klass*(32bit) | len│
├────────────────────────────���─���───────┤
│ 实例数据（Instance Data）              │
│  fields: parent_first, child_last        │
│  按继承顺序、从大到小排              │
├──────────────────────────────────────┤
│ 对齐填充（Padding）                  │
│  8字节对齐                          │
└──────────────────────────────────────┘
```

#### 30.2 对齐规则

```plaintext
Fields对齐规则：
- long, double: 8字节起始
- int, float: 4字节起始
- short, char: 2字节起始
- byte, boolean: 位置无限制

Fields排列优化：
- 相同宽度放一起减少padding
- 把子类 wide fields放父类前面
```

#### 30.3 回答模板

> 对象在堆的内存布局：对象头（Mark/Klass/len）+ 实例数据（继承顺序、字段大小分配合并减少padding）+ padding对齐8字节。把字段按大小排列优化内存。

---

## 第四章 内存模型篇（高频 ★★★★★）

### 31. JVM内存模型（JMM）？

#### 31.1 JMM定义

> JMM（Java Memory Model）是Java定义的多线程内存访问规范，解决可见性、原子性、有序性问题。

```plaintext
JMM：
┌─────────────────────────────────────────────┐
│                  Main Memory                │
│  (主内存，所有线程共享，含堆和方法区)      │
├─────────────────────────────────────────────┤
│              每个线程的本地内存              │
│       (本地内存，线程私有，含工作缓存)       │
└─────────────────────────────────────────────┘

线程A写 → 刷新到主内存 → 线程B读取
```

#### 31.2 三大问题

```plaintext
1. 可见性（Visibility）
  - 线程1改了，线程2看不到
  - 原因：各自缓存

2. 原子性（Atomicity）
  - i++不是原子的
  - 读-改-写三步

3. 有序性（Ordering）
  - 代码乱序执行
  - 编译器/CPU优化
```

#### 31.3 回答模板

> JMM是Java的内存模型，规定线程有本地内存（工作缓存），共享数据在主内存。解决三大问题：可见性（缓存不一致）、原子性（i++三步）、有序性（重排）。volatile保证可见+有序，synchronize保证原子。

---

### 32. volatile关键字？

#### 32.1 volatile作用

```java
volatile boolean flag = true;
```

```plaintext
volatile作用：
1. 可见性
   - 写入立即flush到主内存
   - 读取立即从主内存读取
   - 缓存失效

2. 有序性
   - volatile前后代码不能重排
   - 内存屏障
```

#### 32.2 内存屏障

```plaintext
StoreStore屏障：   禁止Store前重排
LoadLoad屏障：    禁止Load后重排
StoreLoad屏障： 最常用，禁止跨屏障
LoadStore屏障：
```

```c
// hotspot汇编
lock addl $0, (%esp)  // 实际上插入 LOCK前缀
// 作用：store buffer刷出+CPU缓存失效
```

#### 32.3 回答模板

> volatile保证可见性和有序性，不保证原子性。原理是插入LOCK指令使缓存失效+内存屏障。适合一写多读场景如flag、做双重检查锁定DCL。

---

### 33.happens-before原则？

#### 33.1 happens-before定义

> happens-before是JMM定义的偏序关系，表示前一个操作对后一个操作可见且不重排。

```java
// 程序顺序规则
a = 1; hb(a = 1, b = 2);
b = 2;

// 锁规则
synchronized(obj) { a = 1; } hb(a = 1, unlock);
synchronized(obj) { b = 2; } hb(lock, b = 2);
```

```plaintext
常见规则��
- 程序顺序：同一线程内
- 锁规则：unlock后lock前
- volatile：写-读
- start规则：start()-run()
- join规则：join()-结束
- 传递性：A hb B,B hb C => A hb C
```

#### 33.2 应用

```java
// DCL双重检查锁定
class Singleton {
    private static volatile Instance instance;

    public static Instance get() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Instance();
                }
            }
        }
        return instance;
    }
}
// volatile阻止重排：instance = new Instance();
// 不会变成：读取未初始化的对象
```

#### 33.3 回答模板

> happens-before是JMM定义的保证可见性的偏序规则。volatile变量write后read的其他线程看到更新值。DCL用volatile防止指令重排返回未初始化对象。需满足传递性。

---

### 34. synchronized原理？

#### 34.1 synchronized用法

```java
// 方法
synchronized void method() { }

// 代码块
synchronized(obj) { }

// 静态方法（类锁）
synchronized static void method() { }
```

#### 34.2 原理分析

```c
// 编译后字节码
monitorenter:
//   获取对象头mark
//   CAS attempts lock record
//   失败则自旋/阻塞

monitorexit:
//   释放lock
//   unpark阻塞线程

wait():
//   释放锁并进入wait pool
notify()/notifyAll():
//   唤醒一个/所有wait线程
```

#### 34.3 锁优化

```plaintext
锁优化（锁膨胀方向）：
1. 无锁
2. 偏向锁（first thread）
3. 轻量锁（自旋CAS）
4. 重量锁（OS mutex）

锁消除：
- JIT发现不逃逸对象
- 移除synchronized
```

#### 34.4 回答模板

> synchronized是JVM内置的互斥锁，基于对象头Monitor实现。有偏向锁→轻量锁→重量锁的膨胀机制。-XX:+UseBiasedLocking启用。锁消除在逃逸分析发现不逃逸时移除锁。

---

### 35. ReentrantLock？

#### 35.1 ReentrantLock对比

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

| 特性 | synchronized | ReentrantLock |
|------|-------------|--------------|
| 语法 | 关键字 | API |
| 锁类型 | 非公平 | 公平/非公平 |
| 条件变量 | wait/notify | Condition×N |
| 响应中断 | 不可 | lockInterruptibly |
| 尝试获取 | 不可 | tryLock |
| 性能 | 好 | 好(JDK6后) |

#### 35.2 Condition

```java
Condition cond = lock.newCondition();

cond.await();      // 相当于wait
cond.signal();    // 相当于notify
cond.signalAll();

// 多条件队列
Condition full = lock.newCondition();
Condition empty = lock.newCondition();
```

#### 35.3 回答模板

> ReentrantLock是java.util.concurrent的显式锁，比synchronized功能更丰富（可中断、可尝试、非公平）。建议synchronized日常coding用，ReentrantLock在需要高级特性时用。

---

### 36. Atomic原子类？

#### 36.1 原子类概览

```java
// 原子更新基本类型
AtomicInteger ai = new AtomicInteger(0);
ai.incrementAndGet();
ai.getAndIncrement();
ai.compareAndSet(0, 1);

// 原子更新数组
AtomicIntegerArray arr = new AtomicIntegerArray(10);

// 原子更新引用
AtomicReference<User> user = new AtomicReference<>();

// AtomicStampedReference解决ABA
AtomicStampedReference(ref, stamp);
```

#### 36.2 原理

```c
// CAS实现
do {
    old = current;
    new = (old + delta);
} while (!CAS(current, old, new));

// 自旋锁保证原子
// 多线程竞争时自旋，消耗CPU但无锁
```

#### 36.3 Unsafe

```java
// Unsafe直接操作内存
sun.misc.Unsafe unsafe = Unsafe.getUnsafe();
unsafe.getObjectVolatile(obj, offset);

// 开发不要直接用
// 用Atomic*
```

#### 36.4 回答模板

> Atomic系列是基于CAS + 自旋锁实现的原子操作类，比锁无阻塞高性能。更新失败重试，适用于计数、简单更新场景。AtomicStampedReference解决ABA问题。

---

### 37. CountDownLatch？

#### 37.1 用法

```java
// await等待N个完成
CountDownLatch latch = new CountDownLatch(10);

for (int i = 0; i < 10; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown();
    }).start();
}

latch.await();  // 等待10个countDown
// await(timeout, TimeUnit)
```

#### 37.2 特点

```plaintext
CountDownLatch vs CyclicBarrier：
- CountDownLatch一次性
- CyclicBarrier可循环

使用场景：
- CountDownLatch：等待N个任务完成
- CyclicBarrier：N个线程互相等待
```

#### 37.3 回答模板

> CountDownLatch是一次性的N个countDown后await返回，用于等待任务完成。CyclicBarrier是可循环的N个线程互相等待到达后一起继续。

---

### 38. Semaphore？

#### 38.1 用法

```java
// 信号量限制并发数
Semaphore sem = new Semaphore(5);

// 获取许可
sem.acquire(); // 可中断
try {
    // 并发受控代码
} finally {
    sem.release();
}

// 尝试
if (sem.tryAcquire(1, TimeUnit.SECONDS)) {
    try {
        //
    } finally {
        sem.release();
    }
}
```

#### 38.2 原理

```plaintext
Semaphore基于AQS：
- state=许可数
- acquire：state--
- release：state++

公平模式：FIFO
非公平模式：竞争
```

#### 38.3 回答模板

> Semaphore是信号量，控制同一时间最多N个线程执行，用于限流（数据库连接池等）。acquire获取许可，release释放。用tryAcquire可做超时等待。

---

### 39. ConcurrentHashMap？

#### 39.1 ConcurrentHashMap

```java
ConcurrentHashMap<Integer, User> map = new ConcurrentHashMap<>();

// putIfAbsent原子
User u = map.computeIfAbsent(id, k -> new User(k));

// 原子更新
map.merge(key, value, (ov, nv) -> ov + nv);

// sized
LongAdder adder = map.computeIfAbsent(id, LongAdder::new);
adder.increment();

// 推荐
```

#### 39.2 实现原理

```plaintext
JDK 7及以前：
- Segment数组+HashEntry链表
- ReentrantLock分段锁

JDK 8+：
- CAS+synchronized
- Node数组+链表/红黑树
- 无锁化
```

#### 39.3 回答模板

> ConcurrentHashMap是并发安全的Map，JDK8+用CAS+synchronized+nodes红黑树。computeIfAbsent/merge等是原子方法。适合高并发缓存。无ConcurrentSet，用newCopyOnWriteArraySet包装。

---

### 40. ThreadPoolExecutor？

#### 40.1 构建

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // corePoolSize
    10,                     // maxPoolSize
    60L,                    // keepAliveTime
    TimeUnit.SECONDS,       // unit
    new LinkedBlockingQueue<>(100), // queue
   Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy()
);
```

#### 40.2 流程

```plaintext
1. < corePoolSize？ → 创建线程执行
2. 队列不满→入队列等待
3. 队列满 & < maxPoolSize → 创建新线程
4. 都满 → 拒绝
```

#### 40.3 拒绝策略

```java
AbortPolicy       // 抛异常（默认）
CallerRunsPolicy // 调用者执行
DiscardPolicy   // 丢弃
DiscardOldestPolicy // 丢弃最老
```

#### 40.4 回答模板

> ThreadPoolExecutor有core/max线程数、队列、拒绝策略。线程=corePoolSize+keepAliveTime自动回收。Executors有快捷方法但生产建议手动new自己配置。

---

## 第五章 实战与调优（中高级 ★★★★）

### 41. OOM类型及排查？

#### 41.1 OOM类型

```plaintext
Java堆溢出：
java.lang.OutOfMemoryError: Java heap space
- jmap -dump分析
- 看对象大小、数量

元空间溢出：
java.lang.OutOfMemoryError: Metaspace
- -XX:MetaspaceSize设太小
- 类加载太多

栈溢出：
java.lang.StackOverflowError
- 递归死循环
- -Xss增大

直接内存：
java.lang.OutOfMemoryError: Direct buffer memory
- NIO用太多
- -XX:MaxDirectMemorySize
```

#### 41.2 排查步骤

```bash
# OOM时生成dump
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heap.hprof

# 分析工具
jhat heap.hprof
MAT
```

#### 41.3 回答模板

> OOM有几种：Heap（堆对象太多）、Metaspace（类太多）、Stack（递归太深）、DirectMemory（NIO用太多）。HeapDumpOnOutOfMemoryError生成dump用MAT分析OOM原因。对症下药。

---

### 42. GC调优目标？

#### 42.1 调优目标

```plaintext
GC调优目标：
1. 吞吐量优先（Throughput）
   - -XX:GCTimeRatio=N
   - GC时间<1/(N+1)

2. 停顿时间（Latency）
   - -XX:MaxGCPauseMillis=200
   - G1/ZGC

3.  footprint（最小化）
   - -Xmx设定合理
```

```bash
# 例子
# 吞吐量优先
-XX:+UseParallelGC -XX:+UseParallelOldGC
-XX:GCTimeRatio=19 -XX:SoftRefLRUPolicyMSPerMB=50

# 延迟优先
-XX:+UseG1GC -XX:MaxGCPauseMillis=200
```

#### 42.2 常见参数

```bash
# GC日志
-Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps

# GC日志文件滚动
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=10M
```

#### 42.3 回答模板

> GC优化前先设目标和约束：吞吐量>99%（GCTimeRatio=19）、停顿<200ms、G1。然后观察FGC/YGC次数、FGC时间调参。让对象尽量在Young区消亡减少Full GC。

---

### 43. 什么是内存溢出泄漏？

#### 43.1 定义

```plaintext
Memory Leak（内存泄漏）：
- 对象可达但不再使用
- GC无法回收
- 导致OOM

vs Memory Overflow：
- 真的用的内存太多
- 不是泄漏
```

#### 43.2 常见泄漏源

```java
// 典型泄漏案例
// 1. 静态集合
static Set<Object> cache = new HashSet<>();
// 不断add never remove

// 2. 监听器未移除
button.addActionListener(listener);
// 但frame dispose未remove

// 3. 内部类持有外部
class Inner {
    Outer outer; // 隐式持有Outer
}

// 4. 循环引用
A.next = B;
B.prev = A;
```

#### 43.3 回答模板

> 内存泄漏是对象不再使用但仍被引用无法GC排查：用leak suspect功能看堆增长的unreachable objects、常见静态集合内部类等泄漏源。

---

### 44. ReferenceQueue？

#### 44.1 引用队列

```java
// 软引用SoftReference
// 内存不足时被回收，加入queue

ReferenceQueue<Object> queue = new ReferenceQueue<>();
SoftReference<Object> ref = new SoftReference<>(new Object(), queue);

Reference<?> r;
while ((r = queue.poll()) != null) {
    // 被回收了，这里清理related resources
}
```

#### 44.2 WeakReference

```java
// WeakReference
// 只是GC就回收（如果仅弱引用）
WeakHashMap<K,V> 是天然缓存
// 没有其他引用时key消失

// PhantomReference
// 要配合ReferenceQueue
// 总是返回null，无法get()
// 用于post-mortem cleanups
```

#### 44.3 回答模板

> SoftReference内存不足时回收用于缓存；WeakReference无其他引用时回收，WeakHashMap是简单缓存；PhantomReference无法get做finalization后在queue处理后事。ReferenceQueue是被回收后入队的。

---

### 45. 虚引用和管理堆外内存？

#### 45.1 虚引用

```java
// 虚引用PhantomReference
// 必须搭配ReferenceQueue
// 对象的get()总是返回null

ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> ref = new PhantomReference<>(obj, queue);

Reference<?> r;
while ((r = queue.remove()) != null) {
    // 对象被回收了，做cleanup
}
```

#### 45.2 堆外内存

```java
// ByteBuffer.allocateDirect
ByteBuffer buf = ByteBuffer.allocateDirect(1024*1024);

// 实际调用native方法
// 使用sun.misc.Cleaner
Cleaner cleaner = ((DirectByteBuffer)buf).cleaner();
cleaner.clean();

// NIO使用DirectBuffer省去copy
```

#### 45.3 回答模板

> PhantomReference做堆外内存管理等对象的post-mortem清理。allocateDirect用堆外内存减少copy，省内存但allocation时间长。NIO默认用DirectBuffer。-XX:MaxDirectMemorySize设堆外内存大小。

---

### 46. WeakHashMap？

#### 46.1 WeakHashMap特点

```java
WeakHashMap<User, User> cache = new WeakHashMap<>();

User key = new User("001");
cache.put(key, user);
// key没有被其他引用时，GC会被回收
// WeakHashMap的entry被删除
```

```plaintext
与HashMap区别：
- WeakHashMap的key是弱引用
- 没有其他引用时自动移除entry
- 适合缓存
```

#### 46.2 应用场景

```java
// 简单缓存
WeakHashMap<Class<?>, Method[]> cache = new WeakHashMap<>();

Method[] methods(Class<?> cls) {
    return cache.computeIfAbsent(cls, c -> c.getMethods());
}
```

#### 46.3 回答模板

> WeakHashMap的key是WeakReference，没强引用的key会被自动移除entry，key被回收则value也可被回收。常用于简单缓存、避免重复创建（尤其是Class、Method）。

---

### 47. 如何减少GC？

#### 47.1 减少GC策略

```plaintext
1. 对象复用
   - 对象池
   - 复用StringBuilder
   - String.intern()

2. 数据结构优化
   - 选择合适集合
   - primitive wrappers

3. 减少临时对象
   - 避免auto boxing
   - 预分配大小

4. 减少大对象
   - long/double避免包装类
```

#### 47.2 参数调优

```bash
# 减少GC频率
-Xms=Xmx           # 禁用resize
-XX:NewRatio=2     # 调整Young/Old比例
-XX:MaxGCPauseMillis=200

# 对象年龄调大
-XX:InitialTenuringThreshold=10
-XX:MaxTenuringThreshold=15
```

#### 47.3 回答模板

> 减少GC策略：对象复用（如ThreadLocal、static SB）、数据结构优化、减少临时对象和auto boxing。参数设固定堆、Xms=Xmx，调整Young比例让对象young消亡。

---

### 48. 什么是G1的Mixed GC？

#### 48.1 Mixed GC

```plaintext
G1的Mixed GC流程：
1. initial mark（标记Young）
2. root region scan（GCRoots）
3. concurrent mark（并发标记）
4. remark（STW修正）
5. cleanup（STW计算价值）
6. live regions进CollectionSet（young + 选出的old）

7. copying（复制存活对象，STW）
```

#### 48.2 Humongous Objects

```plaintext
Humongous对象：
- >= RegionSize一半的对象
- 单独存Humongous区

问题：
- 容易引起碎片
- 触发连续Region分配

解决：
- -XX:G1HeapRegionSize设大
- 避免大于RegionSize的对象
```

#### 48.3 回答模板

> G1用Mixed GC收集young+选出的old regions，叫 CollectionSet。Humongous对象>Region一半存单独的Humongous区，容易碎片化。开发中要避免创建大对象。

---

### 49. 什么是ZGC的着色指针？

#### 49.1 着色指针

```c
// 64位指针，用高几位做颜色标记
// 0 Marked0
// 1 Marked1
// 2 Remapped
// 等等

// Marked0 || Marked1 表示gc中
// Remapped表示 relocated

读屏障：
if (pointers & marked) {
    // relocate object
    // update pointer
}
```

#### 49.2 特点

```plaintext
ZGC着色指针特点：
- 标记在指针不在对象头
- 扫描只需看指针
- 对象无需mark/unmark
- 并发mark/concurrent relocate
- STW非常短<10ms
```

#### 49.3 回答模板

> ZGC用着色指针Marked0/1/Remapped标记对象gc状态，指针标记而非对象头使得并发标记、relocation时对象无需移动，只要改指针。这是ZGC停顿时间短的根本原因。

---

### 50. JDK 8之后的进化？

#### 50.1 新特性和改进

```plaintext
JDK 9+：
- var类型推断
- 模块化（模块系统）
- 字符串底层改为byte[]
- 新的GC：Epsilon（空GC）、ZGC、Shenandoah

JDK 11+：
- var在lambda
- ZGC正式版
- Flight Recorder

JDK 12+：
- Shenandoah
- Switch表达式
- 文本块

JDK 14+：
- record类
- sealed classes预览
```

#### 50.2 GC发展趋势

```plaintext
GC演进：
Serial → Parallel → CMS → G1 → ZGC/Shenandoah

趋势：
- 更高的并行能力
- 更短的停顿时间
- 更大的内存支持
- 更好的资源利用
```

#### 50.3 回答模板

> JDK 8后有var类型推断、模块系统、ZGC/Shenandoah低延迟GC、字符串底层改byte[]省内存、用record类不可变类。GC趋势是更短停顿和更高并发。

---

## 附录：面试追问

1. **Java 17新特性？**
   - Sealed Classes正式
   - Pattern Matching for instanceof
   - Switch表达式完整
   - 虚拟线程（preview）

2. **G1何时触发Full GC？**
   - 并发模式失败
   - 晋升失败（无足够空间）
   - Evacuation失败

3. **String在JDK 9的变化？**
   - 底层char[]改byte[]
   - CompactString用coder

4. **JVM调优步骤？**
   - 设目标
   - 设约束
   - 监控
   - 迭代

5. **线上Full GC的原因？**
   - Metaspace满
   - 晋升失败（allocation failure）
   - 显式调用System.gc

---

## 参考资料

- 《深入理解Java虚拟机（第3版）周志明》
- 《Java Performance Companion》
- JVM规范文档
- GC Colloctors Papers

---

> 整理by Claude Code | JVM面试高频100问