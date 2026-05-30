# MySQL夺命连环100问——MySQL DBA与后端开发深度指南

> 本文档面向MySQL学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」→「实际怎么用」四个维度讲解。

---

## 第零章 入门铺垫 —— MySQL基础概念

### 0.1 为什么MySQL是后端必修课？

```
互联网数据存储方案：
┌─────────────────┐
│ 热数据 → MySQL   │ ← 需要高速读写 + 事务支持
│ 温数据 → Redis  │ ← 缓存热点
│ 冷数据 → 磁盘   │ ← 归档存储
└─────────────────┘
```

99%的互联网应用核心数据都存MySQL！不会MySQL = 后端半残。

### 0.2 MySQL学习路径

```
第一阶段：基础（CRUD、索引、事务）
    ↓
第二阶段：优化（EXPLAIN、慢查询、索引优化）
    ↓
第三阶段：架构（主从复制、分库分表、高可用）
    ↓
第四阶段：运维（备份恢复、监控、故障处理）
```

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 什么是MySQL？为什么要用MySQL？

#### 1.1 MySQL定义

> MySQL是瑞典MySQL AB公司开发的关系型数据库，现属于Oracle开源项目。使用SQL作为操作语言，99%互联网公司都在用。

#### 1.2 为什么要用MySQL？（背下来）

```
✓ 开源免费：中小企业首选
✓ 社区活跃：文档全、问题好找
✓ 性能优秀：InnoDB引擎经多年优化
✓ 功能完善：事务、主从、集群
✓ 生态丰富：Navicat、MySQL Workbench
```

#### 1.3 回答模板

> MySQL是世界上最流行的开源关系型数据库，使用SQL语言操作。它被广泛使用主要有几个原因：1）开源免费，成本低；2）社区成熟，遇到问题容易找到答案；3）默认的InnoDB引擎支持事务和行级锁，性能足够；4）功能和稳定性都已非常完善，可以满足各种业务需求。我们公司核心业务数据都存在MySQL。

---

### 2. MySQL的架构是什么样的？

#### 2.1 四层架构图解

```
┌───────────────────────────────────────────┐
│          连接层（Connectors）              │
│   JDBC、Golang、Python连接组件             │
└──────────────────┬──────────────────────┘
                   ↓
┌───────────────────────────────────────────┐
│            服务层（Server）               │
│  连接管理 │ 缓存查询 │ SQL解析 │ SQL优化  │
└──────────────────┬──────────────────────┘
                   ↓
┌───────────────────────────────────────────┐
│         存储引擎层（Engine）              │
│    InnoDB │ MyISAM │ Memory             │
└──────────────────┬──────────────────────┘
                   ↓
┌───────────────────────────────────────────┐
│            物理文件层                    │
│   .frm │ .ibd │ .MYD │ .MYI             │
└───────────────────────────────────────────┘
```

#### 2.2 SQL执行流程

```
SQL → 连接器 → 解析器 → 预处理器 → 优化器 → 执行器 → 存储引擎

详细：
1. 连接器：建立连接、权限验证
2. 查询缓存：8.0前有此功能，已有
3. 解析器：词法/语法分析，生成AST
4. 预处理器：验证表/字段存在
5. 优化器：选择索引、JOIN顺序
6. 执行器：调用存储引擎获取数据
```

#### 2.3 回答模板

> MySQL整体是四层架构：1）连接层负责处理客户端连接；2）服务层包括SQL解析、优化、执行；3）存储引擎是插件式的，InnoDB默认；4）物理文件层是最终的数据存储。一条SQL的执行流程是：连接器建立连接→解析器解析SQL→优化器生成执行计划→执行器调用存储引擎→返回结果。

---

### 3. MySQL有哪些存储引擎？有什么区别？

#### 3.1 四大存储引擎对比

| 特性 | InnoDB | MyISAM | Memory | Archive |
|------|-------|--------|--------|---------|
| 事务 | ✅ | ❌ | ❌ | ❌ |
| 行锁 | ✅ | ❌ | ✅ | ✅ |
| 外键 | ✅ | ❌ | ❌ | ❌ |
| 全文索引 | ✅ | ✅ | ❌ | ❌ |
| 存储限制 | 64TB | 256TB | RAM | 无 |
| MVCC | ✅ | ❌ | ❌ | ❌ |

#### 3.2 InnoDB vs MyISAM

```
InnoDB（默认，必选）：
  - 支持事务，行级锁
  - MVCC并发控制
  - 崩溃自动恢复
  - 适用：99%场景

MyISAM（老系统可能用）：
  - 查询快（无事务开销）
  - 全文索引完善
  - 崩溃无法恢复
  - 适用：纯读静态表
```

#### 3.3 引擎选择

```sql
-- 查看引擎
SHOW ENGINES;

-- 创建表指定引擎
CREATE TABLE t (...) ENGINE=InnoDB;

-- 修改引擎
ALTER TABLE t ENGINE=InnoDB;
```

**无脑选InnoDB！**

#### 3.4 回答模板

> MySQL有多种存储引擎，最常用的是InnoDB。它支持事务和行级锁，MVCC实现高并发，崩溃后可以自动恢复，是我们公司的默认选择。MyISAM查询快但不支持事务，适合那种几乎不更新的纯静态表。老系统可能还在用MyISAM，但现在新项目一律用InnoDB就对了。

---

## 第二章 索引与查询（高频 ★★★★★）

### 4. 什么是索引？为什么要用索引？

#### 4.1 索引概念

> 索引就像书的目录，可以快速定位，不用逐行扫描。

```
无索引：O(n) 全表扫描
  SELECT * FROM user WHERE name='张三';
  → 从第1行开始，逐行比对100万行

有索引：O(log n) 索引查找
  CREATE INDEX idx_name ON user(name);
  → 索引树 → 直接定位 → 只需几次I/O
```

#### 4.2 索引代价

```
✓ 查快
✓ 但有代价：
  - 占空间（可能比数据还大）
  - 写变慢（要维护索引）
  - 不是 万 能：Cardinality 低建=白建
```

#### 4.3 回答模板

