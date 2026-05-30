# 并发编程夺命连环100问——并发核心技术深度指南

> 本文档面向并发编程学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是并发？什么是并行？

#### 1.1 概念定义

```plaintext
并发（Concurrency）：
- 多个任务交替执行
- 共享CPU时间片
- 单核CPU即可实现
- 看起来像是同时执行

并行（Parallelism）：
- 真的同时执行
- 需要多核CPU
- 物理层面的同时
```

```java
// 并发：交替执行
Thread 1: ████░░████░░  (CPU时间片轮换)
Thread 2: ░░████████░

// 并行：同时执行
Thread 1: ████████████
Thread 2: ████████████
```

#### 1.2 区分例子

```java
// 并发：一个人处理多个请求
// 看似同时，实际交替快速处理
while (running) {
    process(requestQueue.take());
}

// 并发处理多个请求
// 并行：多个人同时处理
// 真正同时处理不同请求
executor.invokeAll(tasks); // 多线程执行
```

#### 1.3 回答模板

> 并发是多个任务交替使用CPU，看起来像同时执行，单核即可；并行是真的同时执行需要多核CPU。并发编程是为了充分利用CPU时间、提升系统吞吐量；并行是为了加速执行。多线程既可能并发也可能并行，取决于CPU核心数。

---

### 2. 为什么要用多线程？

#### 2.1 使用原因

```plaintext
多线程使用原因：
1. 利用多核CPU：并行计算加速
2. 提升CPU利用率：IO等待时可切换
3. 改善用户体验：后台任务不阻塞UI
4. 简化建模：分解复杂任务
5. 异步通信：非阻塞式处理
```

#### 2.2 场景举例

```java
// 场景1：利用多核（并行计算）
// 矩阵乘法分块计算
ForkJoinTask<Tile> task = new TileTask(matrix, start, end);
forkJoinPool.invoke(task);

// 场景2：IO等待不阻塞
// 爬虫并发下载
for (URL url : urls) {
    executor.submit(() -> download(url));
}

// 场景3：后台处理不阻塞用户
CompletableFuture.supplyAsync(() -> heavyComputation())
    .thenAccept(result -> ui.update(result));
```

#### 2.3 回答模板

> 多线程用于：1) 利用多核CPU提高计算速度；2) IO等待时切换到其他任务提高CPU利用率；3) 后台任务不阻塞主线程提升响应。异步编程、并行计算、并发请求处理都需要多线程。现代服务器的CPU i7 8核16线程，不用浪费了。

---

### 3. 线程的生命周期？

#### 3.1 状态定义

```java
public enum State {
    NEW,           // 创建但未start
    RUNNABLE,      // 可运行（就绪+运行中）
    BLOCKED,       // 阻塞等待锁
    WAITING,       // 无限等待（wait()/join()/park()）
    TIMED_WAITING, // 超时等待（sleep/milliseconds wait/...）
    TERMINATED     // 已结束
}
```

```plaintext
线程状态转换：
NEW ──start()──▶ RUNNABLE ──run()完成──▶ TERMINATED
                │
                ├─blocked──▶ BLOCKED ──获取锁──▶ RUNNABLE
                │
                ├─waiting──▶ WAITING ──notify──▶ RUNNABLE
                │
                ├─timed_wait──▶ TIMED_WAITING──▶ TIMED_WAITING超时──▶ RUNNABLE
                │
                └─wait()──▶ WAITING
```

#### 3.2 状态详解

```java
// NEW → 创建了但还没start
Thread t = new Thread(() -> {});
System.out.println(t.getState()); // NEW

// RUNNABLE → 正在运行或就绪队列
t.start();
System.out.println(t.getState()); // RUNNABLE

// BLOCKED → 没获取到synchronized锁
// WAITING → wait()/join()/LockSupport.park()
// TIMED_WAITING → sleep/milliseconds wait/...

// TERMINATED → run完成或异常终止
```

#### 3.3 回答模板

> 线程有6种状态：NEW（新建）、RUNNABLE（可运行，就绪+运行中）、BLOCKED（阻塞等锁）、WAITING（无限等待）、TIMED_WAITING（超时等待）、TERMINATED（终止）。sleep是TIMED_WAITING，wait()是WAITING，synchronized阻塞是BLOCKED。

---

### 4. 线程创建方式？

#### 4.1 创建方式

```java
// 方式1：继承Thread
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread");
    }
}
new MyThread().start();

// 方式2：实现Runnable
MyRunnable runnable = () -> System.out.println("Runnable");
new Thread(runnable).start();

// 方式3：实现Callable
Callable<String> callable = () -> "result";
FutureTask<String> task = new FutureTask<>(callable);
new Thread(task).start();
String result = task.get(); // 阻塞等待
```

#### 4.2 方式对比

| 方式 | 优点 | 缺点 |
|------|------|------|
| Thread | 简单 | 无法返回结果、无法抛异常 |
| Runnable | 灵活、可继承其他 | 同上 |
| Callable | 可抛异常、可返回 | 需配合FutureTask用 |

#### 4.3 回答模板

> 创建线程有三种方式：1）继承Thread；2）实现Runnable；3）实现Callable。推荐用Runnable/Callable因为还能继承其他类。Runnable无返回值抛异常 Callable可。 Callable+FutureTask可获取返回值和异常。

---

### 5. 线程优先级？

#### 5.1 优先级设置

```java
// 优先级1-10，默认5
Thread t = new Thread(() -> {});
t.setPriority(Thread.MAX_PRIORITY); // 10
t.setPriority(Thread.NORM_PRIORITY); // 5
t.setPriority(Thread.MIN_PRIORITY); // 1

// ForkJoinPool优先级
ForkJoinPool forkJoinPool = new ForkJoinPool(4);
forkJoinPool.execute(ForkJoinTask.adapt(() -> {})); // 默认优先级
```

#### 5.2 优先级特性

```plaintext
线程优先级特性：
1. 范围1-10（MIN/NORM/MAX）
2. 优先级高不代表一定先执行
3. 只是概率更高
4. 依赖操作系统实现
5. 某些系统不支持分级
```

#### 5.3 回答模板

> 线程优先级1-10默认5，高优先级只是概率更高不一定先执行。优先级依赖OS的具体实现，某些系统会忽略。ForkJoinPool默认是希望最大并行，用默认优先级即可。

---

### 6. 守护线程Daemon？

#### 6.1 守护线程定义

```java
// 设为守护线程
Thread daemonThread = new Thread(() -> {});
daemonThread.setDaemon(true);
daemonThread.start();

// 常见守护线程
// JVM自带：
// - GC线程
// - Finalizer线程
// - Reference Handler
// - 信号分发线程
```

