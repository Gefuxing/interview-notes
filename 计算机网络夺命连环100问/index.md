# 计算机网络夺命连环100问——网络核心技术深度指南

> 本文档面向计算机网络学习者，涵盖高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 基础概念篇（高频 ★★★★★）

### 1. 计算机网络五层模型？

#### 1.1 OSI vs TCP/IP

```
OSI七层：
应用层 → 表示层 → 会话层 → 传输层 → 网络层 → 数据链路层 → 物理层

TCP/IP五层：
应用层 → 传输层 → 网络层 → 数据链路层 → 物理层
```

#### 1.2 各层作用

```
物理层：比特传输（电/光/无线信号）
数据链路层：帧传输/MAC寻址/差错检测
网络层：路由选择/IP地址/分组转发
传输层：可靠传输/UDP/TCP/端口
应用层：HTTP/FTP/SMTP/DNS
```

#### 1.3 常见协议

```
应用层：HTTP HTTPS DNS FTP SMTP SSH DHCP
传输层：TCP UDP
网络层：IP ICMP IGMP ARP
数据链路层：MAC VLAN PPP HDLC
```

#### 1.4 回答模板

> 计算机网络通常讲五层模型：应用层（HTTP/FTP）、传输层（TCP/UDP）、网络层（IP）、数据链路层（MAC）、物理层。比OSI少三层因为会话层表示层功能合并到应用层。HTTP是网页、TCP是可靠传输、UDP是无连接、IP是寻址路由、MAC是网卡地址。

---

### 2. TCP三次握手四次挥手？

#### 2.1 三次握手

```
Client → SYN → Server
Client ← SYN-ACK ← Server
Client → ACK → Server
连接建立！

第一次：SYN=1, Seq=x
第二次：SYN=1, ACK=1, Seq=y, Ack=x+1
第三次：ACK=1, Seq=x+1, Ack=y+1
```

**为什么三次？**
- 第一次Client发SYN可能重传
- 第二次Server确认并收到
- 第三次Server确认Client收到
- 避免Server端资源浪费

#### 2.2 四次挥手

```
Client → FIN → Server (我要断开)
Client ← ACK ← Server (知道了)
Client ← FIN ← Server (我也断开)
Client → ACK → Server (好的)
连接关闭！

第一次：FIN=1, Seq=u
第二次：ACK=1, Ack=u+1
第三次：FIN=1, Seq=v
第四次：ACK=1, Ack=v+1
```

**为什么四次？**
- 任何一方都可能先断开
- 需要双方确认断开
- 等待数据传完才能关闭

#### 2.3 状态变迁

```
Client: LISTEN → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
Server: LISTEN → SYN_RCVD → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

#### 2.4 回答模板

> TCP三次握手建立连接：SYN→SYN-ACK→ACK。四次挥手断开连接：FIN→ACK→FIN→ACK。三次握手防止Server被SYN攻击浪费资源，四次挥手保证数据完整传输因为任意一方都可能还有数据没发完。2MSL等待确保对方收到最后一个ACK。

---

### 3. TCP和UDP区别？

#### 3.1 对比表

| 特性 | TCP | UDP |
|------|-----|-----|
| 连接 | 面向连接 | 无连接 |
| 可靠 | 可靠传输 | 尽力而为 |
| 顺序 | 保证顺序 | 无顺序 |
| 速度 | 慢 | 快 |
| 头部 | 20字节 | 8字节 |
| 拥塞 | 有 | 无 |
| 场景 | 网页/邮件/文件 | 直播/DNS/语音 |

#### 3.2 首部格式

```
TCP（20-60字节）：
┌────────┬────────┬────────┬────────┐
│源端口(2)│目的端口(2)│seq(4)   │ack(4)   │
├────────┴────────┬─┴────────┼────────┤
│首部长度│控制标志 │窗口(2)  │校验(2) │
│紧急指针│选项(如) │
└─────────────────────────────┘