> 索引是一种数据结构，用来加速数据查找，就像书的目录。没有索引是逐行扫描，有索引可以直接定位。在百万级数据量下这个差距是几百上千倍的。建索引的原则是：WHERE/JOIN/ORDER BY涉及的列、区分度高的列。主键索引会自动建。

---

### 5. MySQL有哪些索引类型？

#### 5.1 按数据结构分

| 索引类型 | 说明 | 场景 |
|---------|------|------|
| B+Tree索引 | 默认，平衡树 | 绝大多数 |
| Hash索引 | 哈希表 | 等值查询 |
| 全文索引 | Full-Text | 大段文本 |
| RTREE | 空间索引 | 地理坐标 |

#### 5.2 按逻辑分

| 索引类型 | 说明 |
|---------|------|
| 主键索引 | PK，唯一非空 |
| 唯一索引 | UNIQUE，唯一 |
| 普通索引 | INDEX，普通 |
| 全文索引 | FULLTEXT，文本 |

#### 5.3 最左匹配原则（重点！）

```sql
-- 索引 (a, b, c)
CREATE INDEX idx_a_b_c ON user(a, b, c);

-- 能否用索引？
✓ WHERE a=1       -- 用
✓ WHERE a=1 AND b=2  -- 用
✓ WHERE a=1 AND b=2 AND c=3  -- 用
✓ WHERE a=1 AND c=3    -- 用a，但c用不到
✗ WHERE b=2       -- 不用（跳过a）
✗ WHERE c=3       -- 不用（跳过a、b）
```

**原理**：联合索引像电话号码簿，必须按顺序查。

#### 5.4 回答模板

> MySQL索引分几种：按数据结构有B+Tree（默认）、Hash、全文；按逻辑有主键、唯一、普通、全文。
> 重点是联合索引的最左匹配原则：索引(a,b,c)，WHERE必须从a开始匹配，跳过前面的列就都用不到。比如只查b不用索引，这就是为什么有时候明明建了索引却不生效。

---

### 6. InnoDB的索引结构（B+Tree）怎么工作？

#### 6.1 B+Tree vs 红黑树

```
红黑树：高度 log₂N
  100万数据 → 高度约20
  每次I/O读1节点 → 20次I/O

B+Tree：分支多，树更矮
  100万数据 → 高度约2-3
  每节点16KB → 几次I/O

结论：B+Tree树矮 I/O少，所以快
```

#### 6.2 主键索引结构

```
主键索引叶子节点：
┌─────────────────────────────────┐
│  Key → 完整行数据               │
│  每个叶子节点存完整的一行数据     │
└─────────────────────────────────┘

查主键：O(1) → 最快
```

#### 6.3 二级索引结构

```
二级索引叶子节点：
┌─────────────────────────────────┐
│  Key → 主键值（不是完整数据）    │
└─────────────────────────────────┘

查非主键列：
  1. 二级索引找到主键
  2. 回表查主键索引获取完整数据
  → 两次查找，这就是"回表"
```

#### 6.4 覆盖索引（避免回表）

```sql
-- 只需要查a，索引覆盖，不用回表
CREATE INDEX idx_a ON user(a);
SELECT a FROM user WHERE a=1;  -- 直接返回

-- 需要b，要回表
SELECT b FROM user WHERE a=1;
```

#### 6.5 回答模板

> InnoDB的索引底层是B+Tree，一个节点可以存多个数据，所以树高很低，几层就能cover百万数据，比红黑树高效很多。主键索引的叶子节点存完整行数据，二级索引叶子节点只存主键值。查非索引列需要回表，两次查找。要避免回表就用覆盖索引，只查索引已有的列。

---

### 7. 什么是慢查询？如何分析？

#### 7.1 慢查询定义

```
执行时间 > long_query_time（默认1秒）
→ 记录到慢查询日志
```

#### 7.2 EXPLAIN分析（核心！）

```sql
EXPLAIN SELECT * FROM user WHERE id=1;

关键字段：
  type：连接类型（const>eq_ref>ref>range>index>ALL）
  possible_keys：可选索引
  key：实际用到的索引
  rows：扫描行数（越少越好）
  Extra：Using filesort/Using temporary→要优化
```

**type排名**：
```
最好：const（主键/唯一）
    eq_ref（唯一索引）
    ref（非唯一）
    range（范围）
    index（索引扫描）
最差：ALL（全表扫描）
```

#### 7.3 常见慢查询原因

```sql
-- ❌ 全表扫描
SELECT * FROM user;

-- ✅ 走索引
SELECT * FROM user WHERE id=1;

-- ❌ 函数处理导致索引失效
SELECT * FROM user WHERE YEAR(create_time)=2024;

-- ✅ 正确写法
SELECT * FROM user WHERE create_time>='2024-01-01';

-- ❌ 隐式类型转换
SELECT * FROM user WHERE id='1';  -- id是INT

-- ✅ 类型匹配
SELECT * FROM user WHERE id=1;
```

#### 7.4 回答模板

> 慢查询是执行时间超过1秒的SQL，用EXPLAIN分析。从type列看是不是ALL（全表扫描）；rows列看扫描了多少行；Extra列看有没有Using filesort（文件排序）或Using temporary（临时表）这些要优化的信息。常见问题是：没建索引、索引失效（函数/隐式转换）、查���了���必要的列。

---

## 第三章 事务与并发（高频 ★★★★★）

### 8. 什么是事务？ACID是什么？

#### 8.1 事务定义

> 一组SQL语句，要么全部成功，要么全部失败。

```
转账场景：
  A→B转100元

  ✓ 成功：A-100 + B+100
  ❌ 失败：A-100（失败了）→ 钱呢？

  用事务：要么都成功，要么都回滚
```

#### 8.2 ACID四个特性

| 特性 | 含义 | 如何实现 |
|------|------|---------|
| A 原子性 | 全成功或全失败 | Undo Log |
| C 一致性 | 数据始终正确 | 其他三个保证 |
| I 隔离性 | 并发互不影响 | 锁+MVCC |
| D 持久性 | 提交后不丢失 | Redo Log |

#### 8.3 事务使用

