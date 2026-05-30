# Go夺命连环100问——Go语言核心技术深度指南

> 本文档面向Go语言学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. Go语言特点？

#### 1.1 核心特性

> Go是Google 2009年推出的编译型语言，主要特点是：静态类型、自动垃圾回收、天然并发、简洁易读。

```go
// Go核心特点：
// 1. 静态类型 + 编译型
// 2. 自动GC（垃圾回收）
// 3. 协程goroutine（轻量）
// 4. 通道channel（并发通信）
// 5. 反射reflect
// 6. 静态链接
// 7. 多返回值
```

#### 1.2 vs 其他语言

```
          C/C++     Python    Java     Go
类型      静态     动态     静态    静态
编译     是       否       是       是
GC       手动     是       是       是
并发     线程     GIL      线程    goroutine
泛型      C++模板   无       泛型     1.18+
并发     难       难       中      易
```

#### 1.3 回答模板

> Go是Google 2009年的编译型语言，核心特点：自动GC、goroutine轻量协程、channel并发通信、天生适合高并发。简洁静态类型+csp并发模型，Golang在服务器、微服务、云原生领域流行。Gopher是Go开发者昵称。

---

### 2. Go程序编译运行？

#### 2.1 编译运行

```go
// Hello World
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}

// 运行
go run hello.go    // 直接运行
go build hello.go  // 编译
./hello          // 执行编译产物
```

#### 2.2 Go命令

```
go build    - 编译
go run      - 编译+运行
go install - 编译+安装到GOPATH/bin
go get     - 下载依赖
go test    - 测试
go fmt     - 格式化代码
go vet     - 代码静态检查
go doc     - 查看文档
go mod init  - 初始化模块
```

#### 2.3 回答模板

> Go是编译型语言，go run直接运行，go build编译可执行文件，go install安装。main函数是入口，package名main是可执行标志。与Python解释型、Java需JVM不同，Go编译后单文件静态链接。

---

### 3. 变量声明？

#### 3.1 声明方式

```go
// 1. var 关键字
var name string = "Tom"

// 2. 类型推断
var name = "Tom"  // type: string

// 3. 简短声明（函数内）
name := "Tom"

// 4. 多变量
var (
    name string = "Tom"
    age int = 20
)

// 5. new
p := new(int)  // *int指向新实例
*p = 10

// 6. make
ch := make(chan int)  // channel
mp := make(map[string]int)
sl := make([]int, 0)
```

#### 3.2 常量

```go
// const
const Pi = 3.14
const (
    StatusOK = 200
    StatusNG = 500
)
```

#### 3.3 回答模板

> Go变量用var声明或:=简短声明（函数内）。new创建指针实例，make创建slice/map/channel。const定义常量，无类型常量可以理解为编译器辅助。

---

### 4. 基本数据类型？

#### 4.1 数值类型

```go
// 有符号整数
int8   / int16 / int32 / int64
// 无符号整数
uint8  / uint16 / uint32 / uint64
// 浮点数
float32 / float64
// 复数
complex64 / complex128

// byte是uint8别名
// rune是int32别名（Unicode码点）
```

#### 4.2 其他类型

```go
// 布尔
bool

// 字符串
string

// 派生类型
// []int      slice动态数组
// [5]int     数组（固定长度）
// map[string]int   map哈希表
// chan int   通道
// pointer   指针
```

#### 4.3 零值

```
数值类型：0
bool：false
string：""
pointer/channel/slice/map/func/error：nil
```

#### 4.4 回答模板

> Go基本数据类型：整数int/uint，浮点数float32/float64，布尔bool，字符串string。byte是uint8别名，rune是int32 Unicode。零值（初始值）：int=0，bool=false，string=""。

---

### 5. 字符串string？

#### 5.1 字符串操作

```go
// 拼接
s := "hello" + "world"

// 长度
len(s)

// 索引
s[0]  // byte类型 'h'

// 切片
s[0:5]    // hello
s[:5]     // hello
s[5:]     // world
s[:]      // 全部
```

