# Java夺命连环100问——Java核心技术深度指南

> 本文档面向Java学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. Java的特点？

#### 1.1 核心特性

> Java是一门 WORA（Write Once, Run Anywhere）语言，通过JVM实现平台无关性。

```java
// Java核心特性：
// 1. 面向对象(OOP)
// 2. 平台无关(JVM)
// 3. 自动内存管理(GC)
// 4. 异常处理机制
// 5. 多线程支持
// 6. 反射(Reflect)
```

#### 1.2 JVM/JRE/JDK区别

```
JVM (Java Virtual Machine)：运行字节码的虚拟机
JRE (Java Runtime Environment)：JVM + 核心类库
JDK (Java Development Kit)：JRE + 开发工具(javac/java/jar等)

关系：JDK > JRE > JVM
开发需要JDK，生产运行只需要JRE
```

#### 1.3 回答模板

> Java是编译型+解释型语言，源文件.java编译成字节码.class，由JVM解释执行。一次编译到处运行，由JVM实现平台无关。自动GC管理内存，有异常处理、反射、多线程等特性。是企业级后端开发主流语言。

---

### 2. 面向对象四大特性？

#### 2.1 封装Encapsulation

```java
// 封装：属性私有化，公开方法
public class User {
    private String name;  // 私有属性
    private int age;

    public String getName() { return name; }  // getter
    public void setName(String name) { this.name = name; }  // setter
}
// 好处：数据安全、隐藏细节
```

#### 2.2 继承Inheritance

```java
// 继承：复用父类extends
class Person { String name; }
class Student extends Person { int score; }

// 特点：
// - 单继承（一个类只能extends一个）
// - 构造器不被继承
// - 子类访问父类用super
// - 子类可以覆盖(Override)父类方法
```

#### 2.3 多态Polymorphism

```java
// 多态：同一接口不同实现
// 1. 父类引用指向子类对象
Person p = new Student();
// 2. 方法重载(Overload) vs 重写(Override)
class Calc {
    int add(int a, int b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }  // 重载
}
class Calculator extends Calc {
    @Override
    int add(int a, int b) { return a + b + 1; }  // 重写
}
```

#### 2.4 抽象Abstraction

```java
// 抽象类和接口
abstract class Animal {
    abstract void eat();  // 抽象方法无实现
}

interface Flyable {
    void fly();  // 接口默认抽象
    // Java 8+可默认实现：default void test() {}
}
```

#### 2.5 回答模板

> 面向对象四大特性：封装（属性私有+方法访问）、继承（复用extends）、多态（父类引用指子类+重写）、抽象（abstract/interface）。封装保证安全，继承复用代码，重写实现多态，抽象定义规范。接口是规范，类是实现。

---

### 3. equals和==的区别？

#### 3.1 ==操作符

```java
// == 比较基本类型：值
// == 比较引用类型：地址（内存地址）
int a = 10;
int b = 10;
a == b  // true

String s1 = new String("hello");
String s2 = new String("hello");
s1 == s2  // false (不同对象)
s1 == "hello"  // false
```

#### 3.2 equals方法

```java
// equals：子类可重写，默认比较地址
String s1 = new String("hello");
String s2 = new String("hello");
s1.equals(s2)  // true (String重写了equals比较内容)

// equals重写规范：
// 1. 自反性：x.equals(x)为true
// 2. 对称性：x.equals(y) == y.equals(x)
// 3. 传递性
// 4. 一致性
// 5. x.equals(null)返回false
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return age == user.age && Objects.equals(name, user.name);
}
```

#### 3.3 hashCode

```java
// equals相等则hashCode必须相等
// hashCode相等equals不一定相等
// 容器(HashMap/HashSet)用hashCode先过滤再equals

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

#### 3.4 回答模板

> ==比较基本类型值，引用类型比地址（即是否是同一对象）。equals比较内容，String等类重写了equals。equals相等(hashcode相等)是集合(Map/Set)去重的前提。hashcodeequals必须配合重写。

---

### 4. String/StringBuilder/StringBuffer？

#### 4.1 String不可变

```java
// String：final类，char[] value用final修饰
// 每次操作生成新String
String s = "hello";
s = s + "world";  // 生成新对象，旧的被回收
// 性能低但线程安全
```

#### 4.2 StringBuilder

```java
// 可变，线程不安全，性能高
StringBuilder sb = new StringBuilder();
sb.append("hello");
sb.append("world");
sb.toString();  // "helloworld"
```

#### 4.3 StringBuffer

```java
// 可变，方法有synchronized同步，线程安全
StringBuffer sb = new StringBuffer();
sb.append("hello");
sb.toString();
```

#### 4.4 对比

| 类 | 可变性 | 线程安全 | 性能 |
|-----|--------|----------|------|
| String | 不可变 | 安全 | 低（每次新对象） |
| StringBuffer | 可变 | 安全 | 中 |
| StringBuilder | 可变 | 不安全 | 高 |

#### 4.5 回答模板

> String是不可变的final类，每次拼接生成新对象，性能低但线程安全。StringBuilder可变非同步性能最高，StringBuffer可变但方法加了synchronized线程安全。拼接多用StringBuilder，字符串常量池节省内存。

---

### 5. String常量池？

#### 5.1 常量池

```java
// 字面量创建
String s1 = "hello";  // 常量池
String s2 = "hello";
s1 == s2  // true，指向同一对象

// new创建
String s3 = new String("hello");  // 堆
s1 == s3  // false

// intern()
String s4 = new String("hello").intern();
s1 == s4  // true，加入常量池返回已存在的引用
```

#### 5.2 编译期确定

```java
// 只有编译期确定的常量才在常量池
String a = "hello";
String b = "hel" + "lo";  // 编译期合并
String c = "hel";
String d = c + "lo";  // 运行期拼接，不在同一地址
```

#### 5.3 回答模板

> Java有字符串常量池，相同字面量只创建一个String对象。字面量"hello"放常量池，new String每次创建新对象。intern()可以把堆对象加入常量池返回已有引用。常量池节省内存。

---

### 6. final关键字？

#### 6.1 修饰类/方法/变量

```java
// 修饰类：不可继承
public final class String { ... }