```sql
-- 方式1：手动
BEGIN;
UPDATE account SET balance=balance-100 WHERE id=1;
UPDATE account SET balance=balance+100 WHERE id=2;
COMMIT;  -- 或 ROLLBACK;

-- 方式2：自动提交
SET autocommit=1;  -- 默认
UPDATE account...;
-- 只要出错自动回滚
```

#### 8.4 回答模板

> 事务是一组SQL的集合，要么全部执行成功，要么全部回滚。ACID是事务的四个特性：原子性（Atomic）通过Undo Log保证、一致性（Consistency）靠其他三个保证、隔离性（Isolation）靠锁和MVCC、持久性（Durability）靠Redo Log。MySQL默认 autocommit=1，每条SQL自动提交。

---

### 9. MySQL有哪些事务隔离级别？

#### 9.1 四个隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|---------|------|-----------|------|
| READ UNCOMMITTED | ✓ | ✓ | ✓ |
| READ COMMITTED | ✗ | ✓ | ✓ |
| REPEATABLE READ（默认）| ✗ | ✗ | ✓ |
| SERIALIZABLE | ✗ | ✗ | ✗ |

```
✓会发生  ✗不会发生
```

#### 9.2 问题解释

**脏读**：读到其他事务未提交的数据
```
A事务：UPDATE SET money=1000 WHERE id=1（未提交）
B事务：SELECT money FROM user WHERE id=1 → 1000（脏）
A事务：ROLLBACK → 钱没变！但B读到了！
```

**不可重复读**：同一事务两次读到的数据不同
```
T1：SELECT money=1000
其他事务：UPDATE money=2000 COMMIT
T1：SELECT money=2000（变了！）
```

**幻读**：同一事务两次查询结果行数不同
```
T1：SELECT * FROM user WHERE id>10  → 10行
其他事务：INSERT user(11,'xx') COMMIT
T1：SELECT * FROM user WHERE id>10  → 11行（多了！）
```

#### 9.3 设置隔离级别

```sql
-- 查看
SELECT @@transaction_isolation;

-- 设置（会话级）
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 设置（全局）
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

#### 9.4 回答模板

> MySQL有四个隔离级别：READ UNCOMMITTED（最低，会脏读）、READ COMMITTED（解决脏读，Oracle默认）、REPEATABLE READ（解决不可重复读，MySQL默认，实际上InnoDB已经解决了大部分幻读）、SERIALIZABLE（最高，类似加表锁，性能最差）。一般用默认的REPEATABLE READ就够了。

---

### 10. 什么是MVCC？如何实现？

#### 10.1 MVCC概念

> 多版本并发控制，让每个事务看到的数据版本不一样，避免读写冲突。

```
无MVCC：读写互相等
  事务A：读（等写）
  事务B：写（等读）

有MVCC：各自读自己的版本
  事务A：读版本1
  事务B：写版本2 → 并发执行！
```

#### 10.2 InnoDB MVCC实现

```
三大组件：
1. Undo Log：记录历史版本，用于回滚和读取旧数据
2. Read View：记录活跃事务ID，判断哪些版本可见
3. 隐藏列：每行的 trx_id（事务ID）+ roll_ptr（Undo指针）
```

**Read View可见性判断**：
```
1. trx_id= 当前事务ID → 可见（自己改的）
2. trx_id < 最小活跃ID → 可见（已提交）
3. trx_id > 最大事务ID → 不可见（之后开的）
4. trx_id 在活跃列表 → 不可见（未提交）
```

#### 10.3 快照读 vs 当前读

```sql
-- 快照读：不加锁，读历史版本
SELECT * FROM user WHERE id=1;

-- 当前读：加锁，读最新数据
SELECT * FROM user WHERE id=1 FOR UPDATE;
UPDATE user SET ...;  -- 自动加写锁
```

#### 10.4 回答模板

> MVCC是多版本并发控制，让读写可以并发不必互相等待。InnoDB通过Undo Log记录历史版本、通过Read View判断版本可见性、通过隐藏列(trx_id+roll_ptr)实现。普通的SELECT是快照读不加锁，而SELECT...FOR UPDATE是当前读会加锁。

---

### 11. MySQL有哪些锁？

#### 11.1 锁的分类

```
按粒度：
  - 行锁：锁定一行（InnoDB默认）
  - 表锁：锁定整张表（MyISAM）

按模式：
  - 共享锁（S）：只能读，不能写
  - 排他锁（X）：独占，不能共存

按性质：
  - 意向锁：表级锁，表示"我要加行锁"
```

#### 11.2 行锁与表锁

```sql
-- 行锁（InnoDB）
SELECT * FROM user WHERE id=1 LOCK IN SHARE MODE;  -- 共享锁
SELECT * FROM user WHERE id=1 FOR UPDATE;       -- 排他锁

-- 表锁（MyISAM）
LOCK TABLES user READ;
LOCK TABLES user WRITE;
```

#### 11.3 意向锁

```
为什么需要意向锁？
  - 判断能否加表锁时，不用逐行检查有没有行锁
  - 只需要检查表级意向锁IS/IX
  - 提高效率
```

#### 11.4 死锁

```
死锁：两个事务互相等对方的锁

场景：
  事务A：锁行1，等行2
  事务B：锁行2，等行1
  → 死锁！

解决：
  1. 超时等待（默认50秒）
  2. 死锁检测（默认开启，主动回滚）
  3. 调整SQL顺序，避免循环等待
