# 数据结构与算法夺命连环100问——算法工程师核心技术深度指南

> 本文档面向算法学习者，从入门到 LeetCode 刷题，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 时间复杂度？

#### 1.1 复杂度定义

> 时间复杂度是算法执行时间随输入规模增长的渐进行为，用大O记号表示。

```plaintext
O(1)         常数时间
O(log n)      对数时间
O(n)          线性时间
O(n log n)    线性对数时间
O(n²)         平方时间
O(2ⁿ)         指数时间
O(n!)         阶乘时间
```

#### 1.2 常见复杂度对比

```
n=1000000 时的比较：
log n      20
n          1,000,000
n log n    20,000,000
n²         10¹²        ❌不可用
2ⁿ         2^1000000   ❌不可能
```

#### 1.3 回答模板

> 时间复杂度是算法执行时间随输入规模增长的量级，用大O表示。常见：O(1)常数<O(log n)对数<O(n)线性<O(n log n)<O(n²)。O(n²)及以上在n较大时通常不可用。

---

### 2. 空间复杂度？

#### 2.1 空间定义

> 算法使用的内存空间随输入规模的增长关系。

```plaintext
O(1)       原地算法，只用常数空间
O(n)        线性空间（数组、slice等）
O(n²)       二维数组等
```

#### 2.2 例子

```go
// O(1) 原地交换
func swap(a, b *int) {
    *a, *b = *b, *a
}

// O(n) 新数组
func reverse(s []int) []int {
    n := make([]int, len(s))
    for i, v := range s {
        n[len(s)-1-i] = v
    }
    return n
}
```

#### 2.3 回答模板

> 空间复杂度是算法占用内存随输入规模的关系。常用O(1)原地、O(n)线性空间。递归算法注意调用栈空间。

---

### 3. 数组Array？

#### 3.1 数组特性

```
数组特点：
- 连续内存存储
- 随机访问 O(1)
- 插入/删除 O(n)

各操作复杂度：
访问     O(1)
查找     O(n)  遍历
插入     O(n)  平均移动
删除     O(n)  平均移动
```

#### 3.2 代码示例

```go
// 数组简单操作
arr := [5]int{1, 2, 3, 4, 5}
// 访问
x := arr[0]

// 动态数组（Slice）
slice := []int{1, 2, 3}
slice = append(slice, 4)
```

#### 3.3 回答模板

> 数组是连续内存，随机访问O(1)是其最大优势。插入删除需要移动元素平均O(n)。静态数组长度固定，动态数组（Slice）可自动扩容。

---

### 4. 链表LinkedList？

#### 4.1 链表特性

```
链表特点：
- 离散内存，通过指针连接
- 插入/删除 O(1)
- 随机访问 O(n)

分类：
- 单链表：next指针
- 双链表：prev + next
- 循环链表：尾指向头
```

#### 4.2 实现

```go
// 单链表节点
type ListNode struct {
    Val  int
    Next *ListNode
}

// 插入（O(1)）
func insert(node, new *ListNode) {
    new.Next = node.Next
    node.Next = new
}

// 删除（O(1)）
func delete(node *ListNode) {
    node.Next = node.Next.Next
}

// 反转
func reverse(head *ListNode) *ListNode {
    var prev *ListNode
    cur := head
    for cur != nil {
        next := cur.Next
        cur.Next = prev
        prev = cur
        cur = next
    }
    return prev
}
```

#### 4.3 回答模板

> 链表是离散存储，插入删除O(1)是其优势，但访问需要遍历O(n)。需要记住反转、环检测、倒数第K个节点等高频操作。

---

### 5. 栈Stack？

#### 5.1 特性

```
栈特性：LIFO - 后进先出
操作：
- push 入栈 O(1)
- pop 出栈 O(1)
- peek 查看栈顶 O(1)

应用场景：
- 函数调用栈
- 表达式求值
- 括号匹配
- 深度优先搜索
```

#### 5.2 实现

```go
type Stack struct {
    data []int
}

func (s *Stack) Push(x int) {
    s.data = append(s.data, x)
}

func (s *Stack) Pop() int {
    if len(s.data) == 0 {
        return 0
    }
    x := s.data[len(s.data)-1]
    s.data = s.data[:len(s.data)-1]
    return x
}

func (s *Stack) Peek() int {
    if len(s.data) == 0 {
        return 0
    }
    return s.data[len(s.data)-1]
}
```

#### 5.3 回答模板