UDP（8字节）：
┌────────┬────────┬────────┬────────┐
│源端口  │目的端口│长度   │校验   │
└────────┴────────┴────────┴────────┘
```

#### 3.3 应用场景

```
TCP：HTTP、HTTPS、FTP、SMTP、POP3、SSH、Telnet
UDP：DNS、DHCP、TFTP、RTP、SNMP、VoIP、实时视频
```

#### 3.4 回答模板

> TCP和UDP核心区别在可靠性。TCP是面向连接、可靠传输、保证顺序、有拥塞控制，像打电话；UDP是无连接、尽力而为、不保证顺序、无拥塞控制，像发短信。TCP头部20-60字节含seq/ack/flags等用于可靠，UDP只有8字节首部简洁。HTTP/HTTPS/SSH用TCP，DNS/实时视频/Voice用UDP。

---

### 4. HTTP和HTTPS区别？

#### 4.1 HTTP vs HTTPS

```
HTTP（明文）：
┌───────────┐
│  Port:80   │
│  无加密   │
│  无身份验证│
└───────────┘

HTTPS（加密）：
┌────────────┐
│  Port:443   │
│  TLS/SSL加密│
│  证书认证   │
└────────────┘
```

#### 4.2 TLS握手

```
1. Client Hello → 服务器支持的加密套件
2. Server Hello → 选择加密套件+证书
3. 客户端验证证书 → 提取公钥
4. Client生成预主密钥 → 用公钥加密发送
5. 服务端用私钥解密 → 双方得到主密钥
6. 切换加密通道
```

#### 4.3 SSL/TLS协议

```
TLS握手：
- RSA密钥交换（不前向保密）
- DH/DHE（DH + Forward Secrecy）
- ECDHE（椭圆曲线DH + Forward Secrecy）

加密算法：
- 对称加密：AES、3DES、RC4
- 非对称加密：RSA、ECC
- 哈希：SHA-256、MD5
```

#### 4.4 回答模板

> HTTP是明文传输用80端口，HTTPS用443端口在TCP基础上加TLS/SSL层加密。TLS握手先用证书认证身份、交换加密参数、生成会话密钥，之后用对称加密传输数据。握手用RSA或ECDHE实现前向保密，推荐ECDHE + AES保证安全。证书由CA签发验证服务器身份。

---

### 5. Cookie和Session？

#### 5.1 概念

```
Cookie：
- 存储在客户端
- key=value形式
- 有过期时间
- 大小限制4KB

Session：
- 服务端存储
- SessionID识别
- 依赖Cookie存储SessionID
```

#### 5.2 认证流程

```
1. 登录 → 服务端创建Session → 返回SessionID
2. 浏览器Cookie存储SessionID
3. 后续请求带Cookie → 服务端验证
4. 登出 → 删除Session
```

#### 5.3 Session问题

```
分布式Session：
- 问题：多台服务器Session不同步
- 解决：Session复制/Sticky/nginx/共享Session(Redis/Memcached)
```

#### 5.4 回答模板

> HTTP无状态，用Cookie和Session配合实现状态管理。用户登录后服务端存Session返回SessionID到Cookie，后续带Cookie请求服务端验证实现登录状态。分布式环境下需要把Session存Redis/Memcached共享，或用Nginx Sticky让固定IP到固定服务器。

---

### 6. DNS解析过程？

#### 6.1 DNS查询

```
递归查询：
Client → 本地DNS → 根DNS → 顶级DNS → 权威DNS → 返回

1. 浏览器缓存
2. 系统hosts文件
3. 本地DNS缓存
4. ISP DNS服务器（递归）
5. 根DNS→.com→example→IP地址
```

#### 6.2 DNS记录类型

```
A：Address，域名→IPv4
AAAA：域名→IPv6
CNAME：别名
MX：邮件服务器
NS：域名服务器
SOA：授权起始
TXT：文本记录
PTR：反向解析
```

#### 6.3 DNS安全

```
DNS污染：返回错误IP
DNS劫持：中间人篡改
DNS隧道：C&C通信
DNSSEC：验证DNS响应真实性
```

#### 6.4 回答模板

> DNS把域名解析成IP地址。本地先查缓存，没有就向DNS服务器递归查询，最终从权威DNS拿到IP。A记录是���常��的，还有CNAME别名、MX邮件服务器。DNS常见安全问题有污染返回错误IP、劫持篡改回复，DNSSEC可以验证响应真实性。

---

### 7. HTTP状态码？

#### 7.1 状态码分类

```
1xx：信息响应
2xx：成功
3xx：重定向
4xx：客户端错误
5xx：服务端错误
```

#### 7.2 常见状态码

```
1xx：
100 Continue

