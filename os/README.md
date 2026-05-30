# 操作系统夺命连环100问——操作系统核心技术深度指南

> 本文档面向操作系统学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是操作系统？为什么要学操作系统？

#### 1.1 操作系统定义

> 操作系统（Operating System，OS）是管理计算机硬件与软件资源的系统软件，它是用户与计算机硬件之间的接口，也是计算机系统的核心与基石。

```plaintext
计算机系统层次结构：
┌─────────────────────────────────────┐
│         用户/应用程序               │
├─────────────────────────────────────┤
│           操作系统                   │
├─────────────────────────────────────┤
│         计算机硬件                   │
└─────────────────────────────────────┘
```

#### 1.2 操作系统核心功能

```plaintext
进程管理：创建、调度、销毁进程
内存管理：分配/回收内存、虚拟内存
文件管理：文件存储、读取、权限
设备管理：驱动、I/O调度
安全管理：权限控制、资源保护
```

#### 1.3 回答模板

> 操作系统是管理计算机硬件和软件资源的系统软件，它向上对接应用程序，向下管理硬件资源。学习操作系统能帮助理解程序运行原理，解决性能问题和排查Bug，是后端开发的必备基础知识。

---

### 2. 进程和线程的区别？

#### 2.1 基本概念

```plaintext
进程（Process）：
- 正在执行的程序实例
- 拥有独立的地址空间
- 是资源分配的最小单位
- 至少包含一个线程

线程（Thread）：
- 进程内的执行单元
- 共享进程地址空间
- 是CPU调度的最小单位
- 也称为轻量级进程（LWP）
```

#### 2.2 核心区别

| 特性 | 进程 | 线程 |
|------|------|------|
| 资源拥有 | 独立地址空间 | 共享进程资源 |
| 开销 | 大（MB级别） | 小（KB级别） |
| 通信 | IPC复杂 | 直接共享内存 |
| 独立性 | 独立 | 依赖进程 |
| 创建速度 | 慢 | 快 |

#### 2.3 回答模板

> 进程是程序的一次执行实例，拥有独立地址空间，是资源分配的基本单位；线程是进程内的执行单元，共享进程地址空间，是CPU调度的基本单位。进程开销大但隔离性好，线程开销小但要注意同步问题。

---

### 3. 什么是系统调用？

#### 3.1 系统调用定义

> 系统调用（System Call）是用户态程序向内核请求服务的接口，是用户空间进入内核空间的唯一入口。

```plaintext
用户态 → 内核态转换：
┌─────────────┐     系统调用      ┌─────────────┐
│  用户态     │ ←─────────────→ │   内核态    │
│ (Ring 3)   │    触发中断      │  (Ring 0)  │
└─────────────┘                  └─────────────┘
```

#### 3.2 常见系统调用

```c
// 进程相关
fork()      // 创建进程
exec()      // 执行新程序
exit()      // 终止进程
wait()      // 等待子进程

// 文件相关
open()      // 打开文件
read()      // 读取文件
write()     // 写入文件
close()     // 关闭文件

// 内存相关
brk()      // 修改堆大小
mmap()      // 内存映射
munmap()    // 解除映射

// 通信相关
pipe()     // 创建管道
socket()   // 创建套接字
```

#### 3.3 回答模板

> 系统调用是用户程序请求内核服务的接口，是用户态进入内核态的唯一方式。常见系统调用包括进程管理的fork/exec、文件操作的open/read/write、内存管理的mmap等。系统调用比普通函数调用开销大，因为涉及用户态和内核态切换。

---

### 4. 用户态和内核态的区别？

#### 4.1 权限级别

```plaintext
Ring 特权级别：
┌─────────────┐
│  Ring 0     │ ← 内核态（最高权限）
├─────────────┤
│  Ring 1     │
├─────────────┤
│  Ring 2     │
├─────────────┤
│  Ring 3     │ ← 用户态（最低权限）
└─────────────┘

Linux只用Ring 0和Ring 3
```

#### 4.2 两者差异

| 特性 | 用户态 | 内核态 |
|------|-------|-------|
| 权限 | 受限 | 完全 |
| 访问内存 | 仅限自身 | 全部 |
| 访问硬件 | 不能 | 可以 |
| 系统资源 | 不能直接 | 可以 |
| 栈 | 用户栈 | 内核栈 |

#### 4.3 回答模板

> CPU通过Ring级别区分权限，用户态（Ring 3）权限受限只能访问自身地址空间，内核态（Ring 0）拥有最高权限可以访问所有内存和硬件。系统调用和异常会触发用户态到内核态的切换。

---

### 5. 僵尸进程和孤儿进程？

#### 5.1 概念定义

```plaintext
僵尸进程（Zombie）：
- 子进程已结束但父进程未调用wait()回收
- 仍保留PCB在进程表中
- 占用进程表条目（有限）

孤儿进程（Orphan）：
- 父进程先于子进程退出
- 被init进程（PID 1）收养
- init定期调用wait()回收
```

#### 5.2 问题影响

```plaintext
僵尸进程危害：
- 占用进程表条目
- 进程表满则无法创建新进程
- 无法被kill杀死

孤儿进程处理：
- 自动被init收养
- 无危害
```

#### 5.3 回答模板

> 子进程结束后父进程未回收就会变成僵尸进程，会占用进程表导致资源泄漏。孤儿进程是父进程先退出的子进程，会被init收养没有问题。避免僵尸进程需要在父进程中调用wait()或使用信号处理。

---

### 6. 什么是死锁？产生条件？

#### 6.1 死锁定义

> 死锁（Deadlock）是指两个或多个进程在执行过程中，因争夺资源而造成相互等待的现象，若无外力干预，它们都将无法推进下去。

```plaintext
死锁例子：
进程A：持有锁1，等待锁2
进程B：持有锁2，等待锁1
→ 互相等待，形成死锁
```

#### 6.2 产生条件（Coffman条件）