#### 5.2 包函数

```go
import "strings"

// 包含
strings.Contains(s, "ll")

// 分割
strings.Split(s, ",")

// 替换
strings.Replace(s, "l", "L", 1)  // 替换1次
strings.ReplaceAll(s, "l", "L")   // 替换所有

// 大小写
strings.ToLower(s)
strings.ToUpper(s)
strings.Title(s)

// 修剪
strings.TrimSpace(s)
strings.Trim(s, "h")
```

#### 5.3 类型转换

```go
// strconv 包
import "strconv"

i, _ := strconv.Atoi("123")        // 字符串→int
s := strconv.Itoa(123)            // int→字符串
f, _ := strconv.ParseFloat("3.14", 64)  // 字符串→float

// fmt.Sprintf
s := fmt.Sprintf("%d-%s", 123, "abc")
```

#### 5.4 回答模板

> Go字符串用UTF-8编码，是只读byte序列。len()是byte长度，Unicode字符用rune。strings包提供丰富操作，strconv做类型转换。fmt.Sprintf是格式化字符串。

---

### 6. Slice切片？

#### 6.1 创建

```go
// 1. make
s := make([]int, 0)
s := make([]int, 5, 10)  // len=5, cap=10

// 2. 字面量
s := []int{1, 2, 3}
s := []int{}

// 3. 从数组/切片
arr := [5]int{1,2,3,4,5}
s := arr[:]
```

#### 6.2 操作

```go
append(s, 4)      // 追加（自动扩容）
copy(d, s)       // 复制

s = append(s[:2], s[3:]...)  // 删除（变通方式）
// 或用append
```

#### 6.3 原理

```
Slice = 指针 + len + cap
├── Pointer：指向底层array
├── Length：当前长度
└── Capacity：底层array容量

Slice扩充时：
- 若cap够，直接扩大
- 若cap不够，分配2倍容量新数组copy过去
```

#### 6.4 回答模板

> Slice是Go重要的数据结构，三部分：ptr指针+len长度+cap容量。基于array，可动态增长。append自动扩容copy，删除中间元素用append变通。面试常问：slice非指针，append返回值需接收。

---

### 7. Map哈希表？

#### 7.1 创建

```go
// make
m := make(map[string]int)

// 字面量
m := map[string]int{
    "Tom": 20,
    "Jerry": 18,
}

// 默认值判断
if _, ok := m["Bob"]; !ok {
    fmt.Println("not exists")
}
```

#### 7.2 操作

```go
// 添加/修改
m["Tom"] = 20

// 删除
delete(m, "Tom")

// 遍历
for k, v := range m {
    fmt.Println(k, v)
}

// 并发不安全！
// 并发用sync.Map或加锁
```

#### 7.3 回答模板

> Go Map哈希表无序key-value键值对，用make创建或字面量。[]中括号是map[int]int写法，别混淆array和slice。遍历顺序随机，delete删除。重要：Map非线程安全，并发用sync.Map或锁。

---

### 8. 指针Pointer？

#### 8.1 &和*

```go
// &取地址
p := &v

// *解引用
*v = 10

// new创建指针
p := new(int)
*p = 10
```

#### 8.2 值传递vs指针传递

```go
// 值传递
func modify(v int) {
    v = 10
}

// 指针传递
func modify(v *int) {
    *v = 10
}
```

#### 8.3 限制

```go
// Go指针限制：
// 1. 不能进行指针运算
p++ // 错误

// 2. 无法查看内存
// 没有&取地址

// 3. 栈内存安全
// Go会分析逃逸，把对象分到堆
```

#### 8.4 回答模板

> Go指针用*Type，&���地��。指针传递避免值拷贝改变原值，省内存。Java无指针但Go保留指针有限制：不能运算但安全。Go自动分析逃逸决定栈/堆分配。