// 修饰方法：不可重写
public final void show() { ... }

// 修饰变量：常量，不可再赋值
final int AGE = 18;
// final修饰基本类型：值不可变
// final修饰引用类型：引用不可变（对象可变）
final List<String> list = new ArrayList<>();
list.add("a");  // OK，引用不可变
// list = new ArrayList<>(); // Error
```

#### 6.2 static final

```java
// static final：编译时常量
static final int VERSION = 1;  // 在常量池
// 运行时常量
static final Random r = new Random();
static final int num = r.nextInt(10);  // 运行生成
```

#### 6.3 回答模板

> final是终态：final类不可继承，final方法不可重写，final变量不可再赋值。不可变类（String等）用final保证不变性。static final是编译常量。

---

### 7. static关键��？

#### 7.1 静态成员

```java
// 静态成员：类共有，非实例独占
class User {
    static int count;  // 类变量，所有对象共享
    static void test() {}  // 类方法
}

// 调用
User.count = 10;
User.test();
```

#### 7.2 静态代码块

```java
// 只执行一次，类加载时
static {
    System.out.println("类加载");
    // 初始化静态资源
}
```

#### 7.3 静态内部类

```java
// 静态内部类：不需要外部类实例
class Outer {
    static class Inner { }
}
// 非静态内部类：需要外部类实例（会导致内存泄漏慎用）
```

#### 7.4 回答模板

> static是类级别，所有对象共享。static变量是类变量，static方法无需创建对象调用。静态代码块类加载时执行一次。静态内部类不持有外部类引用，安全。

---

### 8. abstract和interface？

#### 8.1 abstract抽象类

```java
// 抽象类：不能实例化
abstract class Animal {
    abstract void eat();  // 抽象方法无实现

    // 可有具体方法
    void breathe() { System.out.println("呼吸"); }
}

// 子类必须实现抽象方法或声明为抽象类
class Dog extends Animal {
    @Override
    void eat() { System.out.println("吃骨头"); }
}
```

#### 8.2 interface接口

```java
// 接口：完全抽象
interface Flyable {
    void fly();
    // Java 8+可默认实现
    default void test() {}
    // 静态方法
    static void staticMethod() {}
}

// 实现
class Bird implements Flyable {
    @Override
    public void fly() { System.out.println("飞"); }
}
```

#### 8.3 区别

| 特性 | abstract类 | interface接口 |
|------|-----------|------------|
| 继承 | extends | implements |
| 数量 | 单一 | 多实现 |
| 变量 | 无限制 | 只能是public static final |
| 方法 | 抽象+具体 | Java 8+可default |
| 构造器 | 有 | 无 |

#### 8.4 接口新特性

```java
// Java 9+接口可私有方法
interface JDBC {
    default void connect() {
        init();  // 调用私有方法
    }
    private void init() { }  // 私有方法
}
```

#### 8.5 回答模板

> abstract是抽象类可包含抽象方法和具体方法，子类单继承。interface是完全抽象JDK8+可默认实现，多 implements。接口用于定义规范，abstract用于模板方法模式。Java不支持多extends但支持多implements。

---

### 9. 重载和重写？

#### 9.1 重载Overload

```java
// 同类同名不同参数
class Calc {
    int add(int a, int b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }  // 参数个数不同
    double add(double a, double b) { return a + b; }  // 参数类型不同
}
```

#### 9.2 重写Override

```java
// 子类重写父类方法
class Animal {
    void shout() { System.out.println("叫"); }
}
class Dog extends Animal {
    @Override
    void shout() { System.out.println("汪汪"); }  // 重写
}
```

#### 9.3 规则对比

| 特性 | 重载 | 重写 |
|--------|------|------|
| 范围 | 同类 | 子父类 |
| 方法名 | 相同 | 相同 |
| 参数 | 不同 | 相同 |
| 返回类型 | 无关 | 兼容或相同 |
| 权限 | - | 子类 >= 父类 |

#### 9.4 回答模板

> 重载是同类同方法名不同参数（个数/类型），编译期决定。重写是子类重写父类方法，运行期绑定。重写权限不能更严格，返回类型要兼容。泛型擦除影响重载。

---

### 10. 泛型 Generics？

#### 10.1 泛型类/方法

```java
// 泛型类
class Box<T> {
    T item;
    void set(T item) { this.item = item; }
    T get() { return item; }
}
Box<String> box = new Box<>();
box.set("hello");
String s = box.get();

// 泛型方法
<T> T identity(T item) { return item; }
```

#### 10.2 泛型通配符

```java
// ? extends T：上限，生产者
List<? extends Number> list = new ArrayList<Integer>();

// ? super T：下限，消费者
List<? super Number> list = new ArrayList<Object>();

// ?：无界
List<?> list = new ArrayList<String>();
```

#### 10.3 泛型擦除

```java
// 编译后类型擦除为Object或上限
// List<String> → List（运行时）
// T → Object

class Node<T> {
    T item;
}
Node<String> node = new Node<>();
// 编译后 equivalent to:
Node node = new Node();
// 长城bridge方法兼容
```

#### 10.4 回答模板

> 泛型<T>编译期类型安全检查，运行期擦除为Object。extends是生产者读（只读），super是消费者写（可add）。泛型不能创建泛型数组。

---

## 第二章 集合框架篇（高频 ★★★★★）

### 11. Collection类结构？

#### 11.1 继承树

```
Collection (接口)
├── List (接口)  // 有序可重复
│   ├── ArrayList // 数组，O(1)随机
│   ├── LinkedList // 双向链表，O(1)头尾
│   └── Vector // 同步（古老）
├── Set (接口)   // 无序不可重复
│   ├── HashSet // 哈希表，O(1)
│   ├── LinkedHashSet // 保证插入顺序
│   └── TreeSet // 红黑树，有序O(logN)
└── Queue (接口)
    ├── Deque
    │   ├── ArrayDeque
    │   └── LinkedList
    └── PriorityQueue // 优先级队列