```plaintext
1. 互斥条件：资源一次只能被一个进程使用
2. 占有并等待：进程已持有一个资源，又请求另一个资源
3. 不可抢占：已分配资源不能被强制抢走
4. 循环等待：形成等待环路
```

#### 6.3 回答模板

> 死锁是多个进程互相等待对方持有的资源而无法推进的现象。产生死锁需要四个条件：互斥、占用并等待、不可抢占、循环等待。解决死锁可以破坏其中一个条件，如使用try-lock而不是阻塞等待。

---

### 7. 哲学家就餐问题？

#### 7.1 问题描述

```plaintext
5个哲学家、5根筷子每位哲学家左右各一根
思考时不能拿筷子的哲学家需要等待左右筷子都空闲才能进餐
容易产生死锁
```

#### 7.2 解决方案

```c
// 方案1：限制同时拿筷子的人数
// 最多4个人同时拿

// 方案2：同时拿起左右筷子
// 用互斥锁保护一次拿两根

// 方案3：奇偶交替
// 奇数先拿左边，偶数先拿右边

// 方案4：服务生解法
// 服务生控制谁能拿筷子
```

#### 7.3 回答模板

> 哲学家就餐问题是经典的死锁问题，5个人每人左右有一根筷子容易死锁。解决方案有限制同时拿筷子的人数、一次拿完两根筷子、或用服务生协调。实际开发中用锁排序和try-lock避免死锁。

---

### 8. 乐观锁和悲观锁？

#### 8.1 概念对比

```plaintext
悲观锁（Pessimistic Lock）：
- 每次操作前先加锁
- 认为数据会被修改
- 如：数据库行锁、synchronized

乐观锁（Optimistic Lock）：
- 操作前后检查版本
- 允许并发冲突发生
- 如：CAS、版本号字段
```

#### 8.2 适用场景

| 锁类型 | 适用场景 | 缺点 |
|--------|----------|------|
| 悲观锁 | 写多并发高 | 阻塞、性能低 |
| 乐观锁 | 写少并发低 | 重试开销 |

#### 8.3 回答模板

> 悲观锁是先加锁再操作，适合并发高的写场景但性能低；乐观锁是先操作再验证，适合并发低的写场景。数据库用版本号或CAS实现乐观锁，编程用synchronized或ReentrantLock实现悲观锁。

---

### 9. CPU调度算法有哪些？

#### 9.1 调度算法分类

```plaintext
批处理调度：
- FCFS：先来先服务
- SJF：最短作业优先
- HRRN：最高响应比优先

交互式调度：
- 时间片轮转（RR）
- 优先级调度
- 多级反馈队列

实时调度：
- Earliest Deadline First
- Rate Monotonic Scheduling
```

#### 9.2 算法对比

| 算法 | 优点 | 缺点 |
|------|------|------|
| FCFS | 简单公平 | 短作业等待长 |
| SJF | 最优平均等待 | 需要预估 |
| RR | 公平响应及时 | 上下文切换多 |
| MLFQ | 兼顾长短作业 | 实现复杂 |

#### 9.3 回答模板

> 常见CPU调度算法有：先来先服务FCFS简单公平、最短作业优先SJF最优平均等待、时间片轮转RR适合交互、优先级调度、多级反馈队列MLFQ综合表现好。Linux用CFS完全公平调度器，实时要求用EDF。

---

### 10. 页面置换算法？

#### 10.1 常见算法

```plaintext
最佳置换（OPT）：
- 置换未来最久不被访问的
- 理论最优，无法实现

先进先出（FIFO）：
- 置换最早进入的
- 简单但可能Belady异常

最近最少使用（LRU）：
- 置换最久未使用的
- 近似最优

时钟算法（Clock/NRU）：
- 用访问位近似LRU
- 实际常用
```

#### 10.2 Belady异常

```plaintext
Belady异常：
增加物理页帧数导致缺页率上升
FIFO算法可能出现
```

#### 10.3 回答模板

> 页面置换算法用于虚拟内存满了时选择淘汰页面。OPT是理论最优不可能实现，FIFO简单可能有Belady异常，LRU接近最优但开销大，Clock是实际常用的折中方案，用访问位近似LRU。

---

## 第二章 进程管理篇（高频 ★★★★★）

### 11. 进程状态有哪些？

#### 11.1 五状态模型

```plaintext
进程状态：
┌─────────┐
│  新建    │ ← 创建
├─────────┤
│  就绪    │ → 运行
├─────────┤
│  运行    │ → 就绪/等待
├─────────┤
│  等待    │ → 就绪
├─────────┤
│  终止    │ ← 完成
└─────────┘
```

#### 11.2 七状态模型

```plaintext
增加挂起状态：
- 就绪挂起
- 阻塞挂起
用于内存紧张时换出
```

#### 11.3 回答模板

> 进程有五种基本状态：创建、就绪、运行、等待、终止。就绪是等待CPU、等待是等待I/O或资源。Linux还有TASK_STOPPED和TASK_TRACED等特殊状态。

---

### 12. 进程调度器？

#### 12.1 Linux调度器演进

```plaintext
O(n) 调度器：遍历所有进程
↓
O(1) 调度器：优先级数组
↓
CFS调度器：完全公平调度
```

#### 12.2 CFS原理

```plaintext
CFS（Completely Fair Scheduler）：
- 用红黑树管理Runnable进程
- virtual runtime越小越优先
- 按时间片轮转保证公平
- nice值影响权重
```

#### 12.3 回答模板

> Linux早期用O(n)和O(1)调度器，现在用CFS完全公平调度器。CFS用红黑树管理进程，virtual runtime小的优先执行，保证公平的同时支持nice值调整优先级。

---

### 13. 进程间通信方式？

#### 13.1 IPC方式

```plaintext
同一主机：
- 管道（Pipe）
- 命名管道（FIFO）
- 消息队列
- 共享内存
- 信号量

不同主机：
- Socket网络通信
```

#### 13.2 对比