> 栈是LIFO数据结构，入栈出栈O(1)。用于函数调用栈、括号匹配、DFS等场景。显式栈可以模拟递归。

---

### 6. 队列Queue？

#### 6.1 特性

```
队列特性：FIFO - 先进先出
操作：
- enqueue 入队 O(1)
- dequeue 出队 O(1)

分类：
- 普通队列
- 循环队列
- 双端队列 deque
```

#### 6.2 实现

```go
type Queue struct {
    data []int
}

func (q *Queue) Enqueue(x int) {
    q.data = append(q.data, x)
}

func (q *Queue) Dequeue() int {
    if len(q.data) == 0 {
        return 0
    }
    x := q.data[0]
    q.data = q.data[1:]
    return x
}
```

#### 6.3 回答模板

> 队列是FIFO数据结构，用于BFS广度优先搜索、生产者消费者模型。循环队列可优化空间。

---

### 7. 哈希表HashTable？

#### 7.1 特性

```
哈希表特性：
- Key-Value 键值对
- 查找 O(1) 平均
- 插入 O(1) 平均
- 空间换时间

哈希冲突：
- 链地址法（拉链）
- 开放地址法
- 再哈希法
```

#### 7.2 Go 实现

```go
// Go Map 就是哈希表
hash := make(map[string]int)
hash["key"] = 1
value, ok := hash["key"]
delete(hash, "key")

// 遍历
for k, v := range hash {
    fmt.Println(k, v)
}
```

#### 7.3 回答模板

> 哈希表通过哈希函数把Key映射到数组位置，平均常数时间查找。冲突用链地址法拉链表解决。Go的map是哈希表实现。

---

### 8. 二叉树BinaryTree？

#### 8.1 定义

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}
```

#### 8.2 遍历

```go
// 前序：根-左-右
func preorder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    result := []int{root.Val}
    result = append(result, preorder(root.Left)...)
    result = append(result, preorder(root.Right)...)
    return result
}

// 中序：左-根-右
func inorder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    result := inorder(root.Left)
    result = append(result, root.Val)
    result = append(result, inorder(root.Right)...)
    return result
}

// 后序：左-右-根
func postorder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    result := postorder(root.Left)
    result = append(result, postorder(root.Right)...)
    result = append(result, root.Val)
    return result
}

// 层级遍历 BFS
func levelorder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    queue := []*TreeNode{root}
    result := []int{}
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        result = append(result, node.Val)
        if node.Left != nil {
            queue = append(queue, node.Left)
        }
        if node.Right != nil {
            queue = append(queue, node.Right)
        }
    }
    return result
}
```

#### 8.3 回答模板

> 二叉树三种遍历：前序（根左右）中序（左根右）后序（左右根）递归实现。面试常考：树的深度、求第K层叶子数、最近公共祖先等。

---

### 9. 堆Heap？

#### 9.1 特性

```
堆：完全二叉树 + 堆序性
- 最大堆：父节点 >= 子节点
- ���小��：父节点 <= 子节点

操作：
- 建堆 O(n)
- 插入 O(log n)
- 取极值 O(1)
- 删除 O(log n)

应用：
- 优先级队列
- TopK问题
- 排序（堆排序）
```

#### 9.2 实现

```go
// Go 的 heap 接口
type IntHeap []int

func (h IntHeap) Len() int            { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] } // 最小堆
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *IntHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}
func (h *IntHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}

// 使用
h := &IntHeap{}
heap.Init(h)
heap.Push(h, 5)
heap.Push(h, 3)
heap.Pop() // 3
```

#### 9.3 回答模板

> 堆是完全二叉树，最大堆父节点大于子节点。堆用于优先级队列、TopK问题。go的container/heap实现了堆接口。

---

### 10. 图Graph？

#### 10.1 表示方法

```go
// 邻接表
adj := map[string][]string{
    "A": {"B", "C"},
    "B": {"A", "C", "D"},
    "C": {"A", "B", "D"},
    "D": {"B", "C"},
}