```

#### 11.2 Map类结构

```
Map (接口)
├── HashMap // JDK8+数组+链表+红黑树O(1)
├── LinkedHashMap // 保证插入顺序
├── Hashtable // 同步（古老）
├── TreeMap // 红黑树，有序O(logN)
└── Properties // 继承Hashtable配置
```

#### 11.3 回答模板

> 集合分Collection和Map。List有序ArrayList，Set无序去重HashSet。Map用HashMap。ArrayList查改O(1)红黑树O(logN)。线程安全用Collections.synchronizedList或java.util.concurrent。

---

### 12. ArrayList和LinkedList？

#### 12.1 区别

```
ArrayList：
- 原理：Object[]数组
- 查找：O(1)（下标）
- 插入删除：O(N)需移动
- 内存：连续

LinkedList：
- 原理：双向链表+Node
- 查找：O(N)需遍历
- 插入删除：O(1)改变指针
- 内存：离散+Nodeleft/right
```

#### 12.2 源码要点

```java
// ArrayList.add()
ensureCapacityInternal(size + 1);
elementData[size++] = e;

// LinkedList.Node
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

#### 12.3 选择

```
频繁查询：用ArrayList
频繁头尾插入删除：用LinkedList
ArrayList是日常首选
```

#### 12.4 回答模板

> ArrayList用数组下标O(1)查询，但插入O(N)移动。LinkedList双向链表插入O(1)但查询需遍历O(N)。HashMap底层数组+链表+红黑树O(1)。日常选ArrayList，频繁插入删除LinkedList。

---

### 13. HashMap底层原理？

#### 13.1 JDK 8+数据结构

```java
// 数组+链表+红黑树
transient Node<K,V>[] table;

static class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;  // 链表
}
```

#### 13.2 扰动函数

```java
// hash = key.hashCode() ^ (h >>> 16)
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### 13.3 插入流程

```java
// 1. 计算hash
// 2. 定位位置：hash & (length-1) 等价 hash%length
// 3. tab[i]为空直接new Node
// 4. 不为空：比较hash和key
// 5. key相等则覆盖value
// 6. 否则链到next或转红黑树（链表>8且数组>64）
// 7. 检查threshold扩容
```

#### 13.4 扩容机制

```java
// threshold = capacity * loadFactor (默认0.75)
if (++size > threshold) resize();
new Tab = oldCap * 2;
transfer(newTab);  // 重新hash（高开销）
```

#### 13.5 回答模板

> HashMap用数组+链表+红黑树，hash扰动降低碰撞，hash%Length用位运算更高效。链表长度>8转红黑树防退化。resize()扩容很耗时，设计预估容量可避免。线程不安全，ConcurrentHashMap是安全版。

---

### 14. HashMap vs HashTable vs ConcurrentHashMap？

#### 14.1 对比

```
HashMap：线程不安全，key可为null，允许一个null key
Hashtable：线程安全（synchronized），key不可为null，性能低
ConcurrentHashMap：线程安全（CAS+Segment），JDK8+用CAS无锁，性能高
```

#### 14.2 ConcurrentHashMap原理

```java
// JDK 8+：
// 无Segment，用CAS添加头结点+synchronized锁冲突节点
// 并发度高的性能好

// putIfAbsent
map.computeIfAbsent(key, k -> function);
```

#### 14.3 选择

```
单线程用HashMap
并发用ConcurrentHashMap（高并发）
Hashtable已淘汰
```

#### 14.4 回答模板

> HashMap非线程安全，并发用ConcurrentHashMap（CAS+synchronized），不建议用Hashtable（旧API）。ConcurrentHashMap无null key，高并发性能好。

---

### 15. Collection工具类？

#### 15.1 Collections常用方法

```java
// 排序
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);

// 线程安全
List<Object> syncList = Collections.synchronizedList(new ArrayList<>());
Map<Object,Object> syncMap = Collections.synchronizedMap(new HashMap<>());

// 空集合（省内存）
Collections.emptyList();  // 只读，不可添加
Lists.newArrayList();
Maps.newHashMap();

// 线程安全集合
CopyOnWriteArrayList, ConcurrentHashMap
```

#### 15.2 Arrays

```java
// 数组转List
String[] arr = {"a","b"};
List<String> list = Arrays.asList(arr);  // 固定大小
List<String> list2 = new ArrayList<>(Arrays.asList(arr));  // 可变

// 排序
Arrays.sort(arr);

// 二分查找
Arrays.binarySearch(arr, "b");
```

#### 15.3 回答模板

> Collections工具类提供排序/反转线程安全集合。Arrays.asList返回的List不可增删。空集合emptyList()比new ArrayList省内存。并发集合用java.util.concurrent包。

---

### 16. ArrayList扩容？

#### 16.1 扩容机制

```java
// add时检查
ensureCapacityInternal(size + 1);
// capacity = 1.5倍 + 1
int newCapacity = oldCapacity + (oldCapacity >> 1);
// Arrays.copyOf复制迁移
elementData = Arrays.copyOf(elementData, newCapacity);
```

#### 16.2 小优化

```java
// 预知大小时指定初始容量
new ArrayList<>(1000);  // 避免多次扩容
new ArrayList<>(Arrays.asList(arr));  // 数组大小
```

#### 16.3 回答模板

> ArrayList每次添加检查容量，扩容量1.5倍+1（2^n-1类似），用Arrays.copyOf迁移数组。频繁add会导致多次扩容迁移数据，可预指定initialCapacity避免。

---

### 17. fail-fast和fail-safe？

#### 17.1 fail-fast

```java
// fail-fast iterator并发修改检测
ArrayList<String> list = new ArrayList<>();
list.add("a");
Iterator it = list.iterator();
list.add("b");  // ModCount改变
it.next();  // ConcurrentModificationException