---

## 第二章 函数与方法篇（高频 ★★★★★）

### 9. 函数定义？

#### 9.1 基本函数

```go
func add(a, b int) int {
    return a + b
}

// 多返回值
func div(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return a / b, nil
}

// 命名返回值
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return  // 返回命名值
}
```

#### 9.2 变参

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
sum(1, 2, 3)
```

#### 9.3 函数类型

```go
// 函数作为变量/参数
type CalcFunc func(int, int) int

func calc(a, b int, f CalcFunc) int {
    return f(a, b)
}
calc(1, 2, add)
```

#### 9.4 回答模板

> Go函数用func定义，多返回值是特色。返回值可命名return简化。变参用...Type语法。函数是一等公民，可作为变量/参数/返回值。

---

### 10. defer延迟执行？

#### 10.1 defer规则

```go
func test() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
}
// 输出顺序：3, 2, 1（LIFO后进先出）
```

#### 10.2 defer与return

```go
func test() int {
    defer func() {
        // defer在return之前执行
    }()
    return 10  // return 10; defer; return返回
}
```

#### 10.3 defer参数

```go
// defer参数在定义时确定
i := 1
defer func(n int) {
    // 此时n=1，非2
}(i)
i = 2
```

#### 10.4 回答模板

> defer是函数返回前最后执行的语句，多个defer按LIFO后进先出顺序执行。用于资源释放、异常捕获。defer参数在定义时确定，非返回值时。

---

### 11. 错误处理error？

#### 11.1 error接口

```go
type error interface {
    Error() string
}

// 自定义error
type MyError struct {
    msg string
}
func (e *MyError) Error() string {
    return e.msg
}

// 或用errors.New
err := errors.New("错误信息")

// 或用fmt.Errorf
err := fmt.Errorf("%s", "格式")
```

#### 11.2 错误检查

```go
result, err := divide(10, 0)
if err != nil {
    fmt.Println(err)
    return  // 或处理错误
}
fmt.Println(result)
```

#### 11.3 多返回值简化

```go
// 常用模式
value, err := dosomething()
if err != nil {
    return err
}
```

#### 11.4 回答模板

> Go无异常Exception，用返回error表示错误。error是interface，实现Error()方法。多个返回值时返回err检查。不用try-catch，用检查err!=nil方式处理。

---

### 12. 方法定义？

#### 12.1 方法vs函数

```go
// 函数
func add(a, b int) int {
    return a + b
}

// 方法（属��于类型）
type User struct {
    Name string
    Age int
}

func (u User) GetName() string {
    return u.Name
}

// 指针receiver（修改原对象）
func (u *User) SetAge(age int) {
    u.Age = age
}
```

#### 12.2 receiver类型

```go
// 值receiver：复制一份
func (u User) String() string {
    return u.Name
}

// 指针receiver：修改原对象
func (u *User) SetName(name string) {
    u.Name = name
}

// 方法变量值receiver和指针receiver可互换
```

#### 12.3 回答模板

> Go方法用func (receiver) Method()定义。值receiver复制一份，指针receiver修改原对象。习惯上修改属性用指针否则值。

---

### 13. Go接口？

#### 13.1 接口定义

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// 接口组合
type ReadWriter interface {
    Reader
    Writer
}
```

#### 13.2 接口实现

```go
// 隐式实现：实现全部方法即实现接口
type File struct{}

func (f *File) Read(p []byte) (n int, err error) {
    return 0, nil
}
func (f *File) Write(p []byte) (n int, err error) {
    return 0, nil
}

// File实现了Reader+Writer接口
var _ io.ReadWriter = (*File)(nil)  // 编译检查
```

#### 13.3 空接口

```go
// 空接口：可接受任意类型
var v interface{}
v = 123
v = "abc"
v = []int{}

// interface{}, any (Go 18+)
var v any
```

#### 13.4 回答模板