// 邻接矩阵
matrix := [][]int{
    {0, 1, 1, 0},
    {1, 0, 1, 1},
    {1, 1, 0, 1},
    {0, 1, 1, 0},
}
```

#### 10.2 遍历

```go
// BFS
func bfs(start string, adj map[string][]string) []string {
    visited := map[string]bool{start: true}
    queue := []string{start}
    result := []string{}

    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        result = append(result, node)

        for _, neighbor := range adj[node] {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
    return result
}

// DFS 递归
func dfs(node string, adj map[string][]string, visited map[string]bool, result *[]string) {
    visited[node] = true
    *result = append(*result, node)
    for _, neighbor := range adj[node] {
        if !visited[neighbor] {
            dfs(neighbor, adj, visited, result)
        }
    }
}
```

#### 10.3 回答模板

> 图的表示方法有邻接表和邻接矩阵。BFS找最短路径，DFS检查连通性。面试常考：岛屿数量、课程表顺序、环检测等。

---

## 第二章  LeetCode 高频题型篇（高频 ★★★★★）

### 11. 两数之和？

#### 11.1 题目

> 给定一个数组和一个目标值，返回两个数的下标，使它们的和等于目标值。

#### 11.2 解法

```go
// 解法1：暴力 O(n²)
func twoSum(nums []int, target int) []int {
    for i := 0; i < len(nums); i++ {
        for j := i + 1; j < len(nums); j++ {
            if nums[i]+nums[j] == target {
                return []int{i, j}
            }
        }
    }
    return nil
}

// 解法2：哈希表 O(n)
func twoSum(nums []int, target int) []int {
    m := make(map[int]int)
    for i, v := range nums {
        if j, ok := m[target-v]; ok {
            return []int{j, i}
        }
        m[v] = i
    }
    return nil
}
```

#### 11.3 回答模板

> 两数之和解法：暴力列举O(n²)或哈希表O(n)。哈希表记录已遍历的元素和下标，查找target-nums[i]。

---

### 12. 反转链表？

#### 12.1 题目

> 反转一个单链表。

#### 12.2 解法

```go
// 迭代
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    cur := head
    for cur != nil {
        next := cur.Next
        cur.Next = prev
        prev = cur
        cur = next
    }
    return prev
}

// 递归
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    newHead := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil
    return newHead
}
```

#### 12.3 回答模板

> 反转链表是高频基础题。迭代3指针（prev,cur,next），递归要理解reverseList(head.Next)返回新头结点。

---

### 13. 最大公约数/最小公倍数？

#### 13.1 公式

```
gcd(a, b) * lcm(a, b) = a * b

gcd: Greatest Common Divisor
lcm: Least Common Multiple
```

#### 13.2 辗转相除法

```go
func gcd(a, b int) int {
    if b == 0 {
        return a
    }
    return gcd(b, a%b)
}

func lcm(a, b int) int {
    return a * b / gcd(a, b)
}
```

#### 13.3 回答模板

> 最大公约数用欧几里得算法（辗转相除），递归或循环。最小公倍数=a*b/gcd。

---

### 14. 斐波那契数列？

#### 14.1 题目

> 计算第 n 个斐波那契数。

#### 14.2 解法

```go
// 解法1：递归（指数级，极慢）
func fib(n int) int {
    if n <= 1 {
        return n
    }
    return fib(n-1) + fib(n-2)
}

// 解法2：DP + 空间优化 O(n)
func fib(n int) int {
    if n <= 1 {
        return n
    }
    a, b := 0, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}

// 矩阵快速幂 O(log n)
func fibMatrix(n int) int {
    if n <= 1 {
        return n
    }
    m := [2][2]int{{1, 1}, {1, 0}}
    res := matrixPower(m, n-1)
    return res[0][0]
}
```

#### 14.3 回答模板

> 斐波那契递归有重复子问题导致指数复杂度。用DP迭代O(n)或矩阵快速幂O(log n)。面试会问到时间复杂度优化。

---

### 15. 合并两个有序数组？

#### 15.1 题目

> 合并两个有序数组为一个有序数组。

#### 15.2 解法

```go
// 从后往前，双指针
func merge(nums1, nums2 []int, m, n int) {
    p1, p2, p := m-1, n-1, m+n-1
    for p1 >= 0 && p2 >= 0 {
        if nums1[p1] > nums2[p2] {
            nums1[p] = nums1[p1]
            p1--
        } else {
            nums1[p] = nums2[p2]
            p2--
        }
        p--
    }
    for p2 >= 0 {
        nums1[p] = nums2[p2]
        p2--
        p--
    }
}
```

#### 15.3 回答模板

> 从后往前双指针避免元素覆盖。时间复杂度O(m+n)，空间复杂度O(1)。

---

### 16. 有效的括号？

#### 16.1 题目

> 判断括号序列是否合法。

#### 16.2 解法

```go
func isValid(s string) bool {
    stack := []rune{}
    pairs := map[rune]rune{'(': ')', '[': ']', '{': '}'}

    for _, c := range s {
        if c == '(' || c == '[' || c == '{' {
            stack = append(stack, c)
        } else if len(stack) == 0 || stack[len(stack)-1] != pairs[c] {
            return false
        } else {
            stack = stack[:len(stack)-1]
        }
    }
    return len(stack) == 0
}
```

#### 16.3 回答模板

> 用栈实现，遍历字符串，左括号入栈，右括号匹配栈顶是否匹配。类似题目：最长有效括号。

---

### 17. 爬楼梯？

#### 17.1 题目

> 爬n阶楼梯，每次可以走1步或2步，有多少种方法？

#### 17.2 解法

```go
// DP：第n阶 = 第n-1阶 + 第n-2阶（斐波那契）
func climbStairs(n int) int {
    if n <= 1 {
        return 1
    }
    a, b := 1, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}