```plaintext
守护线程 vs 用户线程：
- 用户线程：JVM会等待所有用户线程结束才退出
- 守护线程：JVM退出时不管守护线程是否结束
- 所有用户线程结束 → JVM退出 → 守护线程被强制终止
```

#### 6.2 注意事项

```java
// 注意：setDaemon必须在start前调用
Thread t = new Thread(() -> {});
t.setDaemon(true); // OK
t.start();

Thread t2 = new Thread(() -> {});
t2.start();
t2.setDaemon(true); // IllegalThreadStateException!
```

#### 6.3 回答模板

> 守护线程是JVM退出时不等待的线程，用于后台任务如GC。setDaemon必须在start前调用，否则抛IllegalThreadStateException。主线程结束不影响守护线程运行，但JVM退出时不管守护线程是否运行完都会终止。

---

### 7. ThreadLocal是什么？

#### 7.1 ThreadLocal定义

```java
// 每个线程独立的变量副本
ThreadLocal<Integer> counter = new ThreadLocal<>();

// 线程A设置
counter.set(1); // 线程A的副本=1

// 线程B读取
counter.get(); // 线程B返回null，不是1
```

```plaintext
ThreadLocal原理：
┌─────────────────────────────────────────┐
│ Thread LocalMap                     │
│  Thread.threadLocals              │
├─────────────────────────────────────────┤
│ Entry[key=ThreadLocal, value=obj] │ ← 弱引用
│ Entry[key=ThreadLocal, value=obj] │
│ ...                            │
└─────────────────────────────────────────┘
```

#### 7.2 使用场景

```java
// 场景1：数据库连接
class DatabaseUtil {
    private static ThreadLocal<Connection> conn = ThreadLocal.withInitial(
        () -> DriverManager.getConnection(url)
    );

    public static Connection get() { return conn.get(); }
}

// 场景2：Session
private static ThreadLocal<User> currentUser = ThreadLocal.withInitial(() -> null);
```

#### 7.3 内存泄漏问题

```java
// ThreadLocal内存泄漏原因：
// 1. Entry的key是弱引用，value是强引用
// 2. ThreadLocal被回收后，key=null，value还在
// 3. Thread存活，ThreadLocalMap就存在
// 4. 造成内存泄漏

// 解决：使用后remove
try {
    counter.set(1);
    doSomething();
} finally {
    counter.remove(); // 必须！
}
```

#### 7.4 回答模板

> ThreadLocal为每线程提供独立变量副本，线程间互不干扰。常用于数据库连接、Session等。但注意内存泄漏问题：在finally中remove()。key是弱引用可被回收，value是强引用导致泄漏。Alibaba transmittable-thread-local解决线程池传递问题。

---

### 8. 什么是竞态条件？

#### 8.1 竞态定义

```java
// 竞态条件示例
private int counter = 0;

// 线程A和B同时执行
// counter++ 不是原子操作！

// 实际执行（假设初始0）：
Thread A: read  counter -> 0
Thread B: read  counter -> 0  // 同样的时刻读到0！
Thread A: inc  0+1 = 1
Thread B: inc  0+1 = 1  // 应该变成2但实际1
Thread A: write 1 -> counter
Thread B: write 1 -> counter // 丢失一次更新！
```

#### 8.2 三大条件

```plaintext
竞态条件需要的三个条件：
1. 多个线程并发访问
2. 访问同一共享变量
3. 没有同步机制
```

#### 8.3 解决竞态

```java
// 解决方式：加锁
private int counter = 0;
private final Object lock = new Object();

// 方案1：synchronized
synchronized(lock) {
    counter++;
}

// 方案2：AtomicInteger
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();
```

#### 8.4 回答模板

> 竞态条件是多个线程并发访问共享资源产生正确性问题。i++不是原子操作（读-改-写三步）。解决需要同步：synchronized、Lock、AtomicInteger。

---

### 9. 什么是线程安全？

#### 9.1 线程安全定义

```java
// 线程安全
// 多个线程并发访问，某线程的执行结果始终正确

// 反例（线程不安全）
public class UnsafeCounter {
    private int count = 0;
    public void increment() { count++; }
    public int get() { return count; }
}

// 线程A: count++ (0→1)
// 线程B: count++ (应该是2，但可能1)
// 结果无法预测！
```

```plaintext
线程安全程度：
1. 绝对安全：所有情况都安全
2. 相对安全：正常场景安全，边界可能出问题
3. 不安全：多个线程必然出问题
```

#### 9.2 实现线程安全

```java
// 方案1：不可变对象
final class Counter {
    private final int value;
    public Counter(int v) { this.value = v; }
    public int value() { return value; } // getter，无setter
}

// 方案2：同步
synchronized increment/get;

// 方案3：原子类
AtomicInteger count = new AtomicInteger(0);

// 方案4：ThreadLocal
ThreadLocal<Integer> tl = new ThreadLocal<>();
```

#### 9.3 回答模板

> 线程安全指多个线程并发访问某对象的正确性不受影响。Thread Safety通过：1）同步synchronized/Lock；2）原子类AtomicInteger；3）不可变对象(final字段)；4）ThreadLocal隔离。基本原则能不用同步就不用，用lock-free。

---

### 10. 同步和异步？

#### 10.1 概念

```java
// 同步：顺序执行，等待完成
void syncCall() {
    Result r = doWork();  // 阻塞等待
    // 继续处理
}

// 异步：并行执行，callback处理结果
void asyncCall() {
    CompletableFuture.supplyAsync(() -> doWork())
        .thenAccept(result -> handle(result));
    // 不阻塞，继续执行
}
```

#### 10.2 同步场景

```plaintext
同步适合场景：
- 结果依赖
- 事务性
- 简单逻辑不必异步

异步适合场景：
- 不需要结果
- IO密集
- 解耦
- 微服务调用
```

#### 10.3 回答模板

> 同步是顺序执行等待结果，异步是开始执行后不等待结果（后台执行）。同步易调试但阻塞等待，异步吞吐高但复杂度高。IO密集用异步，计算密集用同步（需要CPU锁不住）。

---

## 第二章 锁机制篇（高频 ★★★★★）

### 11. synchronized原理？

#### 11.1 synchronized使用

```java
// 方法锁
synchronized void method() {
    // 临界区代码（一个时间只有一个线程）
}

// 代码块锁
void method() {
    synchronized(this) { // 可指定对象
        // 临界区
    }
}

// 静态方法锁（锁类）
synchronized static void staticMethod() {}
```

#### 11.2 底层原理

```java
// 编译后 bytecode
// monitorenter - 进入同步块
// monitorexit - 退出同步块

// 对应 Object Monitor（重量锁）
// 每个对象都关联一个Monitor
// Owner       - 拥有者线程
// WaitSet    - 等待队列
// Entrys    - 阻塞队列
```