2xx：
200 OK
201 Created
204 No Content

3xx：
301 Moved Permanently
302 Found (临时)
304 Not Modified (缓存)

4xx：
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
405 Method Not Allowed
429 Too Many Requests

5xx：
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

#### 7.3 回答模板

> HTTP状态码含义：2xx成功、3xx重定向、4xx客户端错误、5xx服务端错误。记住最常用的：200成功、301/302重定向、304缓存、401未认证、403禁止、404找不到、500/502/503服务端问题。浏览器和爬虫对不同错误码处理方式不同，SEO关注301/302区别。

---

### 8. IP地址分类？

#### 8.1 IPv4地址

```
五类地址：
A类：0xxxxxxx │ 1-126.x.x.x │ 大型组织
B类：10xxxxxx │ 128-191.x.x.x │ 中型组织
C类：110xxxxx │ 192-223.x.x.x │ 小型组织
D类：1110xxxx │ 224-239.x.x.x │ 组播
E类：1111xxxx │ 240-255.x.x.x │ 保留

特殊地址：
127.0.0.1 本地回环
0.0.0.0 本网络（本机）
255.255.255.255 广播
内网段：10.x.x.x、172.16-31.x.x、192.168.x.x
```

#### 8.2 子网划分

```
子网掩码：255.255.255.0 = /24

CIDR：10.0.0.0/16 = 10.0.0.0~10.0.255.255
     │  │  │
     │  │  └──── 16位网络号
     │  └───────┘ 主机数=65534
```

#### 8.3 NAT

```
NAT（网络地址转换）：
- 多个内网IP共享一个公网IP
- 端口映射PAT（端口复用）
- 解决IPv4地址不够
```

#### 8.4 回答模板

> IPv4分ABCDE五类，内网段：10.x、172.16-31.x、192.168.x是不规划ISP分配的地址。企业常用/16或/24子网。NAT把内网IP转成公网IP解决地址不够问题。CIDR用斜杠表示如/24表示255.255.255.0。

---

### 9. ARP协议？

#### 9.1 ARP作用

```
ARP：IP→MAC地址

同一个局域网内通信需要MAC地址！
流程：
1. 检查本地ARP缓存
2. 广播ARP请求（IP+MACA）
3. 目标响应ARP.reply
4. 更新本地ARP缓存
```

#### 9.2 ARP欺骗

```
ARP欺骗原理：
- 欺骗说我IP地址是你的网关MAC
- 流量经过攻击者
- 中间人攻击/MAN IN MIDDLE

防御：
- 静态ARP绑定表
- ARP防火墙
- 交换机动态ARP检查
```

#### 9.3 回答模板

> ARP把IP地址转换成MAC地址因为局域网内通信需要MAC地址。流程是先查本地ARP表，没有就广播请求目标返回后缓存。ARP欺骗是把自己的MAC伪造成网关，实施中间人攻击，防御用静态绑定或防火墙。

---

### 10. 网络设备区别？

#### 10.1 设备对比

```
物理层：Repeater（中继器）、Hub（集线器）——放大信号
数据链路层：Switch（交换机）——根据MAC转发
网络层：Router（路由器）——根据IP转发

集线器：所有端口在一个冲突域
交换机：每个端口是独立冲突域
路由器：每个端口是独立广播域
```

#### 10.2 交换机分类

```
二层交换机：MAC转发
三层交换机：支持路由功能（一次路由多次交换）
四层交换机：支持负载均衡
```

#### 10.3 VLAN

```
VLAN（虚拟局域网）：
- 划分广播域
- 安全隔离
- 方便管理
- 802.1Q打标签

常用VLAN：1-4094
```

#### 10.4 回答模板