| 方式 | 速度 | 用途 |
|------|------|------|
| 管道 | 快 | 父子进程 |
| 消息队列 | 中 | 解耦 |
| 共享内存 | 最快 | 高频 |
| Socket | 慢 | 跨机器 |

#### 13.3 回答模板

> 进程间通信方式有管道、消息队列、共享内存、信号量等。Linux还支持IPC System V和POSIX两套接口。共享内存是最快的但需要同步，Socket可用于跨主机通信。

---

### 14. 管道和消息队列区别？

#### 14.1 管道

```c
// 匿名管道
int pipe(int fd[2]);
read(fd[0]); write(fd[1]);

// 命名管道
mkfifo("/tmp/myfifo", 0666);
```

```plaintext
特点：
- 单向流动
- 无结构的字节流
- 亲缘进程间通信
- 容量有限（pipe capacity）
```

#### 14.2 消息队列

```c
// 操作
msgget(); msgsnd(); msgrcv(); msgctl();
```

```plaintext
特点：
- 有结构的数据块
- 消息类型标识
- 可以非亲缘进程
- 持久存在
```

#### 14.3 回答模板

> 管道是单向字节流，需要亲缘关系-capacity有限；消息队列是有结构的消息队列可以非亲缘进程用持久存在。管道像水流，消息队列像带标签的包裹。

---

### 15. 共享内存如何实现？

#### 15.1 使用方式

```c
// 创建
shmget(key, size, IPC_CREATE|0666);

// 附着的地址
void *shmat(int shmid, const void *shmaddr, int shmflg);

// 分离
shmdt(const void *shmaddr);

// 控制
shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

#### 15.2 同步问题

```plaintext
共享内存本身不加锁！
需要信号量或原子操作同步

常见模式：
sem_t *sem = sem_open("/mysem", O_CREAT, 0666, 1);
```

#### 15.3 回答模板

> 共享内存通过shmget创建shmat附着使用，是最快的IPC方式但本身不提供同步，需要配合信号量或原子变量实现同步。Linux还支持mmap映射文件实现共享内存。

---

### 16. 信号量是什么？

#### 16.1 概念

```plaintext
信号量（Semaphore）：
- 计数器
- P操作（wait/down）：减1，如<0则阻塞
- V操作（signal/up）：加1，唤醒等待

二值信号量：值为0或1（类似互斥锁）
计数信号量：值>1
```

#### 16.2 使用

```c
// 初始化
sem_init(&sem, 0, 1);  // 二值

// P操作
sem_wait(&sem);

// V操作
sem_post(&sem);
```

#### 16.3 回答模板

> 信号量是带有P/V操作的计数器，用于同步。P操作用于获取资源（如<0阻塞），V操作用于释放资源唤醒等待。可以用作互斥锁（二值信号量）或限制并发数。

---

### 17. fork的实现原理？

#### 17.1 fork过程

```c
pid_t fork(void);
// 返回两次：父进程返回子进程PID，子进程返回0
```

```plaintext
fork步骤：
1. 分配PCB和内核栈
2. 复制父进程地址空间（Copy-On-Write）
3. 复制文件描述符（引用计数+1）
4. 设置子进程资源统计
5. 返回PID给父进程
```

#### 17.2 Copy-On-Write

```plaintext
fork时不立即复制全部内容！
父子进程共享页面，写时才复制

优点：快速创建进程
缺点：复杂度增加
```

#### 17.3 回答模板

> fork是创建新进程的系統调用，返回两次：父进程获得子进程PID，子进程获得0。为了快速fork采用Copy-On-Write，写时才复制内存页。vfork是共享地址空间的古uttle，目前已被淘汰。

---

### 18. exec系列函数？

#### 18.1 函数族

```c
// 六个exec函数
execl(path, arg0, ..., NULL);
execlp(file, arg0, ..., NULL);
execle(path, argv, envp);

execv(path, argv);
execvp(file, argv);
execve(path, argv, envp);
```

#### 18.2 执行新程序

```plaintext
exec和fork常配合使用：
fork()后
  父进程：wait子进程
  子进程：exec新程序

效果：创建新进程执行新程序
```

#### 18.3 回答模板

> exec系列函数用于执行新程序，替换当前进程的代码和数据。fork+exec是创建新进程执行新程序的经典模式。l结尾用列表参数，v结尾用数组参数，p会自动搜索PATH，e可以传环境变量。

---

### 19. 进程池和线程池？

#### 19.1 概念

```plaintext
进程池：
- 预先创建多个进程
- 复用进程处理请求
- 减少fork开销
- 隔离性好（独立地址空间）

线程池：
- 预先创建多个线程
- 复用线程处理任务
- 减少创建开销
- ���享地址空间
```

#### 19.2 对比

| 特性 | 进程池 | 线程池 |
|------|--------|--------|
| 开销 | 大 | 小 |
| 隔离 | 好 | 差 |
| 通信 | 复杂 | 简单 |
| 适用 | CPU密集 | I/O密集 |

#### 19.3 回答模板

> 进程池和线程池都是预先创建一批worker，避免频繁创建销毁的开销。进程池隔离性好但开销大，线程池开销小但共享内存需要注意线程安全。阿里的Dubbo框架就使用了线程池。

---

### 20. 守护进程Daemon？

#### 20.1 定义

```plaintext
守护进程：
- 后台运行的系统服务
- 不受终端控制
- 生命周期长
- 通常以d结尾命名
```

#### 20.2 创建步骤

```c
// 1. fork创建子进程
pid = fork();
if(pid > 0) exit(0);

// 2. 设置新的会话
setsid();

// 3. 再次fork（可选，防止重新获取终端）
// pid = fork();
// if(pid > 0) exit(0);

// 4. 改变工作目录
chdir("/");

// 5. 重设文件权限
umask(0);