// 底层记录modCount，迭代时比较
int expectedModCount = modCount;
```

#### 17.2 fail-safe

```java
// 迭代时复制副本，使用弱一致性
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("a");
Iterator it = list.iterator();
list.add("b");  // 不抛异常
while(it.hasNext()) { System.out.println(it.next()); }
```

#### 17.3 回答模板

> fail-fast迭代器如ArrayList，在迭代时检测并发修改抛ConcurrentModificationException（不等于绝对安全）。fail-safe如CopyOnWriteArrayList，迭代副本不抛异常但可能过时。

---

### 18. List删除元素？

#### 18.1 foreach删除有问题

```java
// 不要在foreach/for中删除
List<String> list = new ArrayList<>(Arrays.asList("a","b","c"));
for(String s : list) {
    if("b".equals(s)) {
        list.remove(s);  // 可能抛ConcurrentModificationException
    }
}
```

#### 18.2 正确删除

```java
// 方案1：iterator
Iterator<String> it = list.iterator();
while(it.hasNext()) {
    if("b".equals(it.next())) {
        it.remove();
    }
}

// 方案2：removeIf (JDK 8+)
list.removeIf("b"::equals);

// 方案3：倒序遍历
for(int i=list.size()-1;i>=0;i--) {
    if("b".equals(list.get(i))) {
        list.remove(i);
    }
}
```

#### 18.3 回答模板

> foreach/for删除会ConcurrentModificationException。用Iterator.remove()或removeIf()或倒序遍历。增强for本质是iterator。

---

## 第三章 异常处理篇（高频 ★★★★★）

### 19. 异常体系？

#### 19.1 Throwable继承

```
Throwable
├── Error（错误）  // JVM错误，不可捕获
│   ├── OutOfMemoryError
│   └── StackOverflowError
├── Exception（异常）
│   ├── RuntimeException（运行时）
│   │   ├── NullPointerException
│   │   ├── IndexOutOfBoundsException
│   │   ├── IllegalArgumentException
│   │   └── ClassCastException
│   └��─ IOException等受检异常
```

#### 19.2 受检vs非受检

```java
// 受检异常Checked：必须捕获或抛出
public void read() throws IOException {
    FileReader fr = new FileReader("file");
    // 不捕获则编译不通过
}

// 非受检异常Unchecked：RuntimeException及其子类
// 可以不捕获
int a = 10 / 0;  // ArithmeticException
```

#### 19.3 回答模板

> Error是JVM错误如OOM，通常不捕获。RuntimeException非受检如NPE/IndexOutOfBounds，程序问题。受检异常IOException必须throws捕获。

---

### 20. try-catch-finally？

#### 20.1 用法

```java
try {
    // 可能异常的代码
} catch (ArithmeticException e) {
    System.out.println("除零：" + e.getMessage());
} catch (NullPointerException e) {
    System.out.println("空指针");
} finally {
    // 总是执行，释放资源
    // close流/锁
}
```

#### 20.2 return和finally

```java
// finally在return前执行！
public int test() {
    try {
        return 1;
    } finally {
        // 这里的代码会在return前执行
        System.out.println("finally执行");
    }
}
```

#### 20.3 try-with-resources

```java
// 自动关闭实现AutoCloseable的资源
try (FileReader fr = new FileReader("file");
     BufferedReader br = new BufferedReader(fr)) {
    String line = br.readLine();
}  // 自动调用close()
```

#### 20.4 回答模板

> try-catch包裹可能异常代码，finally无论是否异常都执行，用于释放资源。return会先执行finally再返回。JDK7+try-with-resources自动关闭。

---

### 21. throw和throws？

#### 21.1 声明throws

```java
// throws：方法声明可能抛出的异常
public void read() throws IOException, SQLException {
    // 方法体
}
```

#### 21.2 抛出throw

```java
// throw：抛出一个异常
if (age < 0) {
    throw new IllegalArgumentException("年龄不能为负数");
}
```

#### 21.3 回答模板

> throws是方法声明可能抛出的异常类型，调用者需处理。throw是手动抛出一个异常。RuntimeException可throw但可不处理，非受检必须处理。

---

### 22. 异常处理规范？

#### 22.1 catch顺序

```java
// 从具体到宽泛
try {
    int a = 1 / 0;
} catch (ArithmeticException e) {
    // 具体异常在前
} catch (RuntimeException e) {
    // 宽泛在后
} catch (Exception e) {
    // 最后兜底
}
```

#### 22.2 不要生吞异常

```java
// ❌ 生吞异常（危险）
try {
    doSomething();
} catch (Exception e) {
    // 什么都不做，相当于没处理
}

// ✅ 正确处理
try {
    doSomething();
} catch (Exception e) {
    log.error(e.getMessage(), e);  // 记录日志
    return Result.error(500, "系统错误");  // 友好返回
}
```

#### 22.3 回答模板

> catch从具体到宽泛顺序。最底层catch要记录日志+有用户友好的返回值，不要生吞异常。用业务异常继承RuntimeException包装业务错误码。

---

## 第四章 多线程篇（高频 ★★★★★）

### 23. 创建线程的方式？

#### 23.1 三种方式

```java
// 1. 继承Thread
class MyThread extends Thread {
    @Override
    public void run() { System.out.println("thread"); }
}
new MyThread().start();

// 2. 实现Runnable
Runnable task = () -> System.out.println("runnable");
new Thread(task).start();