> 中继器和集线器是物理层放大信号，交换机是数据链路层根据MAC转发，路由器是网络层根据IP转发。交换机每个端口是独立冲突域，全双工不冲突。VLAN是用来隔离广播域的逻辑分组，802.1Q给报文打标签实现跨交换机VLAN。

---

## 第二章 传输层篇（高频 ★★★★★）

### 11. TCP滑动窗口？

#### 11.1 滑动窗口

```
TCP滑动窗口：控制发送速度

发送方窗口：已发送待确认 ← 可发送 → 不发
┌─────────────────────────────┐
│  Sent and ACKed  │  Sent not│ Ready │
│   (left edge)    │ ACKed   │to send │
└───────────────────┼─────────┼──────────┘
                   │         │
              &lt;- Window Size -&gt;
```

#### 11.2 流量控制

```
接收方窗口大小 = rwnd = buffer - 使用

发送方每次根据接收方声明的window调整发送量
rwnd=0时发送方停止发送，开始探测
```

#### 11.3 拥塞控制

```
拥塞控制算法：
1. 慢启动：cwnd=1指数增长到ssthresh
2. 拥塞避免：cwnd线性增长
3. 超时/丢包：ssthresh=cwnd/2，cwnd=1
4. 快速重传：触发3次dupACK进入快速恢复

cwnd增长：
- 慢启动：指数 cwnd *= 2
- 拥塞避免：线性 cwnd += 1 MSS
```

#### 11.4 回答模板

> TCP滑动窗口是流量控制机制，接收方通过广告window size告诉发送方自己缓冲区还有多少可用。发送方不能超过这个窗口发送。拥塞控制是另外一套机制，用cwnd控制，算法包括慢启动指数增长、拥塞避免线性增长、超时和快速重传时ssthresh减半。

---

### 12. TCP可靠性保证？

#### 12.1 可靠性机制

```
1. 序列号：Seq标识每个字节
2. 确认ACK：累计确认
3. 超时重传：RTO计时器
4. 校验和：Header+Data校验
5. 流量控制：Window机制
6. 拥塞控制：cwnd控制
```

#### 12.2 超时重传

```
RTT：Round Trip Time 往返时间
RTO：Retransmission Timeout 重传超时

RTO计算：
SRTT = (1-α)*SRTT + α*RTTnew
RTTVAR = (1-β)*RTTVAR + β*|SRTT-RTTnew|
RTO = SRTT + 4*RTTVAR
```

#### 12.3 快速重传

```
触发条件：连续3次相同ACK

Fast Retransmit：
收到3次dupACK → 不等timeout直接重传

Fast Recovery：
cwnd = cwnd/2 + 3 MSS
```

#### 12.4 回答模板

> TCP可靠性靠：序列号标识每个字节、确认ACK确认收到、超时计时器触发重传、校验和校验错误。丢包检测有两种：超时和快速重传（收到3次重复ACK），后者更快恢复。RTO根据RTT动态计算。

---

### 13. Socket编程？

#### 13.1 基础API

```c
// 服务端
socket()        // 创建socket
bind()          // 绑定地址端口
listen()        // 监听
accept()       // 接收连接
read()/write() // 读写
close()         // 关闭

// 客户端
socket()        // 创建socket
connect()       // 连接服务端
read()/write() // 读写
close()         // 关闭
```

#### 13.2 Server客户端通信

```python
# 服务端
import socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('0.0.0.0', 8888))
server.listen(5)
conn, addr = server.accept()
data = conn.recv(1024)
conn.send(b'Hello')
conn.close()

# 客户端
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('127.0.0.1', 8888))
client.send(b'World')
data = client.recv(1024)
client.close()
```

#### 13.3 IO模型

```
阻塞IO：wait直到数据ready
非阻塞IO：立即返回，轮询检测
IO多路复用：select/poll/epoll
异步IO：内核通知完成
```

#### 13.4 回答模板

> Socket是网络编程基础API，TCP用SOCK_STREAM、DGram用SOCK_DGRAM。服务器先socket创建、bind绑定、listen监听、accept接收连接后读写。客户端connect连接后读写。五种IO模型：Blocking/Nonblocking/Select/Epoll/AsyncIO，生产环境Linux用Epoll高效处理万级并发连接。