// 通用方法：走1-m步
func climbStairs(n, m int) int {
    dp := make([]int, n+1)
    dp[0] = 1
    for i := 1; i <= n; i++ {
        for j := 1; j <= m && j <= i; j++ {
            dp[i] += dp[i-j]
        }
    }
    return dp[n]
}
```

#### 17.3 回答模板

> 这是经典DP，本质是斐波那契。第n阶等于第n-1阶方法数加上第n-2阶方法数。进阶：一次可以走1-m步。

---

### 18. 二分查找？

#### 18.1 标准二分

```go
func binarySearch(nums []int, target int) int {
    left, right := 0, len(nums)-1
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1
}
```

#### 18.2 变体

```go
// 查找左边界
func leftBound(nums []int, target int) int {
    left, right := 0, len(nums)
    for left < right {
        mid := left + (right-left)/2
        if nums[mid] < target {
            left = mid + 1
        } else {
            right = mid
        }
    }
    return left
}

// 查找右边界
func rightBound(nums []int, target int) int {
    left, right := 0, len(nums)
    for left < right {
        mid := left + (right-left)/2
        if nums[mid] <= target {
            left = mid + 1
        } else {
            right = mid
        }
    }
    return left - 1
}
```

#### 18.3 回答模板

> 二分查找时间复杂度O(log n)。注意边界条件：while条件(left <= right)，mid计算防止溢出(left+(right-left)/2)。

---

### 19. 快速排序？

#### 19.1 原理

```
快速排序：分治思想
1. 选择基准pivot
2. partition：pivot左边都是小，右边都是大
3. 递归处理左右两部���
```

#### 19.2 实现

```go
func quickSort(arr []int, low, high int) {
    if low < high {
        pivot := partition(arr, low, high)
        quickSort(arr, low, pivot-1)
        quickSort(arr, pivot+1, high)
    }
}

func partition(arr []int, low, high int) int {
    pivot := arr[high]
    i := low
    for j := low; j < high; j++ {
        if arr[j] < pivot {
            arr[i], arr[j] = arr[j], arr[i]
            i++
        }
    }
    arr[i], arr[high] = arr[high], arr[i]
    return i
}
```

#### 19.3 时间复杂度

```
平均：O(n log n)
最坏：O(n²)  （已排序）
最好：O(n log n)
```

#### 19.4 回答模板

> 快速排序用分治+原地分区，选最后一个元素作pivot。时间复杂度平均O(n log n)，最坏O(n²)。

---

### 20. 归并排序？

#### 20.1 原理

```
归并排序：分治+合并
1. 递归拆分到单个元素
2. 合并两个有序数组
```

#### 20.2 实现

```go
func mergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    mid := len(arr) / 2
    left := mergeSort(arr[:mid])
    right := mergeSort(arr[mid:])
    return merge(left, right)
}

func merge(left, right []int) []int {
    result := make([]int, 0, len(left)+len(right))
    i, j := 0, 0
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    return result
}
```

#### 20.3 时间复杂度

```
时间复杂度：O(n log n) 稳定
空间复杂度：O(n)
```

#### 20.4 回答模板

> 归并排序分治+合并，递归拆分到单个元素然后合并。时间复杂度稳定O(n log n)，空间O(n)。

---

## 第三章 算法思想篇（高频 ★★★★★）

### 21. 位运算？

#### 21.1 常见操作

```go
// 获取低位最后一个1
n & -n

// 判断奇数
n & 1 == 1

// 交换
a ^= b
b ^= a
a ^= b

// 清除最低位的1
n & (n - 1)

// 仅保留最低位的1
n & (-n)