> Go接口是方法集合，隐式实现满足即实现。非侵入性是Go接口特色，空interface和any可接受任意类型。

---

### 14. 类型断言？

#### 14.1 类型断言

```go
var v interface{} = "hello"

// 语法
s := v.(string)        // 成功返回，失败panic
s, ok := v.(string)   // ok判断安全

// switch类型断言
switch v.(type) {
case string:
    fmt.Println("string")
case int:
    fmt.Println("int")
default:
    fmt.Println("unknown")
}
```

#### 14.2 类型选择

```go
func whatType(v interface{}) {
    switch v := v.(type) {
    case nil:
        fmt.Println("nil")
    case int:
        fmt.Printf("int: %d\n", v)
    case string:
        fmt.Printf("string: %s\n", v)
    default:
        fmt.Printf("unknown: %T\n", v)
    }
}
```

#### 14.3 回答模板

> 类型断言v.(Type)获取实际类型，失败panic，ok版安全返回。type switch判断任意类型。空interface{}和any等价。

---

## 第三章 并发篇（高频 ★★★★★）

### 15. Goroutine？

#### 15.1 启动

```go
// go关键字启动
go hello()

// 带参数
go func(msg string) {
    fmt.Println(msg)
}("hello")

// 并发执行
```

#### 15.2 Goroutine vs Thread

```
Thread：OS线程，KB级别
Goroutine：Go协程，KB级别，栈可增长，约2KB起步

Thread：创建销毁开销大
Goroutine：创建销毁极小

Thread：需要锁
Goroutine：channel通信CSP
```

#### 15.3 GMP模型

```
G：goroutine
M：machine（OS线程）
P：processor（调度器CPU）

P数量 = GOMAXPROCS
M = P*N + 一些extra M（系统线程）

调度：P调度G到M上
```

#### 15.4 回答模板

> Go用go关键字启动goroutine轻量协程，默认栈约2KB可增长，开销极小。GPM模型：G M P协程-线程-调度。运行时runtime调度，M=N个CPU+少量extra。Go并发模型天然适合高并发。

---

### 16. Channel通道？

#### 16.1 创建

```go
// 1. 无缓冲
ch := make(chan int)

// 2. 有缓冲
ch := make(chan int, 10)

// 3. 只读/只写
func read(ch <-chan int) {}
func write(ch chan<- int) {}
```

#### 16.2 操作

```go
// 发送
ch <- 1

// 接收
v := <-ch

// 关闭
close(ch)

for v := range ch {  // range遍历直到关闭
    fmt.Println(v)
}
```

#### 16.3 特性

```
无缓冲：阻塞直到接收<->发送同时（同步）
有缓冲：不阻塞直到缓冲区满（异步）
关闭后不能发送，读取关闭通道值零值
```

#### 16.4 回答模板

> Channel是Go独有CSP并发模式.Channel通信实现同步。make(chan int, 10)创建有缓冲。无缓冲必须发送接收同时。close()关闭，关闭后读取返回零值。

---

### 17. Select多路复用？

#### 17.1 select语句

```go
select {
case v := <-ch1:
    fmt.Println("ch1:", v)
case v := <-ch2:
    fmt.Println("ch2:", v)
case <-timer.C:
    fmt.Println("timeout")
default:
    fmt.Println("none blocking")
}
```

#### 17.2 特性

```
- 同时多个case就绪，随机选择一个执行
- 无就绪case，default执行，无default阻塞
- 用于非阻塞、超市、channel广播
```

#### 17.3 超时

```go
select {
case v := <-ch:
    return v
case <-time.After(time.Second):
    return errors.New("timeout")
}
```

#### 17.4 回答模板

> select类似switch，多channel监听就绪立即执行。default非阻塞，无default则阻塞。结合time.After实现超时。

---

### 18. sync包常用？

#### 18.1 WaitGroup

```go
var wg sync.WaitGroup
wg.Add(3)
go func() { defer wg.Done(); /*work*/ }()
wg.Wait()  // 等待完成
```