// 6. 关闭标准输入输出
close(STDIN_FILENO);
close(STDOUT_FILENO);
close(STDERR_FILENO);
```

#### 20.3 回答模板

> 守护进程是后台运行的系统服务，不受登录Shell影响。创建步骤包括fork脱离父进程、setsid创建新会话、chdir改工作目录、umask重设权限、关闭stdio。Linux的服务如sshd、nginx都是守护进程。

---

## 第三章 内存管理篇（高频 ★★★★★）

### 21. 分页和分段？

#### 21.1 分页（Paging）

```plaintext
固定大小的页面：
- 页大小：4KB（常见）
- 页表：虚拟页→物理页框
- 无外部碎片
- 有内部碎片

虚拟地址结构：
┌───────────────┬───────────────┐
│  页号(PGNO)  │ 页内偏移(OFF) │
└───────────────┴───────────────┘
```

#### 21.2 分段（Segmentation）

```plaintext
可变大小的段：
- 代码段、数据段、堆段、栈段
- 每个进程有自己的段表
- 符合程序逻辑

虚拟地址结构：
┌─────────────┬───────────┐
│  段号     │  段内偏移  │
└─────────────┴───────────┘
```

#### 21.3 回答模板

> 分页是把虚拟内存分成固定大小4KB的页，无外部碎片；分段是把程序逻辑分成代码/数据/堆/栈段，符合程序逻辑。实际系统常用两者结合：段页式管理。

---

### 22. 什么是虚拟内存？

#### 22.1 虚拟内存概念

```plaintext
虚拟内存：
- 程序看到的内存地址
- 不等于物理内存
- 需要MMU translation

优点：
- 扩大地址空间
- 内存隔离
- 按需分配
```

#### 22.2 工作原理

```plaintext
MMU（Memory Management Unit）：
虚拟地址 → 物理地址转换
  ↓
页表查找 → TLB Cache
  ↓
缺页异常 → 从磁盘加载
```

#### 22.3 回答模板

> 虚拟内存让程序看到比实际物理内存更大的地址空间，通过MMU实现虚拟地址到物理地址的翻译。程序无需关心物理内存分布，可以实现内存隔离和按需加载。Linux的swap就是虚拟内存的一部分。

---

### 23. 什么是缓存？

#### 23.1 缓存层次

```plaintext
计算机缓存体系：
┌──────────────────┐
│  CPU寄存器      │ ← 1cycle
├──────────────────┤
│  L1 Cache      │ ← 4cycle
├──────────────────┤
│  L2 Cache     │ ← 10cycle
├──────────────────┤
│  L3 Cache     │ ← 20cycle
├──────────────────┤
│  主存         │ ← 100cycle
├───��─��────────────┤
│  SSD/磁盘     │ ← 10000cycle
└──────────────────┘
```

#### 23.2 缓存策略

```plaintext
写直达（Write Through）：
- 写cache同时写内存
- 简单但慢

写回（Write Back）：
- dirty标记，写出时机换出
- 复杂但快

预取（Prefetch）：
- 预测访问提前加载
```

#### 23.3 回答模板

> 缓存是基于局部性原理的高速存储，分L1/L2/L3多层，访问速度依次降低。写策略有Write Through和Write Back，开发中要注意缓存失效和一致性问题。

---

### 24. 什么是TLB？

#### 24.1 TLB定义

```plaintext
TLB（Translation Lookaside Buffer）：
- MMU的地址翻译缓存
- 虚拟页号→物理页框号
- 全相联或组相联映射

命中率很高（>99%）
```

#### 24.2 工作流程

```plaintext
1. CPU发出虚拟地址
2. TLB查找虚拟页号
3. 命中：直接获得物理页框
4. 未命中：查页表，再填充TLB
```

#### 24.3 回答模板

> TLB是MMU的地址翻译缓存，存储最近使用的虚拟页到物理页的映射。TLB命中很快，未命中需要查页表再填充。TLB大小有限需要好的局部性才能高效。

---

### 25. 什么是物理内存管理？

#### 25.1 分配方式

```plaintext
伙伴系统（Buddy System）：
- 2的幂次方大小分配
- 相邻小块可合并
- 内部有碎片

SLAB分配器：
- 基于伙伴系统
- 缓存常用对象大小
- 减少碎片
- 内核常用
```

#### 25.2 简述

#### 25.3 回答模板

> 物理内存管理有伙伴系统和SLAB分配器。伙伴系统按2的幂分配可合并，SLAB在此基础上缓存常用对象减少碎片。Linux内核用SLAB，高端malloc用伙伴系统。

---

### 26. 什么是内存泄漏？如何检测？

#### 26.1 内存泄漏

```plaintext
内存泄漏（Memory Leak）：
- 申请的内存在程序结束时未释放
- 或失去引用路径
- 长期运行会耗尽内存

常见原因：
- malloc/new后忘记free/delete
- 异常路径未释放
- 循环引用（智能指针）
```

#### 26.2 检测工具

```bash
# Valgrind
valgrind --leak-check=full ./program

# AddressSanitizer
gcc -fsanitize=address -g program.c

# pmap
pmap -X <pid>
```

#### 26.3 回答模板

> 内存泄漏是申请内存未释放导致内存逐渐耗尽，常见于malloc后未free、智能指针循环引用。检测工具Valgrind和AddressSanitizer，可以用shared_ptr+weak_ptr打破循环。

---

### 27. 什么是缓冲区溢出？

#### 27.1 原理

```c
char buffer[10];
strcpy(buffer, "this is long");
复制长度超过缓冲区
→ 覆盖相邻内存
```

#### 27.2 危害

```plaintext
栈溢出利用：
- 覆盖返回地址
- 注入恶意代码
- 控制程序执行

常见攻击：
- 格式化字符串
- 堆溢出
```

#### 27.3 防护措施

```plaintext
编译保护：
- Stack Canary
- NX bit
- ASLR

代码层面：
- strncpy代替strcpy
- 边界检查
- 安全的字符串函数
```

#### 27.4 回答模板

> 缓冲区溢出是写入数据超过缓冲区边界，会覆盖相邻内存可能被利用注入代码。现代系统用Stack Canary、NX、ASLR防护，开发中要用安全函数strncpy/sprintf_snprintf。

---

### 28. 什么是内存屏障？

#### 28.1 概念

```plaintext
Memory Barrier：
- 阻止CPU重排指令
- 保证可见性
- 编译 barrier + CPU barrier