// 3. 实现Callable（可返回结果/抛异常）
Callable<String> callable = () -> "result";
FutureTask<String> ft = new FutureTask<>(callable);
new Thread(ft).start();
String result = ft.get();
```

#### 23.2 线程池创建

```java
// 使用ExecutorService
ExecutorService pool = Executors.newFixedThreadPool(4);
pool.submit(() -> System.out.println("task"));
pool.shutdown();
```

#### 23.3 回答模板

> 继承Thread、实现Runnable、实现Callable三种方式。Callable可返回值和抛异常，实际推荐用线程池Executors创建复用线程，避免频繁创建销毁。

---

### 24. 线程状态？

#### 24.1 六状态

```
NEW：创建未start
RUNNABLE：运行中或就绪
BLOCKED：等待锁（synchronized）
WAITING：无限等待（wait/join/park）
TIMED_WAITING：限时等待（sleep/wait/join）
TERMINATED：执行完
```

#### 24.2 状态转换

```java
NEW ──start()──▶ RUNNABLE
RUNNABLE──wait()──▶ BLOCKED
RUNNABLE──lock()──▶ BLOCKED
BLOCKED──notify()──▶ RUNNABLE
```

#### 24.3 回答模板

> 线程有NEW/RUNNABLE/BLOCKED/WAITING/TIMED_WAITING/TERMINATED六状态。synchronized阻塞是BLOCKED，Object.wait()是WAITING。BLOCKED/WAITING都是暂停执行，BLOCKED是等锁，WAITING是等notify。

---

### 25. Synchronized的原理？

#### 25.1 用法

```java
// 方法锁
synchronized void method() { }

// 代码块锁
synchronized(this) { }

// 静态方法锁（锁类）
synchronized static void staticMethod() { }
```

#### 25.2 monitorenter/monitorexit

```java
// 字节码层面
monitorenter // 获取monitor
monitorexit  // 释放

// 基于对象头MarkWord中的Owner/Count
// 无锁 → 偏向锁 → 轻量锁 → 重量锁
```

#### 25.3 回答模板

> synchronized基于对象头Monitor实现，多线程共享同一对象时互斥。锁有升级倾向：无锁→偏向锁（首次CAS）→轻量锁（自旋）→重量锁（阻塞），JDK6有优化。高并发可关闭偏向锁-XX:-UseBiasedLocking。

---

### 26. volatile的作用？

#### 26.1 可见性和有序性

```java
// 可见性：确保读到最新值
volatile boolean flag = true;
while(!flag) {}  // 线程可看到主存变化

// 有序性：禁止指令重排
// 插入内存屏障禁止重排
```

#### 26.2 底层实现

```java
// 汇编层面： LOCK指令
// 1.写会立即刷新到主存
// 2.使其他CPU缓存失效
// 3. StoreLoad屏障
```

#### 26.3 不保证原子性

```java
// volatile不自带原子
volatile int count = 0;
count++;  // 复合操作，读-改-写，三步可能被打断

// 原子操作需用AtomicInteger
AtomicInteger count = new AtomicInteger();
count.incrementAndGet();
```

#### 26.4 回答模板

> volatile保证可见性（读最新值）和有序性（禁止重排），不保证原子性。适用一度一排场景如状态标记。单例双重检查DCL用volatile防止指令重排。复合操作仍需原子类。

---

### 27. ThreadLocal？

#### 27.1 用法

```java
// ThreadLocal每个线程独立的值
ThreadLocal<String> tl = new ThreadLocal<>();
tl.set("hello");

String s = tl.get();  // 本线程取值，不受其他线程影响
tl.remove();  // 必须去除防止内存泄漏
```

#### 27.2 原理

```java
// Thread.threadLocals（ThreadLocalMap）
// Entry extends WeakReference<ThreadLocal<?>> 弱引用key
// Value强引用导致内存泄漏
```

#### 27.3 内存泄漏

```java
// 内存泄漏原因：
// key是ThreadLocal弱引用可被回收
// value强引用在线程存活期间不会被回收
// 必须remove()

// 泄漏解决：在finally中remove
try {
    tl.set("value");
} finally {
    tl.remove();
}
```

#### 27.4 回答模板

> ThreadLocal为每个线程提供独立的变量副本，互相隔离。弱引用key可被回收但value强引用导致内存泄漏。线程池环境下必须在finally中remove()。

---

### 28. volatile和synchronized的区别？

#### 28.1 对比

| 特性 | volatile | synchronized |
|------|---------|--------------|
| 作用 | 修饰变量 | 修饰代码块/方法 |
| 原子性 | 不保证 | 保证 |
| 可见性 | 保证 | 保证 |
| 有序性 | 保证 | 保证 |
| 性能 | 高 | 低（需要获取锁） |

#### 28.2 使用场景

```java
// 状态标记用volatile
volatile boolean shutdown = false;

// 互斥操作用synchronized
synchronized(obj) { x++ }
```

#### 28.3 回答模板

> volatile是轻量级同步修饰变量，保证可见+有序但非原子（复合操作不保证）。synchronized可修饰代码块或方法，保证原子、可见、有序但性能低。一度一排用volatile，互斥用synchronized。

---

### 29. wait和sleep的区别？

#### 29.1 wait vs sleep

| 特性 | wait | sleep |
|------|------|-------|
| 所属 | Object | Thread |
| 释放锁 | 是 | 否（持有锁） |
| 超时 | 可 | 必须指定时间 |
| 调用 | this.wait() | Thread.sleep(ms) |
| CPU | 不占用 | 不占用 |

#### 29.2 正确姿势

```java
// wait必须在synchronized中使用
synchronized(obj) {
    while(condition) {
        obj.wait();  // 等待通知
    }
}

// notify唤醒
synchronized(otherObj) {
    otherObj.notify();
}
```

#### 29.3 回答模板

> wait是Object的方法需在synchronized中调用，释放锁让其他线程进入。sleep是Thread静态方法，不释放锁。wait需配合notify/notifyAll使用，常用于生产者-消费者。

---

### 30. 创建线程池的方式？

#### 30.1 Executors工厂方法

```java
// 固定线程数
ExecutorService pool = Executors.newFixedThreadPool(5);

// 单线程
ExecutorService pool = Executors.newSingleThreadExecutor();

// 缓存线程（动态伸缩）
ExecutorService pool = Executors.newCachedThreadPool();