```

```sql
-- 查看死锁
SHOW ENGINE INNODB STATUS;
```

#### 11.5 回答模板

> MySQL锁分行锁和表锁，InnoDB默认是行锁。锁的模式有共享锁（S）和排他锁（X）。意向锁是表级锁表示"我要加行锁"，快速判断能否加表锁。死锁是两个事务互相等待对方的锁形成循环，InnoDB默认开启死锁检测会主动回滚一个事务。避免死锁的方法是保持事务简短、调整SQL执行顺序。

---

## 第四章 SQL与函数（高频 ★★★★★）

### 12. MySQL统计函数有哪些？

#### 12.1 常用聚合函数

```sql
COUNT(*)      -- 总行数（包括NULL）
COUNT(col)   -- 非空行数
SUM(col)     -- 求和
AVG(col)     -- 平均值
MAX(col)     -- 最大值
MIN(col)     -- 最小值
GROUP_CONCAT -- 拼接成字符串
```

#### 12.2 常用数学函数

```sql
ROUND(3.14159, 2)  -- 3.14，四舍五入
CEIL(3.1)           -- 4，向上取整
FLOOR(3.9)          -- 3，向下取整
ABS(-3)              -- 3，绝对值
POW(2, 3)            -- 8，幂运算
SQRT(16)             -- 4，开方
MOD(10, 3)           -- 1，求余
```

#### 12.3 常用字符串函数

```sql
CONCAT('a', 'b')       -- ab，拼接
CONCAT_WS('-', 'a','b') -- a-b，以分隔符拼接
LENGTH('abc')          -- 3，长度
SUBSTRING('abc', 1, 2) -- ab，截取
TRIM(' abc ')          -- abc，去空格
UPPER('abc')          -- ABC
LOWER('ABC')           -- abc
REPLACE('aabb', 'bb', 'xx') -- aaxx替换
```

#### 12.4 常用日期函数

```sql
NOW()              -- 2024-01-01 12:00:00，当前时间
CURDATE()         -- 2024-01-01，当前日期
CURTIME()         -- 12:00:00，当前时间
YEAR(NOW())       -- 2024，年
MONTH(NOW())      -- 1，月
DAY(NOW())        -- 1，日
DATE_FORMAT(NOW(), '%Y-%m-%d')  -- 格式化
DATEDIFF('2024-01-01', '2024-01-10')  -- -9，天数差
```

#### 12.5 回答模板

> MySQL函数很丰富：聚合函数COUNT/SUM/AVG/MAX/MIN、数学函数ROUND/CEIL/FLOOR/ABS、字符串函数CONCAT/SUBSTRING/TRIM/REPLACE、日期函数NOW/YEAR/MONTH/DATE_FORMAT。实际用的时候注意NULL的处理，用IFNULL/COALESCE来处理可能的NULL值。

---

### 13. MySQL联表查询有哪些？

#### 13.1 JOIN种类

```sql
-- 内连接：两表都有的
SELECT * FROM a INNER JOIN b ON a.id=b.a_id;

-- 左外连接：左表全保留
SELECT * FROM a LEFT JOIN b ON a.id=b.a_id;

-- 右外连接：右表全保留
SELECT * FROM a RIGHT JOIN b ON a.id=b.a_id;

-- 全连接：MySQL不支持，用UNION代替
SELECT * FROM a LEFT JOIN b...
UNION
SELECT * FROM a RIGHT JOIN b...;

-- 笛卡尔积（慎用！）
SELECT * FROM a, b;
```

#### 13.2 连接的执行原理

```
Nested Loop Join（NLJ）：
  1. 驱动表先查一条
  2. 拿着这条去被驱动表Match
  3. 循环直到驱动表查完

优化原则：
  - 小表驱动大表
  - 被驱动表要有索引
  - 用小表作驱动表
```

#### 13.3 ON vs WHERE

```sql
-- ON：连接条件
SELECT * FROM a LEFT JOIN b ON a.id=b.id;
  -- b.id=NULL的行也保留

-- WHERE：过滤条件（在连接后）
SELECT * FROM a LEFT JOIN b ON a.id=b.id WHERE b.id IS NOT NULL;
  -- 相当于INNER JOIN
```

#### 13.4 回答模板

> MySQL有多种JOIN：INNER JOIN（内连接，两表都有的）、LEFT/RIGHT OUTER JOIN（外连接，保留一边）、CROSS JOIN（笛卡尔积）。实际用的时候注意：1）小表驱动大表；2）被驱动表要有索引；3）连接条件放ON里，过滤条件放WHERE里效率更高。

---

### 14. SQL执行顺序是什么？

#### 14.1 SQL执行顺序

```
写SQL：
SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT

实际执行：
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

关键字顺序不能变，但执行顺序是定的！
```

#### 14.2 各子句执行时机

```sql
SELECT id, COUNT(*) as cnt  -- 5.SELECT，选择列
FROM user                  -- 1.FROM，确定表
WHERE id>10               -- 2.WHERE，筛选
GROUP BY id                -- 3.GROUP BY，分组
HAVING COUNT(*)>10         -- 4.HAVING，分组后筛选
ORDER BY cnt DESC          -- 6.ORDER BY，排序
LIMIT 10;                 -- 7.LIMIT，限制数量
```

#### 14.3 注意WHERE vs HAVING

```sql
-- WHERE：筛选行，在分组前
SELECT * FROM user WHERE age>20;

-- HAVING：筛选分组结果，在分组后
SELECT age, COUNT(*) FROM user GROUP BY age HAVING COUNT(*)>10;
```

#### 14.4 回答模板

> SQL书写顺序是SELECT...FROM...WHERE...GROUP BY...HAVING...ORDER BY...LIMIT，但实际执行顺序是FROM先确定表，然后WHERE筛选，接着GROUP BY分组、HAVING对分组结果筛选，然后SELECT选择列，最后ORDER BY排序、LIMIT限制。WHERE和HAVING的区别是WHERE在分组前筛选原始数据，HAVING在分组后筛选聚合结果。

---

## 第五章 表设计与约束（中高频 ★★★★）

### 15. 如何设计MySQL表结构？

#### 15.1 设计原则

```
1. 字段精简：小而细，不要大而全
2. 类型合适：够用就行
3. 适当冗余：减少JOIN
4. 主键自增：效率高
5. 命名规范：下划线分割
```

#### 15.2 数据类型选择

| 类型 | 占用 | 使用场景 |
|------|------|---------|
| TINYINT | 1字节 | 状态（0-255） |
| INT | 4字节 | 普通ID |
| BIGINT | 8字节 | 大数值、ID |
| VARCHAR(n) | n+1字节 | 可变字符串 |
| CHAR(n) | n字节 | 定长（性别等） |
| DATETIME | 8字节 | 时间 |
| DECIMAL(10,2) | - | 金额（精准） |

#### 15.3 表结构示例

```sql
CREATE TABLE `user` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `mobile` CHAR(11) NOT NULL COMMENT '手机号',
  `email` VARCHAR(100) DEFAULT NULL COMMENT '邮箱',
  `status` TINYINT NOT NULL DEFAULT 1 COMMENT '状态:1正常',
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`),
  KEY `idx_mobile` (`mobile`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

#### 15.4 回答模板

> 表设计原则：主键用BIGINT自增、字段类型合适、能用TINYINT就不用INT节省空间。时间字段用DATETIME。每列加COMMENT便于维护，必须加索引：主键、唯一键、WHERE条件的列。我通常会预留一些冗余字段避免后续JOIN太多表，也方便扩展。

---

### 16. MySQL约束有哪些？

#### 16.1 六大约束

```sql
-- 主键约束：唯一非空
PRIMARY KEY (id)