类型：
- Full barrier (mfence)
- Read barrier (lfence)
- Write barrier (sfence)
```

#### 28.2 使用场景

```c
// 多线程
volatile int flag;
__asm__ __volatile__("mfence" ::: "memory");
// 确保store先于load
```

#### 28.3 回答模板

> 内存屏障用于多线程同步，阻止CPU和编译器重排指令确保可见性。C++11的memory_order和 atomic线程库封装了这些。

---

### 29. malloc实现原理？

#### 29.1 allocator

```plaintext
glibc malloc：
- ptmalloc2实现
- 分为arena和heap
- bins管理空闲块
- 使用mmap和brk
```

#### 29.2 分配策略

```c
// 小块（<128KB）
// 从arena的bins分配
// fastbin/ybin/bin

// 大块（>=128KB）
// mmap直接分配
```

#### 29.3 回答模板

> malloc使用ptmalloc2实现，小块从bins用brk分配，大块用mmap直接分配。有fastbin优化小额分配，top chunk是备用空间。线程有各自的arena避免锁竞争。

---

### 30. 什么是OOM Killer？

#### 30.1 定义

```plaintext
OOM（Out Of Memory）Killer：
- 内存不足时杀进程
- 基于oom_score选择
- 优先杀高score进程
- /proc/<pid>/oom_score_adj
```

#### 30.2 影响和调优

```plaintext
vm.overcommit_memory：
- 0：启发式
- 1：允许超量
- 2：不允许超量
```

#### 30.3 回答模板

> OOM Killer在内存耗尽时根据oom_score杀掉进程，可以通过/proc/<pid>/oom_score_adj调整。游戏行业常关overcommit防止突发OOM。

---

## 第四章 文件系统篇（中高频 ★★★★）

### 31. 文件系统有哪些？

#### 31.1 常见文件系统

```plaintext
本地文件系统：
- ext4：Linux主流
- xfs：高性能、大文件
- btrfs：COW、快照
- NTFS：Windows

分布式文件系统：
- HDFS：大数据
- CephFS
- GlusterFS

网络文件系统：
- NFS
- SMB/CIFS
```

#### 31.2 回答模板

> 常用本地文件系统ext4、xfs、btrfs，分布式如HDFS、Ceph，网络文件系统如NFS、SMB。选择看场景：ext4通用、xfs大文件、btrfs快照、HDFS大数据。

---

### 32. inode和block？

#### 32.1 inode

```plaintext
inode（索引节点）：
- 文件元数据
- 文件ID标识
- 包含：大小、时间、权限、block指针
- 固定大小（128-256字节）

一个文件一个inode！
```

#### 32.2 block

```plaintext
block（数据块）：
- 文件实际存储单位
- 一般4KB
- 多个block组成文件
- 按需分配
```

#### 32.3 回答模板

> inode存储文件元数据（大小、时间、权限、block指针），一个文件对应一个inode。block是实际存储数据的4KB块，按需分配。所以删除文件快（清inode）而写入慢（找空闲block）。

---

### 33. 硬链接和软链接？

#### 33.1 硬链接

```bash
ln source_file hard_link
```

```plaintext
特点：
- 同一inode
- 不能跨分区
- 不能链接目录
- 删除源不影响
```

#### 33.2 软链接（符号链接）

```bash
ln -s source_file soft_link
```

```plaintext
特点：
- 独立inode
- 可以跨分区
- 可以链接目录
- 源删除失效
```

#### 33.3 回答模板

> 硬链接是同一inode的不同文件名，不能跨分区和链接目录；软链接是独立inode指向路径，可以跨分区和目录但源删除后失效。用ln和ln -s创建。

---

### 34. 什么是RAID？

#### 34.1 RAID级别

```plaintext
RAID 0：条带，性能高无冗余
RAID 1：镜像，可靠
RAID 5：奇偶校验，分布式
RAID 6：双重校验，更高可靠
RAID 10：0+1组合
```

#### 34.2 特点对比

| 级别 | 冗余 | 性能 | 盘利用率 |
|------|------|------|----------|
| 0 | 无 | 高 | 100% |
| 1 | 有 | 中 | 50% |
| 5 | 有 | 高 | (n-1)/n |
| 10 | 有 | 高 | 50% |

#### 34.3 回答模板

> RAID是磁盘阵列提高性能和可靠性。RAID0条带化高性能无冗余、RAID1镜像、RAID5奇偶校验分布式、RAID10是组合。数据库用RAID10或RAID5，家庭NAS用RAID5。

---

### 35. 什么是VFS？

#### 35.1 VFS定义

```plaintext
VFS（Virtual File System）：
- 虚拟文件系统抽象层
- 统一接口
- 屏蔽底层差异

支持多种文件系统：
ext4、NTFS、NFS、proc...
```

#### 35.2 结构

```plaintext
应用 → VFS → 具体文件系统
                ↓
           ext4/NTFS/NFS
```

#### 35.3 回答模板

> VFS是Linux的虚拟文件系统层，提供统一的文件操作接口，底层实际文件系统(ext4/NTFS/NFS)只需实现这些接口。这就是为什么Linux能支持多种文件系统。

---

### 36. 文件描述符？

#### 36.1 概念

```plaintext
文件描述符（FD）：
- 一个整数
- 进程级资源
- 指向内核的文件表项
- 标准FD：0 stdin、1 stdout、2 stderr
```

#### 36.2 文件表结构

```plaintext
每个进程：
  FD表 → 指向 → 文件表项 → inode
              (引用计数)