```c
// monitorenter
// 1. 如果monitor count=0，进入，设置owner=thread，成功
// 2. 如果owner=thread，重入count++，成功
// 3. 否则进入Entrys阻塞

// monitorexit
// 1. count--，如果count>0，成功
// 2. 如果count=0，唤醒WaitSet中一个线程
```

#### 11.3 锁优化

```plaintext
锁膨胀升级（从无到有）：
无锁 → 偏向锁 → 轻量锁 → 重量锁

偏向锁：
- 首次CAS将thread ID写入对象头Mark
- 之后同线程直接进入

轻量锁：
- 自旋CAS获取Lock Record
- 失败多了自旋
- 成功获取

重量锁：
- OS mutex，线程阻塞
- 开销大
```

#### 11.4 回答模板

> synchronized基于对象头Monitor实现，有三种锁膨胀：无→偏向→轻量→重量。偏向锁首次CAS写thread ID，轻量锁CAS自旋，重量锁OS互斥。JDK 1.6后默认启用偏向锁，高并发可关闭。

---

### 12. volatile原理？

#### 12.1 volatile作用

```java
// volatile保证：
// 1. 可见性：修改立即刷新主存，其他线程立即看到
// 2. 有序性：禁止指令重排

volatile boolean running = true;

// 线程1
while (running) {
    doWork();
}

// 线程2
running = false; // 其它线程能看到变化
```

```c
// volatile关键指令
// StoreStore屏障（禁止之前的store和之后的store重排）
// LoadLoad屏障
// StoreLoad屏障（最常用）

// x86实现：
// 增加 LOCK 前缀指令
// 1. 刷新CPU缓存到内存
// 2. 使其他CPU缓存失效
```

#### 12.2 volatile不足

```java
// volatile不保证原子性
volatile int count = 0;

// 线程A、B同时执行 count++
// 假设count=0，两个线程同时读到
// 两个线程都执行 count++
// 期望：count=2  实际：count=1（丢失一次）
// count++ 不是原子！由-read-modify-write三步构成
```

#### 12.3 适用场景

```java
// volatile适用（一度一排）：
// 1. 状态标记
volatile boolean shutdown = false;

// 2. 单例DCL
class Singleton {
    private volatile static Singleton instance;
    public static Singleton get() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

#### 12.4 回答模板

> volatile保证可见和有序（一度一排），不保证原子。适用状态标记、DCL单例。需要原子用AtomicInteger/synchronized。volatile写会锁缓存flush stall，读会invalidate其他CPU缓存，所以一度一排。

---

### 13. 什么是CAS？

#### 13.1 CAS定义

```java
// Compare-And-Swap 比较并交换
// 当前位置值为期望则更新，否则不动作

// 模拟实现
boolean cas(int[] memory, int expect, int update) {
    if (memory[0] == expect) {
        memory[0] = update;
        return true;
    }
    return false;
}

// Java atomic实现
AtomicInteger ai = new AtomicInteger(0);
ai.compareAndSet(0, 1); // 如果是0则设为1
ai.getAndIncrement(); // 自增+cas循环
```

#### 13.2 ABA问题

```plaintext
ABA问题：
线程A：读A，改B 再改 回A
线程B：CAS成功

问题：实际被修改过但看不到

解决：AtomicStampedReference带版本号
AtomicStampedReference<int> ref =
    new AtomicStampedReference<>(value, stamp);
ref.compareAndSet(oldVal, newVal, oldStamp, newStamp);
```

#### 13.3 CAS优缺点

```plaintext
CAS优点：
- 非阻塞 lock-free
- 无死锁风险
- 提高并发性

CAS缺点：
- ABA问题
- 竞争大时反复失败，CPU空耗
- 只能保证一个变量原子
```

#### 13.4 回答模板

> CAS是Compare-And-Swap的CPU指令，实现无锁算法。A操作原子的compare-swap。ABA问题用带版本号的AtomicStampedReference。适合竞争不激烈的场景，竞争激烈反复自旋空耗CPU。

---

### 14. ReentrantLock原理？

#### 14.1 ReentrantLock使用

```java
ReentrantLock lock = new ReentrantLock();

// 加锁
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock(); // 必须finally释放
}

// trylock
if (lock.tryLock(10, TimeUnit.SECONDS)) {
    try {} finally {}
}
```

```java
// 公平锁（按等待时间）
ReentrantLock fairLock = new ReentrantLock(true);

// Condition条件
Condition condition = lock.newCondition();
condition.await(); // 等待
condition.signal(); // 唤醒一个
condition.signalAll(); // 全部唤醒
```

#### 14.2 对比synchronized

```java
| 特性 | synchronized | ReentrantLock |
|------|-------------|-------------|
| 语法 | 语言特性 | API |
| 公平 | 非公平 | 公平/非公平 |
| 响应中断 | 不能 | 可以tryLockInterruptibly |
| 尝试获取 | 不能 | trylock |
| 条件变量 | wait/notify | Condition×N |
| 超时 | 不能 | tryLock(timeout) |
| 性能 | JDK6优化后基本相当 | |
```

#### 14.3 实现原理

```java
// 基于AQS（AbstractQueuedSynchronizer）
// state表示锁重入次数
// 0：无锁
// >0：已持有锁，重入count

// 公平：检查等待队列是否有更早到来
// 非公平：直接CAS抢锁，不管等待队列
```

#### 14.4 回答模板

> ReentrantLock是显式Lock接口，比synchronized功能丰富：可tryLock、可中断、可以、公平锁。底层基于AQS和双向FIFO队列。推荐synchronized日常coding，ReentrantLock需要高级特性时用（如中断）。

---

### 15. ReadWriteLock原理？

#### 15.1 ReadWriteLock

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// 读锁
rwLock.readLock().lock();
try {} finally {}

// 写锁
rwLock.writeLock().lock();
try {} finally {}
```

```plaintext
读写锁规则：
- 读-读：不互斥（可并发）
- 读-写：互斥
- 写-写：互斥

适合读多写少场景（缓存等）
```

#### 15.2 实现细节

```java
// ReentrantReadWriteLock
// 按实现：
// - HoldCounter记录读锁重入次数
// -.firstReader firstReaderHoldCount 第一个读者
// - cachedHoldCounter 最后一个读者的计数器
// - readerShouldBlock() 判断是否应阻塞
```

#### 15.3 潜在问题

```plaintext
读写锁问题：
写锁饥饿：读线程持续涌入，写线程等待久

解决方案：
// 1. 写优先
ReadWriteLock lock = new ReentrantReadWriteLock(true);

// 2. 读的时候不允许新读
// readShouldBlock
```