// 判断是否是2的幂
n > 0 && (n & (n-1)) == 0
```

#### 21.2 异或性质

```
x ^ x = 0
x ^ 0 = x
a ^ b ^ a = b   （交换律/结合律）
```

#### 21.3 回答模板

> 位运算：&与 |或 ^异或 <<>>左移右移。常用技巧：交换、判断奇数、清除最低1、找单数（异或）等。

---

### 22. 动态规划DP？

#### 22.1 思想

```
动态规划：最优子结构 + 重叠子问题
1. 确定状态
2. 确定状态转移方程
3. 确定初始状态
4. 计算
```

#### 22.2 经典示例：背包

```go
// 0-1背包
func knapsack(W, n int, wt, val []int) int {
    dp := make([]int, W+1)
    for i := 0; i < n; i++ {
        for w := W; w >= wt[i]; w-- {
            dp[w] = max(dp[w], dp[w-wt[i]]+val[i])
        }
    }
    return dp[W]
}
```

#### 22.3 回答模板

> DP关键：状态定义、转移方程、初始值。常见类型：背包、最长公共子序列、斐波那契、股票买卖。

---

### 23. 贪心算法？

#### 23.1 思想

```
贪心：每步选择局部最优，期望获得全局最优
不一定是最优，但在很多场景近似最优
```

#### 23.2 经典示例：活动安排

```go
// 区间选择：优先选择结束最早的
func maxEvents(events [][]int) int {
    sort.Slice(events, func(i, j int) bool {
        return events[i][1] < events[j][1]
    })
    count := 0
    lastEnd := -1
    for _, e := range events {
        if e[0] > lastEnd {
            count++
            lastEnd = e[1]
        }
    }
    return count
}
```

#### 23.3 回答模板

> 贪心每步选择最优，常见问题：区间选择、区间覆盖、哈夫曼编码、 Dijkstra单源最短路径。

---

### 24. 回溯算法？

#### 24.1 思想

```
回溯：递归搜索 + 状态重置
类似深度优先搜索，走不通就退回
```

#### 24.2 全排列

```go
func permute(nums []int) [][]int {
    result := [][]int{}
    used := make([]bool, len(nums))
    path := []int{}

    var backtrack func()
    backtrack = func() {
        if len(path) == len(nums) {
            tmp := make([]int, len(path))
            copy(tmp, path)
            result = append(result, tmp)
            return
        }
        for i := 0; i < len(nums); i++ {
            if used[i] {
                continue
            }
            used[i] = true
            path = append(path, nums[i])
            backtrack()
            path = path[:len(path)-1]
            used[i] = false
        }
    }
    backtrack()
    return result
}
```

#### 24.3 回答模板

> 回溯用于组合、排列、子集等问题。用used数组标记选择，递归返回时重置状态。时间复杂度通常是 O(n!)

---

### 25. 分治算法？

#### 25.1 思想

```
分治：Divide and Conquer
1. 分解：将问题拆分成子问题
2. 解决：递归求解子问题
3. 合并：合并结果
```

#### 25.2 经典用例

```
- 归并排序
- 快速排序
- 二分查找
- 大整数乘法
- 最近点对
```

#### 25.3 回答模板

> 分治将大问题拆成小问题分别解决，最后合并结果。典型的有归并排序、快速排序、二分查找及其变体。

---

### 26. 双指针？

#### 26.1 类型

```
1. 对撞双指针：左右往中间靠
   - 有序数组两数之和
   - 反转字符串

2. 快慢指针：同向运动
   - 判定链表有环
   - 删除数组重复项
```

#### 26.2 示例

```go
// 有序数组两数之和
func twoSum(numbers []int, target int) []int {
    left, right := 0, len(numbers)-1
    for left < right {
        sum := numbers[left] + numbers[right]
        if sum == target {
            return []int{left + 1, right + 1}
        } else if sum < target {
            left++
        } else {
            right--
        }
    }
    return nil
}
```

#### 26.3 回答模板

> 双指针分对撞双指针（同向/反向）和快慢指针。对撞找两数之和，快慢判定环、删除重复。时间复杂度O(n)。

---

### 27. 滑动窗口？

#### 27.1 思想

```
滑动窗口：维护一个固定或可变大小的窗口
在窗口内进行计算，窗口滑动时更新
```

#### 27.2 示例

```go
// 最大子数组和
func maxSubArraySum(nums []int, k int) int {
    sum := 0
    maxSum := math.MinInt64
    for i := 0; i < len(nums); i++ {
        if i >= k {
            sum -= nums[i-k]
        }
        sum += nums[i]
        if i >= k-1 {
            maxSum = max(maxSum, sum)
        }
    }
    return maxSum
}
```

#### 27.3 回答模板

> 滑动窗口用于连续子数组问题。固定窗口大小用count，可变窗口用while调整左右边界。时间复杂度O(n)。

---

### 28. 前缀和？

#### 28.1 思想

```
前缀和：pre[i] = sum(nums[0:i+1])