-- 唯一约束：唯一
UNIQUE KEY `uk_username` (username)

-- 非空约束：NOT NULL
`name` VARCHAR(50) NOT NULL

-- 默认值约束：DEFAULT
`status` TINYINT DEFAULT 1

-- 外键约束：REFERENCES
FOREIGN KEY ( dept_id ) REFERENCES dept(id)

-- 检查约束（MySQL8.0前不支持）
CHECK ( age >= 18 )
```

#### 16.2 自增主键

```sql
-- 自增
CREATE TABLE t (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY
);

-- 设置起始值
ALTER TABLE t AUTO_INCREMENT=1000;

-- 查看
SHOW CREATE TABLE t;
```

#### 16.3 外键优缺点

```
优点：
  - 保证数据完整性
  - 级联操作自动处理

缺点：
  - 增加耦合度
  - 影响插入性能
  - 拆分困难

现代开发建议：
  - 应用层保证完整性
  - 不用外键，用逻辑关联
```

#### 16.4 回答模板

> MySQL有六大约束：主键（唯一非空）、唯一（非空）、非空、默认值、外键、检查约束。外键在现代开发中不太推荐用，因为会增加表之间的耦合度，影响性能，后续拆库也麻烦。一般在应用层保证数据完整性。

---

## 第六章 主从复制与集群（高频 ★★★★★）

### 17. MySQL主从复制原理？

#### 17.1 主从架构

```
主库（Master）
  ↓ binlog
从库（Slave）
  → relay log → replay
```

#### 17.2 复制原理

```
1. 主库：写数据 → 记录 binlog
2. 主库：Dump线程发送binlog给从库
3. 从库：IO线程接收 → 写relay log
4. 从库：SQL线程重放 → 执行SQL
```

**三个线程**：
```
主库：Dump（发送binlog）
从库：IO线程（接收）、SQL线程（执行）
```

#### 17.3 三种复制方式

| 方式 | 优点 | 缺点 |
|------|------|------|
| 异步复制 | 主库快 | 可能丢数据 |
| 半同步 | 至少1个确认 | 略慢 |
| 全同步 | 不丢数据 | 很慢 |

#### 17.4 GTID复制

```
GTID：Global Transaction ID
  格式：server_uuid:transaction_id
  优点：自动同步，不用记位置
```

```sql
-- 启用GTID
SET GLOBAL gtid_mode=ON;
SET GLOBAL enforce_gtid_consistency=ON;
```

#### 17.5 回答模板

> MySQL主从复制的原理是：主库把写操作记录到binlog，从库用IO线程拉取binlog并写入relay log，然后用SQL线程重放执行。GTID是全局事务ID，可以自动同步，不用手动指定位置，更可靠。主从复制的三种方式：异步（最快，可能丢数据）、半同步（均衡）、全同步（最慢但安全）。

---

### 18. MySQL读写分离如何实现？

#### 18.1 读写分离架构

```
         ┌──────┐
  写 ────┤主库  │
         └──────┘   ↓ binlog
         ┌──────┐
  读 ────┤从库1 │
         └──────┘
         ┌──────┐
  读 ────┤从库2 │
         └──────┘
```

#### 18.2 实现方式

```
方式1：应用层控制
  写操作→主库
  读操作→从库

方式2：中间件（推荐）
  - MyCat（Java）
  - ShardingSphere（开源）
  - ProxySQL

方式3：MGR
  MySQL Group Replication
```

#### 18.3 主从延迟问题

```
原因：
  1. 从库SQL线程是单线程
  2. 大事务
  3. 网络问题

解決：
  1. 修改为多线程（ Apostles）
  2. 拆分事务
  3. 肉鸡配置网络
```

#### 18.4 回答模板

> 读写分离的架构是：写操作走主库，读操作走从库。实现方式可以用应用层控制（自己判断）、中间件代理（MyCat/ShardingSphere）、或MySQL内置的MGR。主从延迟的常见原因是SQL线程单线程、大事务、网络问题，可以通过多线程复制、拆分事务来优化。

---

### 19. MySQL如何分库分表？

#### 19.1 什么时候分？

```
单表 > 1000万 条 → 考虑分
单库 > 2-3TB → 考虑分
```

#### 19.2 垂直拆分

```
垂直分库：按业务拆分
  用户库、订单库、商品库

垂直分表：按列拆分
  user_info | user_detail
```

#### 19.3 水平拆分

```
水平分库/表：按数据拆分
  按ID取模：
    user_0: id % 2 = 0
    user_1: id % 2 = 1

  按范围：
    user_0: 1-1000万
    user_1: 1000万+
```

#### 19.4 分片键选择

```
原则：1. 数据均匀分散
       2. 业务经常查询
       3. 不经常变化