#### 15.4 回答模板

> ReadWriteLock适合读多写少场景，可并发读但写独占。ReentrantReadWriteLock是实现。有公平模式但写仍可能饥饿（读不断涌入）。实际用ShardedReadWriteToken分散压力。

---

### 16. Condition原理？

#### 16.1 Condition使用

```java
Lock lock = new ReentrantLock();
Condition notEmpty = lock.newCondition();
Condition notFull = lock.newCondition();

int count = 0;
int capacity = 10;

// 生产者
lock.lock();
try {
    while (count == capacity) notFull.await();
    arr[count++] = item;
    notEmpty.signal();
} finally { lock.unlock(); }

// 消费者
lock.lock();
try {
    while (count == 0) notEmpty.await();
    item = arr[--count];
    notFull.signal();
} finally { lock.unlock(); }
```

#### 16.2 对比Object方法

| Object wait/notify | Condition await/signal |
|-------------------|--------------------|
| 必须先synchronized | 必须先Lock |
| 只能有一个Condition | 可创建多个 |
| wait释放锁 | await释放锁 |
| 同一mutex | 同一Lock |
| notify随机 | signal随机 |
| - | signallAll |

#### 16.3 回答模板

> Condition必须配合Lock使用，用signal/signalAll和await/signalAll唤醒await的线程。可以创建多个Condition实现不同条件，比Object wait/notify灵活。常用于生产者-消费者模式。

---

### 17. Semaphore原理？

#### 17.1 Semaphore使用

```java
// 限流，允许N个并发
Semaphore semaphore = new Semaphore(5);

// 获取许可
semaphore.acquire(); // 可中断
try {
    // 并发执行，最多5个
} finally {
    semaphore.release();
}

// 尝试获取
if (semaphore.tryAcquire(100, TimeUnit.MILLISECONDS)) {}
```

#### 17.2 原理

```plaintext
Semaphore基于AQS：
- state = 许可证数量
- acquire：state--
- release：state++

实现：
- 非公平：直接CAS抢
- fair（公平）：加入队尾
```

#### 17.3 应用场景

```java
// 数据库连接池限制
Semaphore dbPool = new Semaphore(10);
// 连接借出
dbPool.acquire();
try {
    Connection con = ds.getConnection();
} finally { dbPool.release(); }
```

#### 17.4 回答模板

> Semaphore信号量控制并发数，获取许可能进入，释放许可能继续。基于AQS state记许可数。常用于限流（数据库连接、API并发限制）。acquire可中断，tryAcquire可超时。

---

### 18. CountDownLatch原理？

#### 18.1 CountDownLatch使用

```java
// 等待N个任务完成
CountDownLatch latch = new CountDownLatch(10);

// 启动10个工作线程
for (int i = 0; i < 10; i++) {
    final int id = i;
    new Thread(() -> {
        doWork(id);
        latch.countDown(); // 完成一个
    }).start();
}

// 等待
latch.await(1, TimeUnit.MINUTES); // 带超时
// 或一直等
// latch.await();

// 主线程继续
```

#### 18.2 特点

```plaintext
CountDownLatch vs CyclicBarrier：
- CountDownLatch：一次性
- CyclicBarrier：可循环重用

CountDownLatch：
- 主线程await等待
- 工作线程countDown扣减
- 为0时唤醒await
- 不可复用
```

#### 18.3 回答模板

> CountDownLatch是一次性N减为0后await返回，用作等待N个任务完成。不可循环复用。CyclicBarrier是可重置的栅栏，让N个线程互相等待都到达后一起继续。

---

### 19. CyclicBarrier原理？

#### 19.1 CyclicBarrier使用

```java
// 互相等待到达
CyclicBarrier barrier = new CyclicBarrier(3);

// 3个线程
for (int i = 0; i < 3; i++) {
    final int id = i;
    new Thread(() -> {
        doPhaseOne(id);
        barrier.await(); // 等待其他
        doPhaseTwo(id);
    }).start();
}
```

```java
// 自定义动作（all arrived后执行）
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    // 所有线程到达后执行
    report();
});
```

#### 19.2 原理

```plaintext
CyclicBarrier基于ReentrantLock+Condition：

1. parties = N 记录需要聚齐的线程数
2. count = parties 剩余未到达
3. 每个线程await()：count--
4. count==0：signalAll所有线程
5. await成功，后续继续
6. 重置：count = parties（可复用！）
```

#### 19.3 回答模板

> CyclicBarrier让N个线程互相等待都到���后���起继续，和CountDownLatch一次性不同是CyclicBarrier可复用。构造函数可传 barrera llback 所有线程到达时执行的动作。Exchanger是两线程间的exchange。

---

### 20. LockSupport原理？

#### 20.1 LockSupport使用

```java
// 停车（阻塞当前线程）
LockSupport.park(); // 等unpark

// 带超时
LockSupport.parkNanos(java.time.Duration.ofSeconds(1));
LockSupport.parkUntil(long deadline);

// 唤醒
LockSupport.unpark(thread);
```

```java
// 与Object wait/notify对比：
// - 不需要先获取锁
// - unpark可在park之前
// - 不会抛InterruptedException
// - 更灵活简单
```

#### 20.2 原理

```plaintext
基于Parker：
- Permit（凭证）概念
- park()：如果permit>0则消耗permit返回；否则阻塞
- unpark()：给permit=1

permit初始为0 unpark后1
permit只能为0或1
```

#### 20.3 回答模板

> LockSupport是更灵活的park/unpark，比Object wait/notify简单：不用获取锁，可先unpark再park，凭证模式。是AQS、Concurrent同步类的底层核心。

---

## 第三章 并发工具篇（高频 ★★★★★）

### 21. ConcurrentHashMap原理？

#### 21.1 ConcurrentHashMap

```java
ConcurrentHashMap<Integer, User> map = new ConcurrentHashMap<>();

// 常用操作
map.putIfAbsent(k, v); // 原子里不存在的放进去
map.computeIfAbsent(k, Function); // 原子计算+存储
map.merge(k, v, (ov, nv) -> ov + nv); // 原子合并
map.getAndUpdate(k, UnaryOperator); // 先get再update

// 遍历（弱一致性）
for (Map.Entry<Integer, User> entry : map.entrySet()) {}
```

```java
// JDK 8+结构：
// Node<K,V>[] table（数组）
// 链表+红黑树（hash冲突时）
// Cas+synchronized保证并发安全
```

#### 21.2 关键方法源码