快速求区间和：[l,r] = pre[r+1] - pre[l]
```

#### 28.2 示例

```go
// 子数组和为目标值
func subarraySum(nums []int, target int) []int {
    pre := 0
    m := make(map[int]int)
    m[0] = -1 // 初始化

    for i, v := range nums {
        pre += v
        if j, ok := m[pre-target]; ok {
            return []int{j + 1, i}
        }
        m[pre] = i
    }
    return nil
}
```

#### 28.3 回答模板

> 前缀和问题简化连续子数组计算。pre[i]表示前i项和，区间[l,r]=pre[r+1]-pre[l]。用于subarray sum相关问题。

---

### 29. 广度优先搜索BFS？

#### 29.1 思想

```
BFS：按层次遍历，使用队列
适合找最短路径、无权图最短路径
```

#### 29.2 示例

```go
// 二进制矩阵中的最短路径
func shortestPath(grid [][]int) int {
    if len(grid) == 0 || grid[0][0] == 1 {
        return -1
    }
    m, n := len(grid), len(grid[0])
    visited := make([][]bool, m)
    for i := 0; i < m; i++ {
        visited[i] = make([]bool, n)
    }

    queue := [][2]int{{0, 0}}
    visited[0][0] = true
    dirs := [][2]int{{-1, 0}, {1, 0}, {0, -1}, {0, 1}}
    steps := 1

    for len(queue) > 0 {
        size := len(queue)
        for i := 0; i < size; i++ {
            x, y := queue[0][0], queue[0][1]
            queue = queue[1:]
            if x == m-1 && y == n-1 {
                return steps
            }
            for _, d := range dirs {
                nx, ny := x+d[0], y+d[1]
                if nx >= 0 && nx < m && ny >= 0 && ny < n && !visited[nx][ny] && grid[nx][ny] == 0 {
                    visited[nx][ny] = true
                    queue = append(queue, [2]int{nx, ny})
                }
            }
        }
        steps++
    }
    return -1
}
```

#### 29.3 回答模板

> BFS用队列按层次遍历，适合找最短路径或层级关系。矩阵中使用dx/dy方向数组控制搜索。

---

### 30. 深度优先搜索DFS？

#### 30.1 思想

```
DFS：一条路走到黑，回溯再尝试
适合遍历、连通分量、迷宫问题
```

#### 30.2 示例

```go
// 岛屿数量
func numIslands(grid [][]int) int {
    if len(grid) == 0 {
        return 0
    }
    m, n := len(grid), len(grid[0])
    count := 0
    var dfs func(i, j int)
    dfs = func(i, j int) {
        if i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == '0' {
            return
        }
        grid[i][j] = '0'
        dfs(i+1, j)
        dfs(i-1, j)
        dfs(i, j+1)
        dfs(i, j-1)
    }

    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if grid[i][j] == '1' {
                count++
                dfs(i, j)
            }
        }
    }
    return count
}
```

#### 30.3 回答模板

> DFS递归或用栈实现，回溯时恢复状态。常用于岛屿问题、迷宫、连通分量。

---

## 第四章 LeetCode 高频100题分类篇（中高级 ★★★★）

### 31. TopK 问题？

#### 31.1 解法

```go
// 方法1：排序 O(n log n)
func topK1(nums []int, k int) []int {
    sort.Ints(nums)
    return nums[len(nums)-k:]
}

// 方法2：堆 O(n log k)
func topK2(nums []int, k int) []int {
    h := &IntHeap{}
    heap.Init(h)
    for _, v := range nums {
        heap.Push(h, v)
        if h.Len() > k {
            heap.Pop(h)
        }
    }
    result := make([]int, k)
    for i := k - 1; i >= 0; i-- {
        result[i] = heap.Pop(h).(int)
    }
    return result
}

// 方法3：快速选择 O(n)
func topK3(nums []int, k int) []int {
    // 快速选择
    return nums
}
```

#### 31.2 回答模板

> TopK三种解法：排序O(n log n)、堆O(n log k)、快速选择O(n)。数据量大用堆，数据量小用排序。

---

### 32. LRU缓存？

#### 32.1 实现

```go
type LRUCache struct {
    capacity int
    cache    map[int]*list.Element
    list     *list.List
}