---

### 14. 粘包和拆包？

#### 14.1 问题

```
TCP粘包：多条消息粘一起
TCP拆包：一条消息分多次收到

原因：
- TCP是流协议，没有消息边界
- 发送方缓冲区累积多条一起发
- 接收方缓冲区一次没读完
```

#### 14.2 解决方案

```
1. 固定长度：每次读取N字节
2. 固定Header：先读长度再读数据
3. 特殊分隔符：如\r\n
4. 序列化：Protocol Buffers/JSON
```

#### 14.3 示例

```
# Header+Length
Header[4字节表示Data长度] + Data[Length字节]

# 特殊分隔符
消息内容 +\r\n
消息内容 +\r\n
```

#### 14.4 回答模板

> TCP是流协议，没有消息边界，发送多条可能粘一起，读多次可能拆开。解决方案是自定义应用层协议：固定长度法、Header+Length法、定界符法或JSON/Protobuf序列化。Http的Content-Length和chunked也是类似原理。

---

### 15. HTTP请求方法？

#### 15.1 方法

```
GET：获取资源
POST：提交创建资源
PUT：替换资源（全量）
PATCH：部分修改
DELETE：删除资源
HEAD：只获取Header
OPTIONS：允许的方法
CONNECT：建立隧道
TRACE：回显
```

#### 15.2 安全和幂等

```
幂等：N次请��=1次结果
- GET：幂等 ✓
- PUT：幂等 ✓
- DELETE：幂等 ✓
- POST：非幂等 ✗
- PATCH：非幂等 ✗

安全：不改变服务器状态
- GET：安全 ✓
- HEAD：安全 ✓
- POST：不安全 ✗
- PUT：不安全 ✗
```

#### 15.3 REST API

```
GET /users - 获取用户列表
GET /users/:id - 获取单个用户
POST /users - 创建用户
PUT /users/:id - 全量更新
PATCH /users/:id - 部分更新
DELETE /users/:id - 删除用户
```

#### 15.4 回答模板

> HTTP方法GET获取、POST创建、PUT全量更新、PATCH部分更新、DELETE删除。关注幂等性，GET/PUT/DELETE幂等POST不幂等。RESTful风格用名词动词对应增删改查。

---

### 16. 长连接和短连接？

#### 16.1 HTTP长连接

```
HTTP/1.0：Short Connection
- 每个请求单独TCP连接
- 多次连接开销大

HTTP/1.1：Long Connection (默认Keep-Alive)
- 多个请求共用一个TCP连接
- 减少建连开销
- header: Connection: keep-alive
```

#### 16.2 HTTP/2

```
HTTP/2特性：
- 多路复用一个TCP连接
- Header压缩HPACK
- Server Push服务端推送
- 二进制分帧
- Stream优先级
```

#### 16.3 WebSocket

```
全双工通信：
- HTTP Upgrade建立连接
- 之后双方都可发消息
- 支持实时推送

心跳保活：
- Ping-Pong frame
- 心跳检测
```

#### 16.4 回答模板

> HTTP/1.1默认长连接Keep-Alive避免反复建连。HTTP/2进一步支持多路复用一个连接减少队头阻塞，还支持Header压缩和Server Push。WebSocket是全双工通道建立后双方可随时发消息，常用于需要实时推送的场景。

---

### 17. 网络抓包分析？

#### 17.1 常用工具

```
tcpdump：Linux命令行抓包
Wireshark：图形化分析
Charles：HTTP代理抓包
Fiddler：HTTP/HTTPS代理
```

#### 17.2 tcpdump示例

```bash
# 抓取指定端口
tcpdump -i eth0 port 80

# 抓取指定主机
tcpdump -i eth0 host 1.2.3.4

# 抓取HTTP请求
tcpdump -i eth0 -A port 80

# 抓取DNS查询
tcpdump -i eth0 port 53
```

#### 17.3 Wireshark过滤语法

```
Protocol过滤：tcp、udp、http、dns
IP过滤：ip.addr == 1.2.3.4
端口过滤：tcp.port == 80
HTTP过滤：http.request.method == "GET"
组合：ip.src == 1.2.3.4 and tcp.port == 80
```