```java
/* JDK 8+ put流程 */
V put(K key, V value, boolean onlyIfAbsent) {
    // 1. hash计算
    // 2. tab = table; if null init
    // 3. f定位，if空CAS
    // 4. else synchronized (f)加锁
    // 5. 链表查找插入或树插入
}
```

#### 21.3 与Hashtable对比

| 特性 | ConcurrentHashMap | Hashtable |
|------|------------------|----------|
| 并发 | 无锁读+synchronized写 | 全synchronized |
| 性能 | 高 | 低 |
| null key | 只允许一个null | 不允许 |
| Iterator | 弱一致 | fail-fast |
| 递归 | computeIfAbsent用递归会栈溢出 | |

#### 21.4 回答模板

> ConcurrentHashMap是并发安全的Map。JDK8用Node数组+Cas+synchronized，红黑树转链表。原子方法：computeIfAbsent/merge/compute/reduce。强烈建议替代Hashtable和Collections.synchronizedMap。有迭代器弱一致性问题。

---

### 22. BlockingQueue原理？

#### 22.1 阻塞队列

```java
// 阻塞队列：队列满阻塞放，队列空阻塞取
BlockingQueue<String> queue = new LinkedBlockingDeque<>(10);

// 生产者
queue.put(item); // 队列满则阻塞
queue.offer(item, 1, SECONDS); // 超时返回false

// 消费者
String item = queue.take(); // 队列空则阻塞
String item = queue.poll(1, SECONDS); // 超时返回null

// drainTo一次取N个
List<String> list = new ArrayList<>();
queue.drainTo(list, 5);
```

#### 22.2 实现类

```java
// ArrayBlockingQueue 数组+ReentrantLock
// LinkedBlockingQueue 链表+ReentrantLock
// PriorityBlockingQueue 优先级+Condition
// DelayQueue      延迟队列（Comparable Delayed）
// SynchronousQueue 直接交给消费者，容量0
```

#### 22.3 生产者-消费者

```java
// 生产者
void produce() {
    while (true) {
        Item item = produceItem();
        queue.put(item);
    }
}

// 消费者
void consume() {
    while (true) {
        Item item = queue.take();
        process(item);
    }
}
```

#### 22.4 回答模板

> BlockingQueue是线程安全的阻塞队列，队列满阻塞offer，队列空阻塞poll��基��实现是ReentrantLock+Condition。ArrayBlockingQueue数组有界，LinkedBlockingQueue链表无界，SynchronousQueue直接交接。

---

### 23. ThreadPoolExecutor原理？

#### 23.1 线程池构建

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // corePoolSize 核心线程数
    10,                     // maximumPoolSize 最大线程数
    60L, TimeUnit.SECONDS, // keepAliveTime 存活时间
    new LinkedBlockingQueue<>(100), // queue队列
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy() // 拒绝策略
);

// 或者直接用快捷
ExecutorService es = Executors.newFixedThreadPool(5);
```

#### 23.2 执行流程

```java
/* 执行流程（execute）*/
1. 运行线程数 < corePoolSize？
   yes → 创建Worker执行
2. 任务加入队列成功？
   yes → 有Worker的话运行，否则拒绝
3. 运行线程数 < maximumPoolSize？
   yes → 创建Worker运行
4. 否则拒绝
```

#### 23.3 拒绝策略

| 策略 | 行为 |
|------|------|
| AbortPolicy | 抛RejectedExecutionException（默认） |
| DiscardPolicy | 静默丢弃任务 |
| DiscardOldestPolicy | 丢弃队列最老的任务 |
| CallerRunsPolicy | 调用者线程执行 |

#### 23.4 回答模板

> ThreadPoolExecutor线程池：core线程数+queue队列+max线程数+kpaliveness。 execute(Runnable)是核心方法，拒绝策略。Executors有便捷方法但生产建议手动new精确控制参数。线程数设置=CUP数×目标利用率×(1+I/O/CPU时间)。

---

### 24. ForkJoinPool原理？

#### 24.1 ForkJoinTask

```java
// ForkJoin加速分治计算
ForkJoinPool fjpool = new ForkJoinPool();

Long sum = fjpool.invoke(new SumTask(arr, 0, arr.length - 1));

// SumTask实现
class SumTask extends RecursiveTask<Long> {
    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            return Arrays.stream(arr, start, end+1).sum();
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(arr, start, mid);
        SumTask right = new SumTask(arr, mid+1, end);
        left.fork();
        return right.compute() + left.join();
    }
}
```

#### 24.2 Work-Steal算法

```plaintext
ForkJoinPool Work-Steal：
- 每个Worker有自己的双端队列 DQ
- 本地任务执行  pop()
- 无任务时 steal 另一DQ尾部 pop()
- 减少锁争用，尽量本地

Work-Stel vs ThreadPool：
- 适合分治递归计算
- 减少同步，work-steal减少抢
- JDK 7+ parallelStream底层
```

#### 24.3 回答模板

> ForkJoinPool用于分治递归计算，类似归并排序把任务分成小任务并行执行再用join合并。Work-Steal算法让worker先从自己队列拿，队列空才去steal别“人”的，减少锁竞争。parallelStream底层就是ForkJoinPool。

---

### 25. CompletableFuture原理？

#### 25.1 CompletableFuture

```java
// 异步编排
CompletableFuture<User> uf = CompletableFuture
    .supplyAsync(() -> fetchUser(id))
    .thenApply(User::enrichPrice)
    .thenCompose(user -> fetchDetails(user.getId()))
    .exceptionally(ex -> defaultUser());

// 并行执行joinall
CompletableFuture.allOf(array).join();
CompletableFuture.anyOf(array).join();
```

#### 25.2 关键方法

```java
// 编排：
thenApply(fn)       // 同步转异步 → R apply(T)
thenApplyAsync(fn)  // 异步 → 上一步完成then
thenCompose(fn)    // flatMap → thenApply + unwrap
thenCombine(f, fn)// 结合两个 → fn.apply(r1,r2)

// 异常：
exceptionally(fn)  // 有异常执行
handle(fn)        // 都执行

// 超时：
orTimeout(1, SECONDS)
completeOnTimeout(val, 1, SECONDS)
```

#### 25.3 原理

```plaintext
CompletableFuture原理：
- 依赖Completion包装任务
- 依赖图DAG拓扑排序
- callback push模式不用等
- 内部ForkJoinPool执行
- whenComplete/callback
```

#### 25.4 回答模板

> CompletableFuture是异步编程利器，thenApply/thenCompose/thenCombine等组合。thenApply同步转异步，Compose是flatMap带扁平化，Combine两者合并，anyOf任一/allOf全部。exceptionally/handle处理异常。

---

### 26. ThreadPoolExecutor饱和策略？

#### 26.1 饱和策略种类

```java
// AbortPolicy（默认）：抛异常
new ThreadPoolExecutor.AbortPolicy();