type entry struct {
    key, value int
}


func Constructor(capacity int) LRUCache {
    return LRUCache{
        capacity: capacity,
        cache:    make(map[int]*list.Element),
        list:     list.New(),
    }
}


func (c *LRUCache) Get(key int) int {
    if e, ok := c.cache[key]; ok {
        c.list.MoveToFront(e)
        return e.Value.(*entry).value
    }
    return -1
}


func (c *LRUCache) Put(key int, value int) {
    if e, ok := c.cache[key]; ok {
        e.Value.(*entry).value = value
        c.list.MoveToFront(e)
        return
    }
    e := c.list.PushFront(&entry{key, value})
    c.cache[key] = e
    if c.list.Len() > c.capacity {
        old := c.list.Back()
        c.list.Remove(old)
        delete(c.cache, old.Value.(*entry).key)
    }
}
```

#### 32.2 回答模板

> LRU用HashMap+双向链表实现，Map O(1)查找，链表维护顺序。Go用container/list实现双向链表。

---

### 33. 字符串匹配 KMP？

#### 33.1 KMP 原理

```
KMP：最长公共前后缀
避免暴力匹配的回溯
```

#### 33.2 实现

```go
func kmp(text, pattern string) bool {
    if len(pattern) == 0 {
        return true
    }
    pi := computePI(pattern)

    j := 0
    for i := 0; i < len(text); i++ {
        for j > 0 && text[i] != pattern[j] {
            j = pi[j-1]
        }
        if text[i] == pattern[j] {
            j++
        }
        if j == len(pattern) {
            return true
        }
    }
    return false
}

func computePI(pattern string) []int {
    pi := make([]int, len(pattern))
    j := 0
    for i := 1; i < len(pattern); i++ {
        for j > 0 && pattern[i] != pattern[j] {
            j = pi[j-1]
        }
        if pattern[i] == pattern[j] {
            j++
        }
        pi[i] = j
    }
    return pi
}
```

#### 33.3 回答模板

> KMP核心next数组（部分匹配表），避免暴力匹配BF的回溯。时间复杂度O(m+n)。

---

### 34. 股票买卖问题？

#### 34.1 通解

```go
// 买卖股票最佳时机（无限交易）
func maxProfit(prices []int) int {
    profit := 0
    for i := 1; i < len(prices); i++ {
        if prices[i] > prices[i-1] {
            profit += prices[i] - prices[i-1]
        }
    }
    return profit
}

// 最多K次交易
func maxProfitK(k int, prices []int) int {
    n := len(prices)
    if k >= n/2 {
        return maxProfit(prices)
    }

    dp := make([][2]int, k+1)
    for i := 0; i <= k; i++ {
        dp[i][0] = 0
        dp[i][1] = -prices[0]
    }

    for i := 1; i < n; i++ {
        for j := 1; j <= k; j++ {
            dp[j][0] = max(dp[j][0], dp[j][1]+prices[i])
            dp[j][1] = max(dp[j][1], dp[j-1][0]-prices[i])
        }
    }
    return dp[k][0]
}
```

#### 34.2 回答模板

> 股票买卖有多种变体：无交易次数限制（当天买卖）、冷冻期、有限交易次数。核心用DP：dp[天数][状态]，状态0持有现金1持有股票。

---

### 35. 打家劫舍？

#### 35.1 解法

```go
// 打家劫舍 I
func rob(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }
    prev, curr := nums[0], max(nums[0], nums[1])
    for i := 2; i < len(nums); i++ {
        prev, curr = curr, max(curr, prev+nums[i])
    }
    return curr
}

// 打家劫舍 II（环形）
func rob2(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }
    f := func(nums []int) int {
        prev, curr := 0, nums[0]
        for i := 1; i < len(nums); i++ {
            prev, curr = curr, max(curr, prev+nums[i])
        }
        return curr
    }
    return max(f(nums[:len(nums)-1]), f(nums[1:]))
}
```

#### 35.2 回答模板

> 一维DP：dp[i]=max(dp[i-1], dp[i-2]+nums[i])。环形房间偷第一家或最后一家。

---

### 36. 最长递增子序列 LIS？

#### 36.1 解法

```go
// DP O(n²)
func lengthOfLIS(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    dp := make([]int, len(nums))
    for i := 0; i < len(nums); i++ {
        dp[i] = 1
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] && dp[j]+1 > dp[i] {
                dp[i] = dp[j] + 1
            }
        }
    }
    return maxSlice(dp)
}