```

#### 36.3 回答模板

> 文件描述符是进程打开文件的整数索引，指向内核中的文件表项。每个进程有独立的FD表，fork会复制FD表（引用计数+1），close减少引用计数。ulimit -n查看最大FD数。

---

### 37. 日志文件系统？

#### 37.1 什么是 journaling

```plaintext
日志文件系统：
- 先写日志再更新元数据
- 崩溃后快速恢复
- 保证一致性

ext4/xfs都有日志
```

#### 37.2 工作流程

```plaintext
1. write to log（事务开始）
2. 日志写入
3. commit（标记完成）
4. metadata更新
5. 日志释放
```

#### 37.3 回答模板

> 日志文件系统先写日志再更新元数据，崩溃时可通过日志快速恢复一致性。ext4和xfs都支持日志，后者性能更好。关闭日志可提高性能但增加风险。

---

### 38. 什么是配额管理？

#### 38.1 Quota

```bash
# 启用配额
mount -o usrquota,grpquota /dev/sda1

# 设置
edquota -u username
```

#### 38.2 类型

```plaintext
用户配额：限制用户使用空间
组配额：限制组使用空间
软限制：宽限期
硬限制：立即生效
```

#### 38.3 回答模板

> 配额度制用户或组的磁盘使用，可以用edquota设置软硬限制。企业Linux服务器常用配额管理防止单用户占满磁盘。 quota -uvs username 查看使用情况。

---

### 39. 什么是NFS？

#### 39.1 网络文件系统

```plaintext
NFS（Network File System）：
- Unix/Linux共享
- RPC协议
- 无状态设计
- v4版本改进

挂载：
mount -t nfs server:/share /mnt
```

#### 39.2 回答模板

> NFS是Unix/Linux的网络文件系统，通过RPC协议共享目录。v4版本改进变成有状态，性能更好。企业内网常用NFS做共享存储。

---

### 40. 什么是SSD优化？

#### 40.1 SSD特性

```plaintext
SSD vs HDD：
- 无机械寻道
- 随机读写快
- 写入有寿命（PE）
- 擦除单位大（block）
```

#### 40.2 优化策略

```plaintext
1. TRIM命令（让SSD知道无效块）
2. 对齐分区（4K对齐）
3. 少用swap
4. ext4 discard选项
```

#### 40.3 回答模板

> SSD无机械部件随机读写快，但写入有寿命需要TRIM。分区要对齐4K， mount用discard开启自动TRIM，少用swap。HDD不需要优化。

---

## 第五章 并发与其他（中高频 ★★★★）

### 41. 什么是原子操作？

#### 41.1 原子性

```plaintext
原子操作（Atomic Operation）：
- 不可中断
- 要么全做要么不做
- 保证可见性

C11 atomics：
#include <stdatomic.h>
atomic_fetch_add(&counter, 1);
```

#### 41.2 回答模板

> 原子操作是不可中断的操作，要保证原子性需要CPU支持或锁保护。C11以后 stdatomic.h 提供原子变量类型和操作。竞态条件需要原子操作或锁保护。

---

### 42. 什么是指令重排？

#### 42.1 编译和CPU重排

```plaintext
指令重排（Instruction Reordering）：
- 编译器优化
- CPU执行优化
- 不改变单线程语义

���题���
- 多线程可见性
- 破坏内存序
```

#### 42.2 内存序

```c
// C++ memory order
atomic<int> x = 0;
atomic<int> y = 0;

// 释放语义
x.store(1, memory_order_release);

// 获取语义
if (x.load(memory_order_acquire) == 1) {
    // 保证看到x之前的写
}
```

#### 42.3 回答模板

> 指令重排是编译器和CPU为优化进行的重排，不改变单线程语义但会破坏多线程可见性。用内存序memory_order_release/acquire或锁保护。

---

### 43. 什么是CAS？

#### 43.1 CAS概念

```c
// Compare-And-Swap
bool CAS(int* addr, int old, int new) {
    if (*addr == old) {
        *addr = new;
        return true;
    }
    return false;
}
```

#### 43.2 ABA问题

```plaintext
ABA问题：
- 值从A变B再变A
- CAS成功但实际被修改过

解决方法：版本号Stamp
```

#### 43.3 回答模板

> CAS是.Compare-And-Swap原子指令，用于无锁算法。但有ABA问题（中间被其他线程修改后又改回来），可用带版本号的Stamp解决。 Java的AtomicStampedReference就是这原理。

---

### 44. 什么是ThreadLocal？

#### 44.1 概念

```c
// C++ thread_local
thread_local int tlsVar = 0;
// 每个线程独立副本

// Java
ThreadLocal<T> tl = new ThreadLocal<>();
tl.set(value);
tl.get();
```

#### 44.2 内存问题

```plaintext
ThreadLocal内存泄漏：
- threadLocals map强引用key
- key是弱引用，value是强引用
- thread结束但value未remove会泄漏

解决：
- 使用后remove
- 建议用static包装
```

#### 44.3 回答模板

> ThreadLocal为每个线程提供独立变量副本，是线程隔离的。Java中要特别注意cleanup，否则可能内存泄漏。建议在线程入口处remove或用static包装。

---

### 45. 什么是条件变量？

#### 45.1 概念

```c
pthread_mutex_t mu = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// 等待
pthread_mutex_lock(&mu);
while (!ready) pthread_cond_wait(&cond, &mu);
pthread_mutex_unlock(&mu);

// 唤醒
pthread_mutex_lock(&mu);
ready = true;
pthread_cond_signal(&cond);
pthread_mutex_unlock(&mu);
```

#### 45.2 回答模板

> 条件变量用于线程等待特定条件，需要配合互斥锁使用。wait要放while循环内判断条件，防止虚假唤醒。signal唤醒一个、broadcast唤醒所有。

---

### 46. 什么是Barrier？

#### 46.1 概念

```c
// C++ barrier
#include <barrier>
std::barrier syncPoint(N);

syncPoint.arrive_and_wait(); // 等待所有线程到达
```

#### 46.2 场景

```plaintext
Barrier（栅栏）：
- 所有线程等待彼此到达
- 然后继续执行