// DiscardPolicy：静默丢弃
new ThreadPoolExecutor.DiscardPolicy();

// DiscardOldestPolicy：丢弃最老
new ThreadPoolExecutor.DiscardOldestPolicy();

// CallerRunsPolicy：调用者执行
new ThreadPoolExecutor.CallerRunsPolicy();
```

```java
// CallerRunsPolicy缓冲
// 拒绝的任务在调用者线程执行,
// 调用者线程被用于执行被拒绝任务,
// 形成间接限流的效果
```

#### 26.2 自定义饱和策略

```java
// 自定义rejectedExecution
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    ...,
    (r, executor) -> {
        // 自己的处理逻辑
        // 记录日志
        // 放入DB queue稍后重试
        // 消息队列推回
    }
);
```

#### 26.3 回答模板

> 饱和策略有四种：Abort抛异常、Discard丢弃、DiscardOldest丢弃最老任务、CallerRunsPolicy调用者线程执行。第四种有缓冲效果：调用者被用于执行被拒绝任务，减轻瞬时压力。

---

### 27. 线程池大小如何配置？

#### 27.1 配置公式

```java
// CPU密集型
// 线程数 = CPU核心数 + 1
int cores = Runtime.getRuntime().availableProcessors();
int threads = cores + 1;

// IO密集型
// 线程数 = CPU核心数 × (1 + 平均等待时间/计算时间)
// 即 2~4倍CPU核心

// 混合型根据比例估算
```

```java
// 参数
-XX:ParallelGCThreads=4 // Parallel GC线程数
-XX:ConcGCThreads=2 // Concurrent GC线程数
XX:CICompilerCount=N // JIT编译线程数
```

#### 27.2 经验值

```java
// 经验公式
threads = numberOfCores × targetUtilization × (1 + waitTime / computeTime)

// 假设：
// CPU 4核，目标是90%利用率
// 如果是纯计算（wait几乎0）→ threads=4
// 如果等IO多（wait 3倍计算）→ threads=4×4=16
```

#### 27.3 回答模板

> 线程数配置根据任务类型：CPU密集型core+1、IO密集型core×(1+等待/执行)大约2-4倍、混合型根据占比。还要参考queue大小和内存。不宜过多，过多切换成本高且内存爆炸。

---

### 28. 线程池状态？

#### 28.1 线程池状态

```java
// 状态定义（int，高3位存储）
// RUNNING = -536870912 < 0
// SHUTDOWN = 0
// STOP = 1073741824
// TIDYING = 536870912
// TERMINATED = 1073741824

/* 状态转换 */
// RUNNING → SHUTDOWN：shutdown()
// RUNNING/STOP → SHUTDOWN：shutdownNow()
// SHUTDOWN → STOP：now中没元素
// STOP → TIDYING：queue空+worker空
// TIDYING → TERMINATED：terminated()完成
```

#### 28.2 状态检测

```java
// isRunning/isShutdown/isTerminated/isTerminating
ExecutorService pool = Executors.newFixedThreadPool(4);
pool.shutdown(); // 设置SHUTDOWN，不接受新任务

pool.isShutdown(); // true
pool.isTerminated(); // false
pool.isTerminating(); // true (还在关)

pool.awaitTermination(10, SECONDS); // 可等待完成
pool.shutdownNow(); // 中断STOP
```

#### 28.3 回答模板

> 线程池有RUNNING/SHUTDOWN/STOP/TIDYING/TERMINATED五个状态。shutdown不再接受新任务、等queue和worker完成；shutdownNow立即停止、尝试interrupt worker。awaitTermination(10, SECONDS)可等待完成。

---

### 29. ScheduledExecutorService原理？

#### 29.1 定时任务

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(4);

// 延迟执行
scheduler.schedule(() -> {}, 3, TimeUnit.SECONDS);

// 固定延迟执行（间隔��上��次完成后算）
scheduler.scheduleWithFixedDelay(() -> {},
    1, 3, TimeUnit.SECONDS);

// 固定频率执行（固定开始间隔）
scheduler.scheduleAtFixedRate(() -> {},
    1, 3, TimeUnit.SECONDS);
```

```java
// scheduleWithFixedDelay vs scheduleAtFixedRate：
// FixedDelay：上次完成→wait→执行
// FixedRate：上次开始→wait→执行（可能叠加！）
```

#### 29.2 实现原理

```java
// Delayed元素
class DelayedTask implements Runnable, Delayed {
    long nextExecuteTime; // 下次执行时间

    // 队列按nextExecuteTime排序
    // 线程取头，如果没有则await
}
```

#### 29.3 回答模板

> ScheduledExecutorService定时任务：schedule延迟，scheduleWithFixedDelay固定延迟（上一次完成算），scheduleAtFixedRate固定频率（可能重叠）。任务延迟可设1秒。

---

### 30. ExecutorService关闭？

#### 30.1 关闭方法

```java
ExecutorService pool = Executors.newFixedThreadPool(5);

// 1. shutdown() - 平滑关闭
pool.shutdown(); // 不接受新任务，等执行完
boolean b = pool.awaitTermination(30, SECONDS);
if (!b) pool.shutdownNow();

// 2. shutdownNow() - 强制关闭
List<Runnable> tasks = pool.shutdownNow(); // 立即返回
pool.shutdownNow();
pool.shutdownNow(); // 两次等于shutdown()

// 3. isShutdown/isTerminated
pool.isShutdown(); // 已经执行了shutdown否
pool.isTerminated(); // 是否已经完成
```

#### 30.2 最佳实践

```java
try {
    // pool.awaitTermination(60, SECONDS); 等完成
    // 一般做法
    pool.shutdown();
    if (!pool.awaitTermination(60, SECONDS)) {
        pool.shutdownNow();
    }
} finally {
    pool.shutdownNow(); // 最终取消
}

// 对于不可中中断任务
try { pool.shutdownNow(); } catch(Exception e) {}
```

#### 30.3 回答模板

> shutdown先新后等完成，shutdownNow立即停并返回未完成任务列表。最佳实践shutdown+AWAIT_TERMINATE+shutdownNow双保险。子任务cancel可能导致Task.get()抛CancellationException。

---

## 第四章 高级特性篇（中高频 ★★★★）

### 31. 虚似线程（JDK 19+）？

#### 31.1 虚拟线程

```java
// JDK 19+ virtual thread
Thread vt = Thread.startVirtualThread(() -> {});

// 或用ThreadFactory
ThreadFactory factory = Thread.ofVirtual().factory();
Thread vt2 = factory.newThread(() -> {});

// ExecutorService用虚似线程
ExecutorService ex = Executors.newVirtualThreadPerTaskExecutor();
```