// 定时任务
ScheduledExecutorService ses = Executors.newScheduledThreadPool(5);
```

#### 30.2 ThreadPoolExecutor

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    corePoolSize,    // 核心线程数
    maxPoolSize,     // 最大线程数
    keepAliveTime,   // 线程存活时间
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100),
    new ThreadFactory() { public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setName("worker-" + pool.getPoolSize());
        t.setDaemon(true);
        return t;
    }},
    new RejectedExecutionHandler() {
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r + " rejected");
        }
});
```

#### 30.3 参数设置

```java
// CPU密集型：CPU核心数+1
// IO密集型：CPU核心数 * 2 或 CPU/(1-阻塞系数)
// 混合型：基准评估

int cores = Runtime.getRuntime().availableProcessors();
int threads = cores + 1;  // CPU密集型
```

#### 30.4 回答模板

> Executors.newFixedThreadPool创建固定线程数，newCachedThreadPool动态伸缩生产少用。参数设置core/max/core+1队列，高并发队列别太大。Executors创建易，但生产建议用ThreadPoolExecutor手动设置参数。

---

## 第五章 Java 8+新特性（中高频 ★★★★）

### 31. Lambda表达式？

#### 31.1 语法

```java
// 完整语法
(参数列表) -> { 方法体 }

// 省略
list.forEach(s -> System.out.println(s));
list.forEach(System.out::println);
list.forEach(StringUtils::method);

// 单参数可省略括号
list.forEach(s -> {
    String str = s.trim();
    System.out.println(str);
});
```

#### 31.2 函数式接口

```java
// 只有一个抽象方法的接口是函数式接口，用于Lambda
@FunctionalInterface
interface Converter<F,T> {
    T convert(F from);
}

// 使用
Converter<String, Integer> cv = s -> Integer.parseInt(s);
Integer num = cv.convert("123");
```

#### 31.3 回答模板

> Lambda是函数式编程，使代码简洁。()->{}语法，单参数可省略().常见于list.forEach/map等集合操作。方法引用::Class::method简化。

---

### 32. Stream流？

#### 32.1 创建流

```java
Stream<String> stream = list.stream();  // 从集合
int[] arr = {1,2,3};
Arrays.stream(arr).sum();

// 生成
Stream.generate(Math::random).limit(5).forEach(Math::println);

// 迭代
Stream.iterate(1, n -> n+1).limit(10).sum();
```

#### 32.2 操作分类

```java
// 中间操作（返回Stream）
.filter(s -> s.startsWith("a"))
.map(String::toUpperCase)
.distinct()
.limit(10)
.skip(5)
.sorted();

// 终止操作（返回结果）
.collect(Collectors.toList());
.count();
.reduce(0, Integer::sum);
.forEach(System.out::println);
.toArray();
.anyMatch(BooleanExpression);
.allMatch(BooleanExpression);
.noneMatch(BooleanExpression);
```

#### 32.3 收集器

```java
// 收集到List
List<String> list = stream.collect(Collectors.toList());

// 分组
Map<String,List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

// 分区
Map<Boolean, List<Person>> primes = stream
    .collect(Collectors.partitioningBy(p -> p.getAge() > 18));

// joining合并字符串
String names = list.stream()
    .map(Person::getName)
    .collect(Collectors.joining(","));
```

#### 32.4 回答模板

> Stream提供函数式集合操作。中间操作filter/map返回Stream可链式，terminal operation触发执行。常用collect收集结果toList/groupingBy/toMap。惰性求值需terminal才执行。

---

### 33. Optional？

#### 33.1 创建

```java
Optional<String> opt = Optional.of("value");  // 非null
Optional<String> empty = Optional.empty();  // 空
Optional<String> opt2 = Optional.ofNullable(null);  // 允许null

// 使用
opt.isPresent()
opt.get()
opt.orElse("default")
opt.orElseGet(() -> "lazy default")
opt.orElseThrow(Supplier::newException)
```

#### 33.2 常用方法

```java
// map转换
Optional<Integer> len = opt.map(String::length);

// flatMap展平
Stream<String> stream = opt.flatMap(o -> o.stream());

// filter过滤
Optional<String> filtered = opt.filter(s -> s.length() > 3);

// ifPresent消费
opt.ifPresent(System.out::println);
```

#### 33.3 链式使用

```java
// 避免NPE
String city = person.getOpt()
    .map(Person::getAddress)
    .map(Address::getCity)
    .orElse("unknown");
```

#### 33.4 回答模板

> Optional容器化null值，API丰富避免NPE。of非null，ofNullable允许null，empty空值。map/filter/flatMap链式操作。orElse/ifPresent提供默认值或消费值。

---

### 34. 接口默认方法？

#### 34.1 default方法

```java
interface Formula {
    // 普通抽象方法
    double calculate(int a);
    // 默认实现
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
new Formula() {
    @Override
    public double calculate(int a) { return a * 2; }
}.sqrt(16);  // 4.0
```

#### 34.2 static方法

```java
interface Interface {
    static void staticMethod() {
        System.out.println("static");
    }
}
Interface.staticMethod();
```

#### 34.3 多实现冲突

```java
// 两个接口都有相同default，子类必须重写
interface A { default void test() { System.out.println("A"); }}
interface B { default void test() { System.out.println("B"); }}

class C implements A, B {
    @Override
    public void test() { A.super.test(); B.super.test(); }}
```

#### 34.4 回答模板

> JDK 8+接口新增default和static方法。default方法有实现，实现类可直接用。避免default导致接口彻底僵化。static等同于工具方法。多实现冲突需子类显式指定A.super.test()。

---

### 35. 日期时间API？

#### 35.1 时间对象

```java
// LocalDateTime
LocalDate now = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dt = LocalDateTime.now();

// 指定时间
LocalDate d = LocalDate.of(2024, 1, 1);
LocalDateTime dt = LocalDateTime.of(2024,1,1,9,30);
```

#### 35.2 格式化

```java
// 格式化
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String str = dt.format(fmt);

// 解析
LocalDateTime parsed = LocalDateTime.parse("2024-01-01 09:30:00", fmt);

// Instant时间戳
Instant instant = dt.toEpochSecond(ZoneOffset.UTC);
long ts = instant.toEpochMilli();
```