推荐：user_id、create_time
避免：status（值少）、sex（就2个）
```

#### 19.5 分片中间件

```
- ShardingSphere（Apache）
- MyCat（Java）
- Vitess（YouTube）
- StarRocks
```

#### 19.6 回答模板

> 当单表超过1000万或单库3TB时就需要考虑了。垂直分库是按业务拆分表到不同库，垂直分表是把一个表的大字段拆出去。水平分库/表是把数据按某个规则（如取模、时间）分散到多个库或表。选择分片键很关键：要数据分散均匀、要业务经常查询。生产环境常用ShardingSphere等中间件。

---

## 第七章 高可用与运维（中高频 ★★★★）

### 20. MySQL高可用方案？

#### 20.1 常见高可用架构

| 方案 | 说明 | 适用场景 |
|------|------|---------|
| 主从+MHA | MHA监控自动切换 | 中小规模 |
| 主从+Sentinel | 应用层检测切换 | 中小规模 |
| MGR | 组复制，自选主 | 中大规模 |
| PXC | Percona Galera | 中大规模 |
| 双主 | 互为主从 | 小规模 |

#### 20.2 MGR（Group Replication）

```
MGR特点：
  ✓ 自动选主
  ✓ 多数派决策
  ✓ 强一致
  ✓ 单主或多主模式

架构：
  ┌────┐   ┌────┐   ┌────┐
  │ S1 │ ←→ │ S2 │ ←→ │ S3 │
  └────┘   └────┘   └────┘
   ↓         ↓         ↓
 多数派 → 谁是主
```

```sql
-- 启动MGR
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
SET globally group_replication_group_name='uuid';
SET globally group_replication_start_on_boot='ON';
```

#### 20.3 MHA vs MGR

| 特性 | MHA | MGR |
|------|-----|-----|
| 一致性 | 异步 | 强一致 |
| 自动选主 | ✅ | ✅ |
| 最小节点 | 2 | 3 |
| 故障恢复 | 手动 | 自动 |

#### 20.4 回答模板

> MySQL高可用常见方案：MHA（需要额外组件监控）、MGR（5.7+内置，组复制，强一致）、PXC（Percona Galera）、双主（互相同步）。MGR是目前比较流行的方案，自动选主，多数派决策，保证强一致。生产环境至少要3节点。

---

### 21. MySQL备份恢复？

#### 21.1 备份类型

```
逻辑备份：导出SQL
  - mysqldump（小数据量）
  - mydumper（大数据并行）

物理备份：拷贝文件
  - xtrabackup（热备，支持增量）
  - 直接cp（冷备，需停库）
```

#### 21.2 常用备份命令

```bash
# 逻辑备份（mysqldump）
mysqldump -uroot -p dbname > backup.sql

# 恢复
mysql -uroot -p dbname < backup.sql

# 物理备份（xtrabackup）
xtrabackup --backup --target-dir=/backup/
xtrabackup --prepare --target-dir=/backup/
xtrabackup --copy-back --target-dir=/backup/
```

#### 21.3 备份策略

```
每日全量 + Binlog
  每天1次全量 + 实时binlog
  恢复：全量 + binlog point-in-time

每周全量 + 每日增量
  周日全量 + 周一到周六增量
  恢复：最近全量 + 之后增量
```

#### 21.4 回答模板

> 备份分逻辑备份（mysqldump，适合小数据量）和物理备份（xtrabackup，生产环境推荐，支持热备和增量）。恢复的时候只需要source sql文件或xtrabackup --copy-back。备份策略一般是每日全量+Binlog，或每周全量+每日增量。

---

### 22. MySQL日常运维命令？

```sql
-- 查看状态
SHOW STATUS;
SHOW PROCESSLIST;  -- 当前连接
SHOW VARIABLES;     -- 参数配置

-- 分析表
ANALYZE TABLE t;  -- 更新统计信息
OPTIMIZE TABLE t; -- 回收碎片

-- 查看慢查询
SHOW VARIABLES LIKE 'slow_query%';
SHOW GLOBAL STATUS LIKE 'Slow_queries';

-- 查看连接
SELECT * FROM information_schema.PROCESSLIST;

-- killsession
KILL [CONNECTION | QUERY] process_id;
```

```ini
# my.cnf常用配置
[mysqld]
max_connections = 500
innodb_buffer_pool_size = 4G
long_query_time = 2
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
```

#### 22.1 回答模板

> 日常运维常用命令：SHOW PROCESSLIST看当前连接、SHOW VARIABLES看参数配置、ANALYZE TABLE更新统计信息、OPTIMIZE TABLE回收碎片。慢查询相关的参数是slow_query_log和long_query_time。连接数过多时可以KILL掉空闲连接。

---

## 第八章 数据库三范式（初中频 ★★★）

### 23. 数据库三范式？

#### 23.1 第一范式（1NF）

> 每个字段不可再分。

```
❌ 违反1NF：
  address: "北京市朝阳区xx街道1号"

✓ 符合1NF：
  province: "北京"
  city: "北京"
  district: "朝阳"
  detail: "xx街道1号"
```

#### 23.2 第二范式（2NF）

> 满足1NF + 非主键字段完全依赖主键。

```
❌ 违反2NF（部分依赖）：
  主键：(order_id, product_id)
  order_date: 只依赖order_id → 部分依赖

✓ 符合2NF：
  拆分成：
  orders(order_id, order_date, ...)
  order_items(order_id, product_id, ...)
```

#### 23.3 第三范式（3NF）

> 满足2NF + 非主键字段消除传递依赖。

```
❌ 违反3NF（传递依赖）：
  user_id → dept_id → dept_name → dept_manager

  这里dept_name其实是传递依赖于dept_id

✓ 符合3NF：
  users(user_id, ..., dept_id)
  departments(dept_id, dept_name, dept_manager)
```

#### 23.4 回答模板

> 数据库设计的三范式是：1NF是字段不可��分��2NF是非主键字段完全依赖主键而不是部分依赖；3NF是非主键字段消除传递依赖（即A→B→C这种情况）。反正规化的情况也有，比如为了查询快可以适当冗余字段。

---

## 第九章 实战问题（高频 ★★★★★）

### 23. 如何处理高并发数据写入？

#### 23.1 方案1：批量插入

```sql
-- ❌ 低效：循环插入
INSERT INTO t VALUES(1);
INSERT INTO t VALUES(2);
...