#### 18.2 Mutex

```go
var mu sync.Mutex
mu.Lock()
defer mu.Unlock()
// 临界区代码
```

#### 18.3 Once

```go
var once sync.Once
once.Do(func() {
    // 只执行一次
})
```

#### 18.4 Map并发

```go
var m sync.Map
m.Store("key", "value")  // 存
v, ok := m.Load("key")   // 取
m.Delete("key")
m.Range(func(key, value interface{}) bool {
    return true
})
```

#### 18.5 Pool对象池

```go
var pool sync.Pool
pool.Get()        // 取，可能nil
pool.Put(obj)    // 放入复用
```

#### 18.6 回答模板

> sync.WaitGroup计数等待完成、Mutex互斥锁、Once单次执行、Map并发安全、Pool对象池复用减少GC。

---

### 19. Context？

#### 19.1 Context���口

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

#### 19.2 根Context

```go
// TODO：空Context
ctx := context.TODO()

// Background：顶级
ctx := context.Background()
```

#### 19.3 With函数

```go
// WithCancel：传递取消
ctx, cancel := context.WithCancel(context.Background())

// WithDeadline：超时
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(context.Background(), deadline)

// WithTimeout：超时
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)

// WithValue：传递值
ctx := context.WithValue(parentCtx, "key", "value")
```

#### 19.4 使用

```go
func main() {
    ctx := context.Background()
    // 传递到HTTP请求、goroutine
    go worker(ctx)
}

func worker(ctx context.Context) {
    select {
    case <-ctx.Done():
        fmt.Println("done:", ctx.Err())
    }
}
```

#### 19.5 回答模板

> Context传递取消/timer/cancellation/value。WithCancel/WithDeadline/WithTimeout创建派生Context。HTTP/gRPC/goroutine传递取消信号。Background是根。value传递请求级数据。

---

### 20. 原子操作atomic？

#### 20.1 atomic操作

```go
import "sync/atomic"

// 整数原子操作
var i int64
atomic.AddInt64(&i, 1)
atomic.LoadInt64(&i)
atomic.StoreInt64(&i, 10)
atomic.CompareAndSwapInt64(&i, old, new)  // CAS

// 指针
var ptr *int
atomic.LoadPointer(&ptr)
atomic.StorePointer(&ptr, newPtr)
```

#### 20.2 使用场景

```go
// 计数器、标记位、自旋锁
// 比Mutex更轻量
```

#### 20.3 回答模板

> Go sync/atomic包提供原子操作，Add/Load/Store/CAS整数和指针。Lock-free无锁比Mutex更轻量适合低开销并发。低并发计数器、状态标记。

---

## 第四章 Go语言特性篇（中高频 ★★★★）

### 21. 错误panic和recover？

#### 21.1 panic

```go
// panic触发的异常
panic("something wrong")
// 打印堆栈，程序退出
```

#### 21.2 recover

```go
func test() {
    defer func() {
        if err := recover(); err != nil {
            // 恢复
            fmt.Println("recovered:", err)
        }
    }()
    panic("test")
}
```

#### 21.3 defer/recover模式

```go
func safeCall(fn func()) (err error) {
    defer func() {
        if e := recover(); e != nil {
            err = e.(error)  // 类型转为真正error
        }
    }()
    fn()
    return nil
}
```

#### 21.4 回答模板

> panic触发异常退出，recover恢复。defer+recover是Go异常处理模式，通常捕获panic转为error返回。recover只对直接调用它的defer函数有效，一个顶层defer足够。

---

### 22. Go module依赖管理？

#### 22.1 go.mod

```
go.mod模块定义：
module github.com/user/repo

go 1.21

require (
    github.com/pkg/errors v0.9.1
)
```

#### 22.2 go get

```
go get     下载
go mod tidy 清理依赖
go mod vendor 打包到vendor
```

#### 22.3 proxy