#### 35.3 时间计算

```java
// 加减
dt.plusDays(1);
dt.plusHours(2);
dt.minusYears(5);

// 取值
dt.getYear();
dt.getMonthValue();
dt.getDayOfMonth();

// Period Duration
Period p = LocalDate.of(2024,1,1).until(LocalDate.now());
Duration d = LocalDateTime.MINUS.between(start, end);
```

#### 35.4 回答模板

> JDK 8+日期API在java.time包，比Date/SimpleDateFormat线程安全。LocalDate日期，LocalTime时间，LocalDateTime日期时间。DateTimeFormatter格式化解Parse，支持Duration/Period时间计算。

---

### 36. 反射Reflection？

#### 36.1 获取Class

```java
// 三种方式
Class<?> c1 = Class.forName("com.demo.User");
Class<?> c2 = User.class;
Class<?> c3 = new User().getClass();
```

#### 36.2 创建实例

```java
// 无参构造
Object o = c.newInstance();

// 有参数构造
Constructor<?> cons = c.getConstructor(String.class, int.class);
Object o = cons.newInstance("John", 30);
```

#### 36.3 访问属性

```java
Field f = c.getDeclaredField("name");
f.setAccessible(true);  // private需设置
String name = (String) f.get(obj);
f.set(obj, "new value");
```

#### 36.4 访问方法

```java
Method m = c.getMethod("methodName", String.class, int.class);
m.setAccessible(true);
Object result = m.invoke(obj, "param", 1);

// 私有方法
Method m = c.getDeclaredMethod("methodName");
m.setAccessible(true);
```

#### 36.5 回答模板

> 反射在运行期动态获取类信息、创建对象、访问属性/方法。Spring DI、JDBC、序列化都是反射的应用。反射比直接调用慢3-20倍，无必要时不用。Accessible强accessible(true)。

---

### 37. 动态代理？

#### 37.1 JDK动态代理

```java
// InvocationHandler
InvocationHandler handler = (proxy, method, args) -> {
    // 前置处理
    Object result = method.invoke(target, args);
    // 后置处理
    return result;
};

// 生成代理对象
Object proxy = Proxy.newProxyInstance(
    classLoader,
    interfaces,
    handler
);
```

#### 37.2 CGLib动态代理

```java
// Enhancer
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(TargetClass.class);
enhancer.setCallback((MethodInterceptor)(obj, method, args, proxy) -> {
    // 前置处理
    Object result = proxy.invokeSuper(obj, args);
    // 后置处理
    return result;
});
TargetClass proxy = (TargetClass) enhancer.create();
```

#### 37.3 对比

| 特性 | JDK代理 | CGLib代理 |
|------|--------|---------|
| 原理 | 实现接口 | 继承子类 |
| 性能 | 稍慢 | 快 |
| 限制 | 需接口 | 不能final |

#### 37.4 回答模板

> JDK动态代理基于接口+CGLib基于继承实现AOP。Spring默认接口用JDK，需要子类用CGLib（spring.aop.proxy-target-class=true）。JDK不能代理无接口类。

---

### 38. 注解原理？

#### 38.1 元注解

```java
@Retention(RUNTIME)  // 保留到运行期
@Target(METHOD)      // 作用目标
@Documented         // 生成JavaDoc
@Inherited         // 子类可继承
```

#### 38.2 自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Login {
    String value() default "";
}
```

#### 38.3 读取注解

```java
// 读取类/方法/字段注解
@Login("admin")
public class Test {
    @Login
    public void test() {}
}

// 反射读取
Method m = Test.class.getMethod("test");
Login login = m.getAnnotation(Login.class);
if(login != null) { /*有注解*/ }
```

#### 38.4 回答模板

> 注解是元数据标签，@Retention定义保留阶段 SOURCE/CLASS/RUNTIME。@Target定义作用目标METHOD/FIELD/PARAMETER等。运行期通过反射读取注解来实现功能如Spring MVC @RequestMapping。

---

### 39. 序列化和反序列化？

#### 39.1 实现Serializable

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private transient String password;  // transient不序列化
}
```

#### 39.2 序列化流

```java
// 序列化
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.ser"));
oos.writeObject(user);

// 反序列化
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.ser"));
User user = (User) ois.readObject();
```

#### 39.3 serialVersionUID

```
serialVersionUID影响反序列化：
- 相同：正常
- 不同：InvalidClassException
建议手动声明，防止修改类字段反序列化失败
```

#### 39.4 回答模板

> 序列化实现Serializable，transient字段不序列化。serialVersionUID版本号不同会反序列化失败需保持一致。serialVersionUID=1L手动声明。Java序列化不安全且性能一般，JSON/XML/Protobuf更常用。

---

### 40. 类的加载过程？

#### 40.1 加载环节

```
加载 loading：
- 通过类名获取类的二进制字节流
- 转化为方法区的运行时数据结构
- 生成java.lang.Class对象

链接 linking：
- 验证：文件格式/语义/字节码验证
- 准备：为static变量分配内存赋零值
- 解析：符号引用转直接引用

初始化 initialization：
- clinit()方法执行
- 静态赋值+静态代码块
- 父类先于子类
```

#### 40.2 类加载器

```
Bootstrap ClassLoader：C++实现，加载JAVA_HOME/lib
Extension ClassLoader：加载JAVA_HOME/lib/ext
App ClassLoader：加载classpath
自定义加载器
```

#### 40.3 双亲委派

```java
// 委派父类加载
protected Class<?> loadClass(String name, boolean resolve) {
    Class<?> c = findLoadedClass(name);
    if (c == null) {
        if (parent != null) c = parent.loadClass(name, false);
        else c = findSystemClass(name);
    }
    // 父无法加载，自己加载
    if (c == null) c = findClass(name);
    return c;
}
```

#### 40.4 回答模板

> 类加载分加载/链接/初始化三阶段。链接的准备阶段赋零值，initialization赋初值执行static块。双亲委派保证类唯一性，自定义ClassLoader可打破委派实现热加载等高级功能。