-- ✅ 高效：批量插入
INSERT INTO t VALUES(1),(2),(3),(4),...;
-- 一次最多1000条
```

#### 23.2 方案2：Load Data

```bash
# 比INSERT快10倍+
LOAD DATA LOCAL INFILE '/tmp/data.csv'
INTO TABLE t
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n';
```

#### 23.3 方案3：异步写入

```
写入流程：
  1. 先写Redis/Queue
  2. 后台异步批量写入MySQL

这样可以抗住瞬时高峰！
```

#### 23.4 回答模板

> 高并发写入优化几个方法：1）批量INSERT，一次500-1000条；2）用LOAD DATA INFILE导入，比普通INSERT快10倍+；3）写Redis/消息队列做缓冲，后台异步批量落库，这样可以扛住瞬时峰值。

---

### 24. 分页查询如何优化？

#### 24.1 OFFSET大问题

```sql
-- ❌ 慢：OFFSET过大
SELECT * FROM t ORDER BY id LIMIT 100000, 10;
-- 先扫描100010行，只返回后10行

-- ✅ 快：
  1）基于ID
  SELECT * FROM t WHERE id > 100000 LIMIT 10;

  2）先查ID再JOIN
  SELECT t.* FROM t
  INNER JOIN (SELECT id FROM t LIMIT 100000, 10) b
  ON t.id = b.id;
```

#### 24.2 子查询分页

```sql
-- 深度分页优化
SELECT * FROM t
WHERE id >= (
    SELECT id FROM t ORDER BY id LIMIT 100000, 1
) LIMIT 10;
```

#### 24.3 游标分页

```python
# 服务端游标分页
last_id = 0
while True:
    users = db.query("SELECT * FROM user WHERE id > ? ORDER BY id LIMIT 20", last_id)
    if not users:
        break
    last_id = users[-1]['id']
    process(users)
```

#### 24.4 回答模板

> 分页查询OFFSET过大的问题是会先扫描很多行再返回。优化方法：1）基于ID的游标分页，只查下一页；2）先查ID用子查询再JOIN；3）不深度分页的用户体验用"加载更多"替代页码。

---

### 25. 数据库连接池如何配置？

#### 25.1 连接池参数

```java
// HikariCP参数
HikariConfig config = new HikariConfig();
config.setMaximumPoolSize(20);      // 最大连接数
config.setMinimumIdle(5);          // 最小空闲
config.setConnectionTimeout(30000);// 获取超时30秒
config.setIdleTimeout(600000);    // 空闲10分钟回收
config.setMaxLifetime(1800000);   // 最大生命周期30分钟
config.setConnectionTestQuery("SELECT 1"); // 测试sql
```

#### 25.2 连接数计算

```java
// 经验公式
// CPU密集型：CPU核心数 × 2 + 磁盘数
// IO密集型：CPU核心数 × 2 + 磁盘数 × 2

// 例如：4核CPU + 1块SSD = 10个连接
// 生产环境：一般50-100足够
```

#### 25.3 回答模板

> 连接池是必须的，可以复用连接避免频繁建立断开。主要参数：maximumPoolSize（最大连接数，建议CPU×2+磁盘×2）、minimumIdle（最小空闲保持预热）、connectionTimeout（获取连接超时）、maxLifetime（连接最大生命周期，到期回收）。我们公司用HikariCP，最大连接数设为20，日常使用率在50%-80%健康。

---

### 26. 如何防止SQL注入？

#### 26.1 什么是SQL注入？

```sql
-- 输入：admin' --
SELECT * FROM user WHERE username='admin' --' AND password='xxx';
-- 恒成立，因为--后面都被注释了
```

#### 26.2 防御方法

```java
// ✅ PreparedStatement
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);
ps.setString(1, username);
ps.setString(2, password);

// ❌ 字符��拼��（有漏洞）
"SELECT * FROM user WHERE username='" + username + "'"
```

#### 26.3 MyBatis使用

```xml
<!-- ✅ SQL安全 -->
<select id="findUser">
    SELECT * FROM user WHERE username = #{username}
</select>

<!-- ❌ 危险写法 -->
<select id="findUser">
    SELECT * FROM user WHERE username = ${username}
</select>
```

#### 26.4 回答模板

> SQL注入是通过在输入中植入SQL片段来攻击。解决办法是：1）用PreparedStatement / MyBatis的#{}，不要字符串拼接；2）输入过滤验证；3）最小权限原则，应用只给必要权限。实际开发中养成习惯用#{}就对了。

---

### 27. MySQL和Redis有什么区别？

#### 27.1 区别表

| 特性 | MySQL | Redis |
|------|------|-------|
| 类型 | 关系型 | NoSQL |
| 存储 | 磁盘 | 内存 |
| 事务 | ACID | 有限支持 |
| 索引 | B+Tree | Hash/List |
| 场景 | 持久化数据 | 缓存/计数器 |
| 数据量 | 亿级 | 千万级 |
| 复杂查询 | ✅ | 有限 |

#### 27.2 使用场景互补

```
MySQL：核心业务数据（用户、订单、交易）
Redis：缓存热点、计数器、Session、分布式锁
```

#### 27.3 回答模板

> MySQL是关系型持久化数据库，功能全、支持事务、适合复杂查询；Redis是内存缓存在不同��，用法不同。常见组合：核心数据用MySQL、热点数据用Redis缓存、计数用Redis的INCR、分布式锁用Redis SETNX。它们是互补关系，不是替代关系。

---

### 28. 查询超时怎么处理？

```sql
-- 查看执行时间
SHOW PROFILES;
SHOW PROFILE;

-- 超时设置
SET SESSION MAX_EXECUTION_TIME=1000;  -- 1秒超时
SET GLOBAL MAX_EXECUTION_TIME=1000;
```

#### 28.1 排查慢查询

```sql
-- 慢查询日志
SHOW VARIABLES LIKE 'slow_query_%';

-- 查看哪些查询慢
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;

-- EXPLAIN分析
EXPLAIN FORMAT=JSON SELECT * FROM t WHERE ...
```

#### 28.2 回答模板

> 查询超时可以用SHOW PROFILES和慢查询日志排查。先EXPLAIN看执行计划，确认是否走索引、扫描行数多少。然后根据原因：加索引、优化SQL、减少返回列、分表处理或改用缓存。

---

### 29. 大数据量删除怎么优化？

```sql
-- ❌ 大数据量删除会很慢并产生大量binlog
DELETE FROM t WHERE create_time < '2023-01-01';