// 二分查找 O(n log n)
func lengthOfLIS2(nums []int) int {
    piles := 0
    x := make([]int, len(nums))
    for _, num := range nums {
        i := binarySearch(x, num)
        x[i] = num
        if i == piles {
            piles++
        }
    }
    return piles
}
```

#### 36.2 回答模板

> 最长递增子序列两种方法：DP O(n²)或二分+tails数组O(n log n)。 tails[i]表示长度为i+1的递增子序列最小结尾。

---

### 37. 全排列？

#### 37.1 解法

```go
func permute(nums []int) [][]int {
    result := [][]int{}
    used := make([]bool, len(nums))
    path := []int{}

    var dfs func()
    dfs = func() {
        if len(path) == len(nums) {
            tmp := make([]int, len(path))
            copy(tmp, path)
            result = append(result, tmp)
            return
        }
        for i := 0; i < len(nums); i++ {
            if used[i] {
                continue
            }
            used[i] = true
            path = append(path, nums[i])
            dfs()
            path = path[:len(path)-1]
            used[i] = false
        }
    }
    dfs()
    return result
}
```

#### 37. 2 回答模板

> 全排列用backtrack + used数组标记选择。递归返回记得重置状态。时间复杂度O(n!)。

---

### 38. 子集？

#### 38.1 解法

```go
func subsets(nums []int) [][]int {
    result := [][]int{}
    path := []int{}

    var dfs func(idx int)
    dfs = func(idx int) {
        tmp := make([]int, len(path))
        copy(tmp, path)
        result = append(result, tmp)

        for i := idx; i < len(nums); i++ {
            path = append(path, nums[i])
            dfs(i + 1)
            path = path[:len(path)-1]
        }
    }
    dfs(0)
    return result
}
```

#### 38. 2 回答模板

> 子集是组合问题的变形，要包含空集。回溯时从当前位置idx开始，保持选择或不选择两种分支。

---

### 39. 单词搜索/矩阵搜索？

#### 39.1 解法

```go
// 单词搜索
func exist(board [][]byte, word string) bool {
    m, n := len(board), len(board[0])

    var dfs func(i, j, k int) bool
    dfs = func(i, j, k int) bool {
        if k == len(word) {
            return true
        }
        if i < 0 || i >= m || j < 0 || j >= n || board[i][j] != word[k] {
            return false
        }
        temp := board[i][j]
        board[i][j] = '*'
        found := dfs(i+1, j, k+1) || dfs(i-1, j, k+1) ||
            dfs(i, j+1, k+1) || dfs(i, j-1, k+1)
        board[i][j] = temp
        return found
    }

    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if dfs(i, j, 0) {
                return true
            }
        }
    }
    return false
}
```

#### 39. 2 回答模板

> 单词搜索用DFS+回溯遍历四条路径，注意标记已访问避免重复使用。典型面试题。

---

### 40. 岛屿问题？

#### 40.1 解法

```go
// 岛屿的最大面积
func maxAreaOfIsland(grid [][]int) int {
    m, n := len(grid), len(grid[0])
    var dfs func(i, j int) int
    dfs = func(i, j int) int {
        if i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == 0 {
            return 0
        }
        grid[i][j] = 0
        return 1 + dfs(i+1, j) + dfs(i-1, j) + dfs(i, j+1) + dfs(i, j-1)
    }

    maxArea := 0
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if grid[i][j] == 1 {
                maxArea = max(maxArea, dfs(i, j))
            }
        }
    }
    return maxArea
}
```

#### 40. 2 回答模板

> 岛屿问题核心：遍历到1时沉没（置0），dfs求岛屿面积。时O(m*n)，额外空间递归栈O(m*n)。

---

## 附录：算法面试高频100题分类速查

### 简单难度（高频）
- 两数之和、反转链表、有效括号
- 二分查找、爬楼梯、最大公约数
- 合并两个有序数组、反转字符串

### 中等难度（高频）
- 子数组最大和、最小覆盖子串
- 岛屿问题、子序列问题
- 股票买卖、打家劫舍、LIS/LCS

### 困难难度（参考）
- 接雨水、编辑距离
- 最短超级串、流星雨

### 常见套路
- 二分必背
- 哈希表：两数之和、快乐数
- 双指针：滑动窗口、合并数组
- 动态规划：斐波那契、股票买卖、背包
- 回溯/DFS/BFS：岛屿、全排列、子集

---

> 整理by Claude Code | 算法面试高频100问