---

## 第六章 JVM与性能篇（中高级 ★★★★）

### 41. JVM内存结构？

#### 41.1 运行时数据区

```
线程共享：
- 堆 Heap：对象实例数组（GC主要区域）
- 方法区/Metaspace：类信息/常量/静态变量

线程私有：
- 虚拟机栈 VM Stack：方法帧栈
- 本地方法栈 Native Stack
- PC寄存器：执行位置
```

#### 41.2 对象在堆中布局

```
对象头（Object Header）：
- Mark Word(64bits)：锁/hash/age
- Klass指针(32bits)：指向类元数据

实例数据（Instance Data）：
- 父类字段先，子类字段后
- 字段按大小排列减少padding

对齐填充（Padding）：
- 8字节对齐
```

#### 41.3 回答模板

> JVM内存分线程共享的堆/方法区和私有的栈/本地栈/PC寄存器。堆对象GC主要区域，方法区存类信息 JDK8改为Metaspace本地内存。对象头有Mark Word锁状态。

---

### 42. GC垃圾回收？

#### 42.1 判断对象可回收

```java
// 1. 引用计数（无法处理循环引用，已淘汰）
// 2. 可达性分析（GC Roots遍历）

GC Roots包括：
- 栈中对象
- static属性对象
- 常量对象
- 本地方法栈对象
- Synchronized锁对象
```

#### 42.2 GC算法

```
标记清除：Mark-Sweep，两次扫描效率低
标记压缩：Mark-Compact，多scan一次效率中等
复制：Copying，适合新生代（95%对象死）
分代：年轻代复制，老年代标记压缩
```

#### 42.3 垃圾收集器

```
Serial：单线程，简单
ParNew：多线程，年轻代
Parallel：吞吐量优先
CMS：并发低停顿
G1：_REGION分区，可预测停顿
ZGC：TB级低延迟
```

#### 42.4 回答模板

> GC用可达性分析从GC Roots开始。年轻代用复制算法（Survivor），老年代用标记压缩/清除。Serial单线程停顿长，ParNew多线程，G1是JDK11前默认，ZGC低延迟。

---

### 43. OOM内存溢出？

#### 43.1 OOM类型

```
Java heap space：堆对象太多
Metaspace：类加载太多
StackOverflowError：递归太深 -Xss
Direct buffer memory：NIO分配太多

PermGen space (JDK7-)：JDK8改为Metaspace
- XX:MetaspaceSize=256m
- XX:MaxMetaspaceSize=512m
```

#### 43.2 解决

```
1. HeapDumpOnOutOfMemoryError生成dump
-XX:+HeapDumpOnOutOfMemoryError
2. MAT分析dump
3. 代码优化
```

#### 43.3 回答模板

> OOM_heap是堆对象太多或内存泄漏，OOM_PermGen/J8_PermSize是类加载太多OOM_Metaspace。StackOverflowError用-Xss增大栈。HeapDumpOnOutOfMemoryError生成dump并用MAT分析。

---

### 44. 内存调优参数？

#### 44.1 堆参数

```bash
-Xms512m    # 初始堆
-Xmx2048m   # 最大堆
-Xmn256m    # 年轻代大小
-XX:NewRatio=2  # 年轻代:老年代=1:2
-XX:SurvivorRatio=8  # Eden:Survivor=8:1
```

#### 44.2 GC参数

```bash
-XX:+UseSerialGC       # Serial
-XX:+UseParNewGC       # ParNew+CMS
-XX:+UseParallelGC     # Parallel
-XX:+UseG1GC          # G1
-XX:+UseZGC           # ZGC

-XX:+PrintGCDetails  # 打印详情
-Xloggc:gc.log       # GC日志
```

#### 44.3 回答模板

> 参数-Xms/-Xmx设堆大小，-Xmn设年轻代，G1用-XX:MaxGCPauseMillis设停顿目标。PrintGCDetails+Xloggc看GC日志分析。G1是JDK11默认。

---

### 45. JDK版本差异？

#### 45.1 JDK 8

```
1. 接口default方法
2. Lambda表达式
3. Stream流
4. 新的日期API (java.time)
5. Nashorn JS引擎
6. Metaspace替代PermGen
```

#### 45.2 JDK 9-17

```
JDK 9: 模块化
JDK 10: var类型推断
JDK 11: ZGC正式、Ephemeron
JDK 14: switch表达式
JDK 16: record类
JDK 17: SealedClasses正式 LTS
```

#### 15.3 回答模板

> Java 8有重大更新Lambdanbsp>+Stream+接口default+新日期API+Lombda。JDK 9模块化，JDK10 var，JDK11 ZGC LTS，JDK17 Sealed类LTS。生产建议LTS（8/11/17/21）。

---

## 附录：面试追问

1. **为什么HashMap初始容量capacity必须是2^n？**
   - hash%capacity转换为hash&(capabilit-1)，位运算提升性能
   - 2^n确保散列均匀

2. **为什么ConcurrentHashMap不用synchronized？**
   - JDK8CAS+synchronized细粒度锁冲突节点
   - 锁住一个桶而不是整个Map
   - 并发读无锁

3. **Java 8为什么把永久代替换成Metaspace？**
   - 永久代是堆内存，受GC管理容易OOM
   - Metaspace是本地内存，不受堆GC影响可动态扩容

4. **JVM参数如何设置？**
   - 吞吐量优先：-XX:+UseParallelGC -XX:GCTimeRatio=19
   - 延迟优先：-XX:+UseG1GC -XX:MaxGCPauseMillis=200
   - 响应时间要设target

5. **finalize方法干什么的？**
   - GC前最后一次机会，但不确定执行
   - 显示调用System.runFinalization()可加速
   - 不推荐用，应改用try-with-resources

---

## 参考资料

- 《Effective Java》Joshua Bloch
- 《Java核心技术 卷I》
- JDK API Documentation

---

> 整理by Claude Code | Java面试高频100问