#### 31.2 特点

```plaintext
Virtual Thread特点：
- 用户态线程（非OS线程）
- JVM调度，数千线程可同屏
- 一个物理线程可跑多个VT
- 阻塞不阻塞OS线程
- ThreadLocal同真线程一样用

与Platform线程区别：
- 创建：VK内存约200-300B，P约1-2MB
- 阻塞：VK不占线程，阻塞不OSThread
- 调度：VK JVM调度，PT OS调度
```

#### 31.3 使用场景

```java
// 好场景：IO阻塞多、大量并发
// web server handler, HTTP client

// 不适合：计算密集、长期持有
// 矩阵运算不适合
// 保持连接不适合

// 最佳：IO密集、高并发、短任务
public CompletableFuture<Response> handle(Request req) {
    // Virtual Thread处理
}
```

#### 31.4 回答模板

> 虚拟线程（VThread）是JDK 19+的用户态轻量线程，千级并发不占OS线程，阻塞时JVM切到其他VThread。一个Carrier线程可运行多个VThread。好场景IO阻塞-web服务/爬虫，不适合计算密集型。

---

### 32. 原子类有哪些？

#### 32.1 原子类概览

```java
// 基础类型
AtomicInteger, AtomicLong, AtomicBoolean


// 数组
AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray


// 引用
AtomicReference<V>, AtomicMarkableReference<V>,
AtomicStampedReference<V, Stamp>


// 属性更新器
AtomicIntegerFieldUpdater<T>,
AtomicLongFieldUpdater<T>,
AtomicReferenceFieldUpdater<T, V>


// 累加器（LongAdder JDK 8）
LongAdder, LongAccumulator,
DoubleAdder, DoubleAccumulator


// 偏移地址Updater
VarHandle
```

#### 32.2 LongAdder vs AtomicLong

```java
// 高并发性能：
// LongAdder  >>  AtomicLong
// 因为分段+cell

// LongAdder.sum()
// 返回当前累加总和

LongAdder adder = new LongAdder();
adder.add(1);
adder.increment();
long sum = adder.sum();

// 在计数/Cache场景用LongAdder
// AtomicLong cas竞争激烈
```

#### 32.3 VarHandle

```java
// VarHandle: 更强大的原子操作
// 支持：get/set/compareAndSet/getAndSet/add...

// 数组访问
VarHandle arrayVH = MethodHandles.arrayElementVarHandle(int[].class);

// 对象字段
VarHandle refVH = MethodHandles.lookup()
    .findVarHandle(User.class, "age", int.class);

// CAS操作
refVH.compareAndSet(user, 18, 19);
refVH.getAndAdd(user, 1);
```

#### 32.4 回答模板

> 原子类分基础AtomicInteger/Long/Boolean、数组Atomic*Array、引用AtomicReference、字段Updater。LongAdder在极端高并发性能优于AtomicLong，用分段Cell。VarHandle更通用可实现自定义原子操作。

---

### 33. 并发容器有哪些？

#### 33.1 List/Set/Queue

```java
// List: CopyOnWriteArrayList
List<String> list = new CopyOnWriteArrayList<>();
// 读无锁，写时COPY新数组，适合读多写少

// Set: CopyOnWriteArraySet
Set<String> set = new CopyOnWriteArraySet<>();

// Queue/Deque
ConcurrentLinkedQueue<T>, // 无界CAS
LinkedBlockingQueue<T>,     // 有界
LinkedBlockingDeque<T>,
ConcurrentLinkedDeque<T>
// Map: ConcurrentHashMap<K,V>
```

#### 33.2 Navigable

```java
// ConcurrentSkipListMap // 跳跃表
// Sorted: ConcurrentSkipListSet
ConcurrentSkipListSet<String> set = new ConcurrentSkipListSet<>();
set.lower("d"); // 小于
set.higher("b"); // 大于
set.subSet("a","d"); // 范围

// ConcurrentNavigableSetMap
ConcurrentNavigableSetMap<String, Integer> map = new ConcurrentSkipListMap<>();

// TreeMap线程不安全 → ConcurrentSkipListMap
```

#### 33.3 使用建议

```java
// List：
// - 读多写少：CopyOnWriteArrayList
// - 并发频繁：synchronizedList/writer锁，但性能低

// Map:
// - ConcurrentHashMap（非Sorted）
// - ConcurrentSkipListMap（Sorted/Navigable）

// Set:
// - CopyOnWriteArraySet（非Sorted）
// - ConcurrentSkipListSet（Sorted）
```

#### 33.4 回答模板

> 并发容器：List用CopyOnWriteArrayList适合读多写少，Set对应CopyOnWriteArraySet，Map用ConcurrentHashMap（���通���/ConcurrentSkipListMap（排序）/ConcurrentNavigableMap（跳跃表导航）。

---

### 34. Thread.interrupted vs Thread.currentThread().isInterrupted？

#### 34.1 中断方法

```java
Thread t = Thread.currentThread();

// 静态方法interrupted
boolean b = Thread.interrupted(); // 清空中断标记
// Thread.sleep/wait/join响应中断抛 InterruptedException

// 实例方法isInterrupted
t.isInterrupted(); // 不清除
t.interrupted(); // static，调用等价上者
```

#### 34.2 正确响应中断

```java
// 正确写法
void run() {
    try {
        while (!Thread.currentThread().isInterrupted()) {
            doWork();
        }
    } catch (InterruptedException e) {
        // 接收到中断信号，清除flag
        Thread.currentThread().interrupt(); // 重新设置
    }
}

// Callable+F utureCancel
future.cancel(true); // 可能失败
```

#### 34.3 回答模板

> Thread.interrupted()是静态，中断后会清除中断标志。 t.isInterrupted() 是实例不清除。如果在循环中catch到InterruptedException需要重新设置标志（因为被清除了），或者用isInterrupted判断。

---

### 35. StampedLock原理？

#### 35.1 StampedLock

```java
StampedLock sl = new StampedLock();

// 写锁
long stamp = sl.writeLock();
try {} finally { sl.unlockWrite(stamp); }

// 乐观读
long stamp = sl.tryOptimisticRead();
if (sl.validate(stamp)) {
    // 成功
} else {
    // 转悲观读
    stamp = sl.readLock();
    try {} finally { sl.unlockRead(stamp); }
}

// 读转写锁
stamp = sl.tryConvertToWriteLock(stamp);
```

```java
// 和ReentrantReadWriteLock对比：
// RW：read和write互斥
// Stamped：read返回stamp可验证
// OptimisticRead不阻塞写，但需要validate
```