```
# GOPROXY代理
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOSUMDB=off

# 常用代理
https://goproxy.cndirect
```

#### 22.4 回答模板

> Go.mod定义module和依赖，go get下载，go mod tidy整理。环境变量GOPROXY代理常用GOPROXY.IO。国内GOPROXY设置go env -w GOPROXY=https://goproxy.cn,direct。

---

### 23. Slice底层原理？

#### 23.1 数据结构

```go
// slice header
type slice struct {
    array unsafe.Pointer  // 底层数组指针
    len int              // 长度
    cap int              // 容量
}
```

#### 23.2 扩容

```go
// append时容量不足
newcap := old.cap * 2
if newcap < required {
    newcap = required
}
newArray := make([]Type, newcap)
copy(newArray, oldArray)
```

#### 23.3 注意事项

```go
// 1. slice被截取后仍指向同一个底层数组
// 2. 谨慎：append超出cap会分配新array
// 3. 删除小心：截取值复制后修改原数组

s := []int{1,2,3,4,5}
sub := s[:3:3]  // 第三个是cap限制
```

#### 23.4 回答模板

> Slice是struct结构，len元素数、cap容量、array指针。append超出cap扩容2倍。slice指向底层array，子slice修改会影响元。注意子slice的cap=3限制。

---

### 24. Map底层原理？

#### 24.1 桶结构

```
bucket结构：
每bucket有8个bucket
hash高8位决定bucket位置
hash低8位在bucket内顺序

冲突处理：
- 少于8个key，顺序存
- 满8个key，新建bucket用next指针链接
```

#### 24.2 负载因子

```
loadFactor = count/bucketCount
> 6.5时 触发扩容
```

#### 24.3 回答模板

> Go Map用Bucket数组+bucket链表实现，hash分高低位分配位置。bucket8个槽位，冲突用next指针串联。超过负载因子自动扩容2倍。

---

### 25. 闭包Closure？

#### 25.1 闭包

```go
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

cnt := counter()  // cnt是闭包
cnt()  // 1
cnt()  // 2
```

#### 25.2 闭包变量

```go
// 闭包捕获外部变量by reference
for _, v := range []int{1,2,3} {
    go func() {
        // v是共享
        fmt.Println(v)
    }()
}
```

#### 25.3 回答模板

> 闭包是anonymous函数引用外部环境变量，变量by reference导致共享。通过循环创建goroutine获取当前值，for循环中创建closure需要显式捕获loop var:go func(v int){fmt.Println(v)}(v)

---

### 26. 接口值比较？

#### 26.1 接口比较

```go
// 接口类型可以比较
var er error = errors.New("error")
fmt.Println(er == er)  // true，same

var er2 error = errors.New("error")
fmt.Println(er == er2) // false，different

// 可比：底层类型可compar
var r io.Reader
var w io.Writer
```

#### 26.2 不可比

```go
// slice,maps,functions cannot be compared
var s []int
// s == s // compile error

var m map[string]int
// m == m // compile error
```

#### 26.3 回答模板

> Go接口可用==比较，与error具体type有关。Slice/Map/Function不能用==比较，reflect.DeepEqual可做深度比较。常用interface和error判等。

---

### 27. 类型嵌入Embedding？

#### 27.1 嵌入

```go
type Reader interface {
    Read([]byte) (int, error)
}

type Writer interface {
    Write([]byte) (int, error)
}

// 嵌入接口
type ReadWriter interface {
    Reader
    Writer
}

// 嵌入结构体
type MyReader struct {
    io.Reader
    // 嵌入不能改名
}
```

#### 27.2 方法提升

```go
type MyReader struct {
    io.Reader
}

var mr MyReader
mr.Read(p)  // 嵌入的Reader.Read方法
```

#### 27.3 回答模板

> Go用类型嵌入（非继承）实现组合，功能组合代替继承。interface嵌入interface，struct嵌入struct。嵌入方法直接提升用mr.Read()。