vs 条件变量：
- 条件变量：不确定次数等待
- 栅栏：确定次数（线程数）
```

#### 46.3 回答模板

> Barrier让N个线程都到达栅栏点后一起继续执行，适合需要所有worker完成才能进入下一阶段的场景。CountDownLatch是一次性的，CyclicBarrier可循环。

---

### 47. 什么是生产者消费者？

#### 47.1 模式

```c
// 队列作为缓冲
Queue<T> queue;
Mutex mu;
Condition full, empty;

// 生产者
lock(full);
queue.push(item);
signal(empty);

// 消费者
lock(empty);
auto item = queue.pop();
signal(full);
```

#### 47.2 无锁实现

```plaintext
Disruptor：
- 环形缓冲区
- 缓存行填充
- 内存屏障
- 高性能消息队列
```

#### 47.3 回答模板

> 生产者-消费者是经典并发模式，用队列解耦生产和消费。无锁实现用环形缓冲区+CAS，注意缓存行伪共享。企业级用Disruptor，高吞吐场景如日志采集。

---

### 48. 什么是读写锁？

#### 48.1 概念

```c
std::shared_mutex rwMu;
std::shared_lock read(rwMu); // 多读
std::unique_lock write(rwMu); // 独占��
```

#### 48.2 设计要点

```plaintext
RwLock要点：
- 读者之间不互斥
- 写者独占
- 写者优先防饥饿

实现：
- 读者计数
- 写者标记
- 队列
```

#### 48.3 回答模板

> 读写锁适合读多写少场景，读者之间并行但写者独占。设计要注意写者优先级防止饥饿。Linux读写锁pthread_rwlock，Java用ReentrantReadWriteLock。

---

### 49. 什么是自旋锁？

#### 49.1 概念

```c
// 自旋锁
while (atomic_swap(lock, 1) == 1) {
    // busy wait
}
unlock(0);
```

#### 49.2 特点

```plaintext
自旋锁（Spin Lock）：
- 线程原地忙等
- 适用于短临界区
- 避免了线程切换

缺点：
- CPU空耗
- 不适合长临界区
```

#### 49.3 回答模板

> 自旋锁是让线程在锁外busy wait，适很短时间的临界区可以避免线程切换开销。长临界区应用mutex让线程阻塞。自旋+阻塞组合是常见的自适应锁。

---

### 50. 什么是信号Signal？

#### 50.1 Unix信号

```plaintext
信号（Signal）：
- 异步通知机制
- 类UNIX通知进程

常见信号：
SIGINT (Ctrl+C)：SIGTERM：SIGKILL：SIGSEGV：
SIGCHLD：子进程结束
```

#### 50.2 处理

```c
// signal handler
signal(SIGINT, handler);
// 或sigaction更可靠
struct sigaction sa;
sa.sa_handler = handler;
sigaction(SIGINT, &sa, NULL);
```

#### 50.3 回答模板

> 信号是Unix的异步通知机制，常见有SIGINT（Ctrl+C）、SIGTERM（优雅终止）、SIGKILL（强制终止）、SIGSEGV（段错误）。用signal或sigaction注册处理函数。注意handler中只调用async-signal-safe函数。

---

## 第六章 实战与优化（中高频 ★★★★）

### 51. 如何排查CPU 100%？

#### 51.1 排查步骤

```bash
# 1. top找到java进程
top

# 2. 按H转为线程视图
top -Hp <pid>

# 3. 找到高CPU线程ID
# 转16进制
printf "%x\n" <tid>

# 4. jstack打印线程堆栈
jstack <pid> | grep <hex tid>
```

#### 51.2 常用命令

```bash
# CPU占用
top -b -n 1 -p <pid>

# 线程
ps -Lp <pid>

# 各种诊断
vmstat 1
mpstat 1
iostat 1
sar -u 1
```

#### 51.3 回答模板

> CPU 100%排查用top找到进程后转线程视图，找高CPU线程ID后jstack dump看都在做什么。常见死循环、频繁GC、正则递归等。

---

### 52. 如何排查内存问题？

#### 52.1 内存分析方法

```bash
# 堆内存
jmap -heap <pid>
jmap -histo <pid>
jmap -dump:format=b,file=heap.bin <pid>

# MAT分析dump
# https://www.eclipse.org/mat/

# OOM时生成dump
-XX:+HeapDumpOnOutOfMemoryError
```

#### 52.2 GC分析

```bash
# 查看GC日志
-Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps

# jstat
jstat -gcutil <pid> 1000

# GCEasy等在线分析
# https://gceasy.io/
```

#### 52.3 回答模板

> 内存问题用jmap分析堆，jstat观察GC，OOM时加HeapDumpOnOutOfMemoryError生成dump用MAT分析。GC日志用GCEasy分析关注FGC频率和GC时间。

---

### 53. 什么是抖动？

#### 53.1 Thrashing

```plaintext
Thrashing（抖动）：
- 频繁page in/out
- CPU忙于I/O
- 系统效率急剧下降

原因：
- 内存不足
- 进程过多
```

#### 53.2 解决

```plaintext
处理：
- 加内存
- 减少进程
- 优化代码
- 调整vm.swappiness
```

#### 53.3 回答模板

> 抖动是内存严重不足时大量换入换出，导致CPU忙于I/O而非计算。解决方法加内存、减少并发、优化代码，降低swappiness可以缓解。

---

### 54. 什么是惊群效应？

#### 54.1 Thundering Herd

```plaintext
惊群效应（Thundering Herd）：
- 多进程/线程等待同一事件
- 事件发生时全部被唤醒
- 但只有一个能处理

常见于：
- epoll wait
- accept
```

#### 54.2 解决

```plaintext
方案：
- Nginx/Lighttpd：用锁+单进程accept
- Linux 2.6：SO_REUSEPORT分到多个监听socket
- 防止全部wake up
```

#### 54.3 回答模板

> 惊群效应是多人等的同一资源就绪时全部唤醒但是只有一人能用到，导致无效上下文切换。Nginx用锁保证单accept，Linux后续版本用SO_REUSEPORT解决。

---

### 55. 什么是协程？

#### 55.1 概念

```plaintext
Coroutine：
- 用户态线程
- 自主调度
- 共用内核线程