-- ✅ 分批删除
DELETE FROM t WHERE create_time < '2023-01-01' LIMIT 1000;
-- 循环执行，直到删完

-- ✅ 先删索引再删数据
ALTER TABLE t DROP INDEX idx_xxx;
DELETE FROM t;
ALTER TABLE t ADD INDEX idx_xxx;
```

#### 29.1 惰性删除和硬链接

```bash
# 硬链接方式（几乎瞬间）
ln /data/mysql/t.ibd /data/mysql/t.ibd.bak
rm /data/mysql/t.ibd
# 数据库不会重建，但磁盘空间回收了
```

#### 29.2 回答模板

> 大数据量删除不要一次性DELETE，会产生大量binlog和锁等待。正确做法是分批删除，或先删索引删数据再加索引。也可以用硬链接方式删大表，接近瞬间完成。日常维护中可以定期归档历史数据，不要把冷数据堆积在主表。

---

### 30. 如何设计分库分表后的ID？

#### 30.1 ID生成方案

```java
// 方案1：数据库自增
// 每库设置不同起始值和步长
DB1: 1,3,5,... (step=2)
DB2: 2,4,6,...

// 方案2：UUID
UUID.randomUUID() // 字符串

// 方案3：Snowflake算法（推荐）
// 时间戳 + 机器ID + 序列号
// 41位时间戳 + 10位机器ID + 12位序列号
```

#### 30.2 Snowflake示例

```java
// 雪花算法
// 1位不用 + 41位时间戳 + 5位数据中心 + 5位机器ID + 12位序列号
// 每秒百万级，趋势递增
// Twitter开源

// 已有封装：leaf（美团）、tinyid（滴滴）
```

#### 30.3 回答模板

> 分库分表后的ID生成有几种：1）数据库自增，设置不同的起始值和步长；2）UUID，简单但无序；3）Snowflake算法，时间戳+机器ID+序列号，性能和趋势性都好，推荐使用。生产环境可以选美团的leaf或滴滴的tinyid中间件。

---

## 第十章 高级问题（中高频 ★★★★）

### 31. Char和Varchar的区别？

| 类型 | 存储 | 长度 | 使用场景 |
|------|------|------|---------|
| CHAR(n) | 固定n字节 | n | 性别、状态等定长 |
| VARCHAR(n) | n+1字节 | 可变 | 名字、地址等变长 |

```sql
-- CHAR(100) 即使只存1个字符也占100字节
-- VARCHAR(100) 存1个字符只占2字节（1+1）
```

#### 31.1 InnoDB行格式

```
COMPACT行格式：
  - 变长字段长度列表（2字节）
  - NULL标志位
  - row header
  - column data

VARCHAR最大：65535字节，但实际只有16383字节（2字节存长度）
CHAR最大：255字节
```

---

### 32. Int各种类型的区别？

| 类型 | 范围 | 字节 |
|------|------|------|
| TINYINT | -128~127 | 1 |
| SMALLINT | -32768~32767 | 2 |
| INT | -21亿~21亿 | 4 |
| BIGINT | 很大 | 8 |

```sql
-- TINYINT：状态、开关
-- SMALLINT：普通ID
-- INT：大ID、统计
-- BIGINT：金融、精确计算
```

---

### 33. Where条件多个and和or的优先级？

```sql
-- 实际执行顺序：先AND后OR
SELECT * FROM t WHERE a=1 or a=2 AND b=3;
-- 等于
SELECT * FROM t WHERE a=1 OR (a=2 AND b=3);

-- 应该加括号明确
SELECT * FROM t WHERE (a=1 OR a=2) AND b=3;
```

---

### 34. COUNT(列名)和COUNT(*)的区别？

| 函数 | 说明 |
|------|------|
| COUNT(*) | 统计总行数，包括NULL |
| COUNT(col) | 统计非NULL行数 |
| COUNT(DISTINCT col) | 统计非NULL去重 |

```sql
-- NULL不参与统计
SELECT COUNT(col) FROM t;  -- 只统计非NULL
SELECT COUNT(*) FROM t;  -- 统计所有行，包括NULL
```

---

### 35. UNICODE和UTF-8的区别？

```
MySQL字符集：
- utf8：最多3字节，emoji不支持
- utf8mb4：最多4字节，支持emoji
- utf8mb4是MySQL的叫法，等于标准UTF-8

推荐：utf8mb4
CREATE TABLE t (...) CHARSET=utf8mb4;
```

---

## 附录：面试追问

1. **MySQL为什么用B+Tree而不是B-Tree？**
   - B+Tree非叶子节点不存数据，只存指针，同样的磁盘页可以存更多指针，树更矮

2. **MySQL如何实现乐观锁？**
   - 用版本号：WHERE id=? AND version=? 然后UPDATE SET version=version+1

3. **Redis和MySQL如何保证一致性？**
   - Cache Aside：先更新数据库后删除缓存，或延迟双删

4. **数据库死锁怎么排查？**
   - SHOW ENGINE INNODB STATUS查看最近死锁，画的图看等待关系

5. **分库分表后如何做JOIN？**
   - 只能应用层JOIN，或用中间件支持跨库JOIN

6. **MySQL 8.0新特性？**
   - 窗口函数 CTE Recursive、Hash Grouping Instant、GIS增强

7. **如何监控MySQL？**
   - Prometheus + MySQLExporter + Grafana

8. **explain有哪些重要字段？**
   - type、key、rows、Extra

9. **慢查询怎么优化？**
   - 加索引、改写SQL、减少返回列、分表

10. **InnoDB和MyISAM适合什么场景？**
    - InnoDB通用，MyISAM纯读静态表

---

## 参考资料

- MySQL官方文档：https://dev.mysql.com/doc/
- 《高性能MySQL》
- 《MySQL技术内幕》

---

> 整理by Claude Code | MySQL面试高频100问