---

### 28. Go生成器Generators？

#### 28.1 生成

```go
// 生成器模式：channel
func fibonacci() chan int {
    ch := make(chan int)
    go func() {
        n1, n2 := 0, 1
        for {
            ch <- n1
            n1, n2 = n2, n1+n2
        }
    }()
    return ch
}

// 使用
for i := 0; i < 10; i++ {
    fmt.Println(<-fib())
}
```

#### 28.2 Generator作用

```
生成器的核心：
- lazy evaluation惰性求值
- 并发产生数据，调用方按需消费
- 生产者和消费者解耦
```

#### 28.3 回答模板

> Go生成器用channel实现，生产发送到channel goroutine，调用方接收。用于按需/流式数据。

---

### 29. Go JSON处理？

#### 29.1 编解码

```go
import "encoding/json"

// Marshal序列化
b, _ := json.Marshal(user)
string(b)

// Unmarshal反序列化
json.Unmarshal(b, &user)

// 结构体tag
type User struct {
    Name string `json:"name"`
    Age  int   `json:"age,omitempty"`
}

// 跳过、忽略
type User struct {
    Pass  string `json:"-"`
}
```

#### 29.2 Encoder/Decoder

```go
enc := json.NewEncoder(os.Stdout)
enc.Encode(user)  // 直接输出到writer

dec := json.NewDecoder(r)
dec.Decode(&user)  // 从reader解码
```

#### 29.3 回答模板

> encoding/json包的Marshal/Unmarshal序列化和反JSON。Tag用json定义key名称，omitempty表示零值忽略，-跳过字段。encoder/decoder流式API适合大数据。

---

### 30. 并发安全Map？

#### 30.1 sync.Map

```go
var m sync.Map
m.Store("key", 1)
v, ok := m.Load("key")
m.Delete("key")
// Load和Store非原子
```

#### 30.2 读写RWMutex

```go
type Map struct {
    mu    sync.RWMutex
    data map[string]interface{}
}

func (m *Map) Get(key string) (interface{}, bool) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    val, ok := m.data[key]
    return val, ok
}

func (m *Map) Set(key string, val interface{}) {
    m.mu.Lock()
    defer m.mu.Unlock()
    m.data[key] = val
}
```

#### 30.3 应对方法

```
sync.Map适合读多写少场景
高并发更新用RWMutex
更复杂用redis/mysql
```

#### 30.4 回答模板

> sync.Map适合读多写少case。高并发写case用sync.RWMutex包装自定义Map。更复杂用外部存储如Redis。

---

## 第五章 I/O与网络篇（中高级 ★★★★）

### 31. HTTP服务？

#### 31.1 简单服务

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello")
})
http.ListenAndServe(":8080", nil)
```

#### 31.2 Router

```go
mux := http.NewServeMux()
mux.HandleFunc("/api/users", handler.handleUsers)
mux.HandleFunc("/api/posts/", handler.handlePost)
server := &http.Server{Addr: ":8080", Handler: mux}
server.ListenAndServe()
```

#### 31.3 框架

```gin.gin's Router
// go get -u github.com/gin-gonic/gin

router := gin.Default()
router.GET("/user", func(c *gin.Context) {
    c.JSON(200, "user")
})
router.Run(":8080")
```

#### 31.4 中间件Middleware

```go
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        c.Next()  // 处理请求
        latency := time.Since(start)
        fmt.Println(latency)
    }
}

router.Use(Logger())
```

#### 31.5 回答模板

> 标准库net/http的http.HandleFunc创建服务，NewServeMux路由。流行框架Giniris.Echo。中间件Middleware统一日志/鉴权等。

---

### 32. TCP/UDP编程？

#### 32.1 TCP Server

```go
listener, err := net.Listen("tcp", ":8080")
conn, err := listener.Accept()
buf := make([]byte, 1024)
n, err := conn.Read(buf)
conn.Write([]byte("response"))
conn.Close()
```

#### 32.2 TCP Client

```go
conn, err := net.Dial("tcp", "localhost:8080")
conn.Write([]byte("request"))
conn.Close()
```

#### 32.3 UDP

```go
// Server
pc, _ := net.ListenPacket("udp", ":8080")
buf := make([]byte, 1024)
n, addr, _ := pc.ReadFrom(buf)
pc.WriteTo([]byte("reply"), addr)