vs 线程：
- 创建开销：KB vs MB
- 切换成本：用户态 vs 内核态
- 调度：主动 vs 抢占
```

#### 55.2 实现

```go
// Go goroutine
go func() { }()

// Java virtual thread (JDK19+)
Thread.startVirtualThread(() -> {});

// Spring WebFlux响应式
```

#### 55.3 回答模板

> 协程是用户态线程，创建和切换成本极低，比内核线程轻量。Go的goroutine、Java的Virtual Thread都是协程实现，适合IO密集型任务大幅提高并发。

---

### 56. Epoll原理？

#### 56.1 I/O多路复用

```c
// epoll_create
int epfd = epoll_create(1);

// epoll_ctl
epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &event);

// epoll_wait
epoll_wait(epfd, events, maxevents, timeout);
```

#### 56.2 工作模式

```plaintext
LT（Level Trigger）：
- 水平触发
- 不读走会重复通知
- 默认，兼容old code

ET（Edge Trigger）：
- 边沿触发
- 只通知一次
- 需非阻塞fcntl设置
```

#### 56.3 回答模板

> Epoll是Linux高效I/O多路复用，LT模式数据未读走会重复通知，ET模式只通知一次需要用非阻塞IO。select/poll是轮询，epoll是事件触发可支持百万连接，Nginx/Redis都用epoll。

---

### 57. select/poll/epoll对比？

#### 57.1 对比

| 特性 | select | poll | epoll |
|------|--------|------|-------|
| 最大FD | 1024 | 无上限 | 无上限 |
| 复杂度 | O(n) | O(n) | O(1) |
| 触发方式 | 轮询 | 轮询 | 事件 |
| 内存 | 创建Bitmap | 创建数组 | 红黑树 |
| FD复制 | 全量 | 全量 | MD |
| 适用范围 | 小规模 | 中规模 | 百万级 |

#### 57.2 回答模板

> select/poll/epoll都是I/O多路复用，select有FD限制1024/poll无限制/epoll高效O(1)。 select/poll是轮询模式每次全量遍历，epoll基于回调、红黑树管理，只返回就绪FD所以高效。百万连接用epoll。

---

### 58. 什么是零拷贝？

#### 58.1 实现原理

```c
// sendfile
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);

// 传统read+write
read(fd, buf, n); write(sock, buf, n);

// sendfile
sendfile(sock, fd, NULL, n);
```

#### 58.2 零拷贝流程

```plaintext
传统（4次copy）：
disk → kernel_buf → user_buf → socket_buf → NIC

零拷贝（1次copy）：
disk → kernel_buf → NIC

对比：
数据从4次copy降到1次copy
CPU从全程参与变成不参与
```

#### 58.3 回答模板

> 零拷贝指数据在内核空间流转不经过用户空间，sendfile用DMA把磁盘数据直接发到网卡跳过用户态。Kafka/RocketMQ消息队列高性能就靠零拷贝。 Java用transferTo()底层就是sendfile。

---

### 59. 什么是大_page/HugePages？

#### 59.1 Transparent HugePage

```bash
# 查看
cat /sys/kernel/mm/transparent_hugepage/enabled
#[always] madvise never

# 禁用（对Java有时必要）
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

#### 59.2 标准大页

```bash
# 配置
sysctl -w vm.nr_hugepages=200

# JVM
-XX:+UseLargePages -XX:LargePageSizeInBytes=2m
```

#### 59.3 回答模板

> HugePages是大内存页可以减少TLB miss，对大内存Java程序有用但因为内存布局问题效果有限。更重要的是保持GC停顿可控、内存布局合理。Transparent HugePage在某些场景会导致内存碎片化问题。

---

### 60. 什么是内核参数调优？

#### 60.1 网络参数

```bash
# 文件描述符
ulimit -n

# TCP参数
sysctl -w net.core.somaxconn=65535
sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sysctl -w net.ipv4.tcp_fin_timeout=15
sysctl -w net.ipv4.tcp_keepalive_time=300
sysctl -w net.ipv4.tcp_keepalive_probes=2
sysctl -w net.ipv4.tcp_keepalive_intvl=15
```

#### 60.2 内存参数

```bash
# swap
sysctl -w vm.swappiness=10

# TCP内存
sysctl -w net.ipv4.tcp_rmem="4096 87380 6291456"
sysctl -w net.ipv4.tcp_wmem="4096 16384 6291456"

# 文件
sysctl -w fs.file-max=2000000
sysctl -w fs.nr_open=2000000
```

#### 60.3 回答模板

> Linux内核参数调优包括：文件描述符限制ulimit -n、TCP参数net.core.somaxconn/net.ipv4.tcp_*、内存vm.swappiness、网络core参数。生产环境网络相关要调优防止连接问题。

---

## 附录：面试追问

1. **64位和32位系统区别？**
   - 地址空间：2^32=4GB vs 2^64=16EB
   - CPU指令集不同
   - 指针大小不同
   - 高内存访问能力

2. **如何查看进程内存？**
   - /proc/<pid>/status、pmap -X、smaps
   - jstat -gcutil、jmap

3. **什么是/proc文件系统？**
   - 内核向外暴露信息的虚拟文件系统
   - /proc/cpuinfo、/proc/meminfo、/proc/<pid>/*

4. **什么是OOM？**
   - Out of Memory，根据oom_score杀进程
   - vm.overcommit_memory控制

5. **什么是cgroups？**
   - Control Groups：资源配额
   - Docker/K8s底层技术
   - cpu/memory/blkio/网络限制

---

## 参考资料

- 《操作系统概念》
- 《深入理解计算机系统》
- Linux内核文档
- man pages

---

> 整理by Claude Code | 操作系统面试高频100问