#### 35.2 适用场景

```java
// 读多写少，且写不频繁
// 比RW更高效（读不阻塞写）

// 数据不一致可接受的场景
// 配合validate
```

#### 35.3 回答模板

> StampedLock是读写锁的优化，tryOptimisticRead返回stamp后用validate验证是否被修改，没修改继续，有则转悲观的readLock。比ReentrantReadWriteLock读并发性更好。适合读多写少的缓存场景。

---

### 36. Phaser原理？

#### 36.1 Phaser使用

```java
Phaser phaser = new Phaser(3); // 注册parties

// phases可以重复register/unregister

// 每个阶段线程arriveAndAwaitAdvance
for (int i = 0; i < 3; i++) {
    final int id = i;
    Thread t = new Thread(() -> {
        doPartA();
        phaser.arriveAndAwaitAdvance(); // 等待others

        doPartB();
        phaser.arriveAndAwaitAdvance(); // 等待again

        doPartC();
        phaser.arriveAndDeregister(); // 退出
    });
    t.start();
}
```

#### 36.2 特点

```plaintext
Phaser vs CountDownLatch/CyclicBarrier：
- 可动态register/unregister parties
- 可以重用的多阶段
- 每个phase可onAdvance回调
- 适合多轮同步
```

#### 36.3 回答模板

> Phaser是多阶段性屏障 parties注册+arriveAndAwaitAdvance +每个phase回调。适合多轮比赛、游戏关卡stage同步。register parties后，每arriveAndAwaitAdvance一次phase推进。

---

### 37. Exchanger原理？

#### 37.1 Exchanger

```java
Exchanger<String> exchange = new Exchanger<>();

// 线程A
String fromA = "A";
String getFromB = exchange.exchange(fromA);

// 线程B
String fromB = exchange.exchange("B");

// 结果：
// A收到"B"
// B收到"A"
```

#### 37.2 原理

```java
// 槽slot实现
// 1. 单槽 CAS改为配对者的引用
// 2. 两线程先后到→前者等地
// 3. 两者都到→交换并唤醒

// 用途：
// 交换缓冲buffer
//遗传算法交叉操作
```

#### 37.3 回答模板

> Exchanger用于两线程间交换数据，一个ex exhange后获��另��个的值。单槽CAS实现，一个线程先到等待另一个，后到则交换返回。可用于双向数据传输或遗传算法交叉。

---

### 38. 什么是伪共享（False Sharing）？

#### 38.1 伪共享原理

```java
// CPU缓存行：64字节
// 一个缓存行的多个变量被不同线程修改
// 导致缓存行失效

class LongAdder {
    // Cell加下Padding防止伪共享
    @Contended  // JDK8注解 自动填充
    static final class Cell {
        long value; // ...
    }
}
```

```c
Padding: 64字节填充
Cache Line: 0~63  [value][pad][value][pad]
Line修改失效其他 → 重新load → 性能下降
```

#### 38.2 解决

```java
// 1. JDK 8+ @Contended
@sun.misc.Contended
class Cell { long value; }

// 2. 手动填充
class PaddedValue {
    long value;
    long p1,p2,p3,p4,p5,p6; // 7个long=56字节+value=64
}

// 3. 线程数与CPU核心数的对齐
```

#### 38.3 回答模板

> 伪共享是不同线程修改同一缓存行的不同变量导致缓存行失效、重新加载的性能杀手。LongAdder/ConcurrentHashMap用@sun.misc.Contended注解自动填充64字节。关闭CPU缓存：-XX:-RestContended。

---

### 39. 什么是ABA问题？

#### 39.1 ABA问题的解决

```java
// 1. AtomicStampedReference带版本号
AtomicStampedReference<Integer> ref =
    new AtomicStampedReference<>(initialValue, stamp);

int stamp[] = new int[1];
Integer oldVal = ref.get(stamp);
ref.compareAndSet(oldVal, newVal, stamp[0], stamp[0]+1);

// 2. AtomicMarkableReference只标记是否更改过
AtomicMarkableReference<Integer> ref = new AtomicMarkableReference<>(initialValue, false);
```

#### 39.2 应用场景

```java
// 栈CAS防止ABA导致的数据结构错误
class CasStack<E> {
    AtomicReference<Node<E>> top = ...

    boolean push(E e) {
        Node<E> newNode = new Node<>(e);
        Node<E> oldTop;
        do {
            oldTop = top.get();
            newNode.next = oldTop;
        } while (!top.compareAndSet(oldTop, newNode));
    }

    // CAS+版本号防止ABA
}
```

#### 39.3 回答模板

> ABA问题是其他线程把A改成B又改回A导致CAS成功但实际被改过。AtomicStampedReference带stamp版本号解决，AtomicMarkableReference只标记是否改过。数据结构如栈/队列CAS要防ABA。

---

### 40. 并发Debug技巧？

#### 40.1 查看线程

```bash
# 线程堆栈
jstack -l <pid>

# 按CPU排序
top -Hp <pid>; printf "%x\n" <tid>
jstack <pid> | grep <thread hex>

# jConsole
jconsole <pid>

# VisualVM
visualvm <pid>

# Async-Profiler
async-profiler -d 30 -o flame.html -e cpu,alloc <pid>
```

#### 40.2 Arwas工具箱

```bash
# Arthas
java -jar arthas.jar

# 核心命令
dashboard          # 线程+内存+GC
thread -n 10      # Top10 CPU线程
jad ClassName     # 反编译
watch -x 2 'target method' '{params,returnObj}'
trace ClassName method '#cost > 10'
monitor -c 5 ClassName method
```

#### 40.3 回答模板

> 并发Debug用jstack查看线程，jvisualvm/Arthas监控。看哪个线程占用CPU高，找出该线程的堆栈。Arthas的watch/trace monitor是排查利器。

---

## 附录：面试追问

1. **线程池的最佳大小？**
   - 依据任务类型： IO密集>CPU×(1+IO/CPU)、CPU密集≈CPU+1
   - 还要参考队列大小、内存

2. **ConcurrentHashMap为什么不安全迭代？**
   - iterator是弱一致性，非fail-fast
   - 可能返回过期的数据但不会抛异常

3. **Callable如何获取返回？**
   - FutureTask包装
   - CompletableFuture.supplyAsync

4. **SynchronousQueue特点**
   - 容量0，生产者必须等消费者
   - 用于direct handoff线程池

5. **Thread优先级有用吗**
   - 大多数情况下没用，取决于OS
   - 不同OS策略可能忽略

---

## 参考资料

- 《Java并发编程实战》
- 《Java高并发编程详解》
- JUC源码
- openjdk源码

---

> 整理by Claude Code | 并发编程面试高频100问