#### 17.4 回答模板

> 排查网络问题用tcpdump命令抓包，-i指定网卡、port指定端口、-A显示内容。也用Wireshark图形化工具过滤分析。常用过滤条件按IP、端口、协议、请求方法组合。生产问题可能需用tcpdump dump到文件用Wireshark慢慢分析。

---

### 18. 负载均衡？

#### 18.1 LB类型

```
四层LB：基于IP+Port
- LVS、NAT、HAProxy (TCP)
- 只转发

七层LB：基于应用层
- Nginx、HAProxy (HTTP)
- 能根据URL/Header转发

DNS轮询：基于域名返回不同IP
```

#### 18.2 转发算法

```
轮询：Round Robin
加权轮询：Weighted Round Robin
最少连接：Least Connections
IP哈希：Source IP Hash
```

#### 18.3 高可用

```
HAProxy/Keepalived：
- VRRP协议
- 双主热备
- 自动故障转移
```

#### 18.4 回答模板

> 负载均衡分四层和七层，四层用LVS/F5，七层用Nginx/HAProxy。七层能根据URL/Cookie转发支持灰度发布。四层转发效率高但不解析内容。常用转发算法有轮询、加权、最少连接、Ip_hash。高可用需要主备如Keepalived+VRRP。

---

### 19. CDN？

#### 19.1 CDN作用

```
CDN：
- 就近接入提升速度
- 减轻源站压力
- 抗DDoS
- HTTPS加速
```

#### 19.2 架构

```
用户 → 最近CDN节点 → 缓存？返回: 追溯源站:返回+缓存
```

#### 19.3 缓存策略

```
Cache-Control：
- max-age：缓存秒数
- s-maxage：代理缓存秒数
- no-cache：需验证
- no-store：不缓存

Cache-Control: max-age=3600, s-maxage=86400
```

#### 19.4 回答模板

> CDN内容分发网络让用户访问最近边缘节点，提升速度减小源站压力。回源命中机制根据CacheControl，静态资源CDN缓存几个月，动态API不缓存只用做加速。设置HTTP Header的Cache-Control控制缓存时间。

---

### 20. HTTPS握手过程？

#### 20.1 TLS1.2握手

```
1. Client Hello
- 支持的加密套件
- Random Bytes

2. Server Hello
- 选择的加密套件
- Random Bytes
- 证书

3. 验证证书（客户端）
- 有效期、CA签名、域名匹配

4. Client Key Exchange
- Pre-Master Secret（用公钥加密）

5. Finished
- 切换加密通道
```

#### 20.2 TLS1.3简化

```
1. Client Hello → 支持的加密套件
2. Server Hello → 证书+公钥+参数
3. Finished → 切换加密通道

1-RTT完成，之前需要2-RTT！
```

#### 20.3 加密套件

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
├─密钥交换：ECDHE
├─身份验证：RSA
├─对称加密：AES 256-bit GCM
└─消息认证：SHA384
```

#### 20.4 回答模板

> HTTPS用TLS协议握手。1.2版流程：ClientHello建议、ServerHello选套件+返回证书、验证通过后Client发PreMasterSecret用公钥加密、双方生成会话密钥、切换加密。1.3简化成1RTT更快。密钥交换推荐ECDHE实现前向保密，用RSA签名认证，AES-GCM对称加密。

---

## 附 录：面试追问

1. **GET POST区别？**
   - GET幂等、参数URL长度限制、会缓存；POST不幂等、安全、参数Body
2. **为什么80/443端口？**
   - HTTP默认80、HTTPS默认443是IANA注册端口
3. **MTU和MSS？**
   - MTU最大传输单元（1500）、MSS最大分段
4. **SYN Flood攻击？**
   - 攻击者发SYN不完成握手耗尽TCB表，可用SYN Cookie防御
5. **Nagle算法？**
   - 合并小包减少网络负担，与Nagle算法用delay

---

## 参考资料

- 《计算机网络：自顶向下方法》
- RFC相关协议文档

---

> 整理by Claude Code | 计算机网络面试高频100问