// Client
conn, _ := net.Dial("udp", ":8080")
```

#### 32.4 回答模板

> net.Listen("tcp")创建监听，Accept读取连接读写，Close关闭。UDP是无连接Packet不需要Accept，直接ReadFrom/WriteTo。

---

### 33. 连接池？

#### 33.1 http.Transport

```go
transport := &http.Transport{
    MaxIdleConns:        100,    // 最大空闲连接
    MaxIdleConnsPerHost:  100,    // 每个host最大空闲
    IdleConnTimeout:     90 * time.Second,
}

client := &http.Client{
    Transport: transport,
    Timeout:   30 * time.Second,
}
```

#### 33.2 golang.org/x/net/html

```
这个http package的Transport自动维护连接池
不需要手动维护
```

#### 33.3 回答模板

> http.Transport参数MaxIdleConnsConns/PerHost控制连接池大小，复用HTTP connections减少tcp三次握手。合理设置connection pool减少延迟。

---

### 34. WebSocket？

#### 34.1 gorilla/websocket

```go
// go get github.com/gorilla/websocket

 upgrader := websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool { return true },
}
conn, _ := upgrader.Upgrade(w, r, nil)
for {
    mt, message, err := conn.ReadMessage()
    if err != nil {
        break
    }
    conn.WriteMessage(mt, message)
}
```

#### 34.2 回答模板

> gorilla/websocket是WebSocket流行库。Upgrader转换HTTP连接。ReadMessage/WriteMessage读消息。WebSocket全双工。

---

### 35. 文件操作？

#### 35.1 os包

```go
// 读取
data, err := os.ReadFile("file.txt")
str := string(data)

// 写入
os.WriteFile("file.txt", data, 0644)

// 创建/打开/关闭
f, _ := os.Create("file.txt")
f.WriteString("content")
f.Close()

// 目录操作
os.Mkdir("dir", 0755)
os.MkdirAll("path/a/b", 0755)
os.RemoveAll("path")
```

#### 35.2 路径

```go
filepath.Join(dir, "name.txt")  // 跨平台路径拼接
filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
    return nil
})
```

#### 35.2 文件夹操作

```
ioutil已废弃 go 1.16
改用os和path/filepath
os.ReadFile替代ioutil.ReadFile
```

#### 35.3 回答模板

> Go标准os包提供文件操作os.ReadFile/os.WriteFile/ioutil.ReadAll已弃用改os。filepath.Walk遍历文件树。Go的路径用filepath.Join跨平台处理。

---

## 附录：面试追问

1. **Go为什么删除GOTO？**
GOTO容易造成流程混乱，现代编程语言倾向不用。Go保留goto但不允许跨函数跳转。简化代码流程控制。

2. **Go为什么删除try-catch？**
异常exception影响API的设计和并发。Go选择返回error方式处理，让函数返回明确。保持简单。

3. **Go GC何时触发？**
go GC运行时机：后台定期运行、内存分配达到阈值、手动runtime.GC()。

4. **Go为什么不支持范型？**
范型从1.18支持go1.18。Go1.18之前用interface+reflection实现泛型。

5. **Go Module replace？**
replace用于模块替换，例如本地调试：
require (module v1.0.0)
replace (module v1.0.0 => ../local/path)

---

## 参考资料

- Go官方文档：https://golang.org/doc/
- Go语言设计与实现： draveness.me
- 《Go语言核心编程》

---

> 整理by Claude Code | Go面试高频100问