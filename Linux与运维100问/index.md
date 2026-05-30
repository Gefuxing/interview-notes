# Linux与运维100问——Linux操作系统核心深度指南

> 本文档面向Linux运维学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 Linux基础（高频 ★★★★★）

### 1. 什么是Linux？

#### 1.1 定义

> Linux是一个开源的类Unix操作系统内核，基于POSIX标准，广泛应用于服务器、嵌入式和超级计算机。

```plaintext
Linux发行版：
- RHEL/CentOS/Fedora   企业级
- Debian/Ubuntu      桌面/服务器
- SUSE/openSUSE     企业级
- Alpine            轻量容器
```

#### 1.2 回答模板

> Linux是开源操作系统内核，发行版有Ubuntu、CentOS、Debian等。服务器市场占主导，嵌入式和云计算基础。

---

### 2. Linux文件系统层级标准（FHS）？

#### 2.1 目录结构

```plaintext
FHS结构：
/bin      系统命令
/etc     配置文件
/home    用户目录
/root    root用户目录
/usr     只读程序数据
/var     可变数据（日志）
/tmp     临时文件
/opt     可选软件
/dev     设备文件
/proc    进程信息
/sys     系统信息
```

#### 2.2 回答模板

> FHS定义了Linux目录规范：/etc配置、/usr程序、/var日志、/dev设备、/proc内核信息。符合规范便于管理和排查。

---

### 3. Linux文件权限？

#### 3.1 权限模型

```bash
# 查看权限
ls -la file

-rwxr-xr-x 1 user group 1234 Jan 10 10:00 file
   ├──┬─┬─┬─  ├──┬─
   │  │ │ │  │  └─ others
   │  │ │ │  └─ group
   │  │ │ └─ owner
   │  │ └─ sticky bit/suid/sgid
   └─ file type (-/d/l/b/c)
```

```bash
# 权限数字
chmod 755 file    # rwxr-xr-x
chmod 644 file   # rw-r--r--
chmod +x file   # 加执行

# ACL权限
setfacl -m u:bob:rw file
getfacl file
```

#### 3.2 特殊权限

```bash
# SUID（4）
chmod 4755 file    # -rwsr-xr-x
# 运行文件时以owner身份

# SGID（2）
chmod 2755 dir    # rwxrwsr-x
# 文件以group身份，目录内新建文件继承group

# Sticky（1）
chmod 1777 /tmp  # rwxrwxrwt
# 只能删除自己的文件
```

#### 3.3 回答模板

> 文件权限：owner、group、others三层，rwx三位。SUID运行提权、SGID继承组、Sticky防误删。

---

### 4. Linux进程管理？

#### 4.1 查看进程

```bash
# 静态查看
ps aux               # 所有进程
ps -ef               # 父子关系
ps -ely              # 长格式

# 动态监控
top                  # 实时监控
htop                 # 交互增强
atop                 # 历史+资源

# 按条件筛选
pgrep -f pattern     # 匹配名称
pkill pattern       # 杀掉匹配
```

#### 4.2 进程状态

```
进程状态：
Rrunning    运行中
Ssleeping   可中断睡眠
Dsleeping   不可中断睡眠（IO）
Tstopped   已停止
Z defunct  僵尸
```

#### 4.3 进程控制

```bash
# 信号机制
kill -SIGTERM PID  # 优雅终止（默认）
kill -SIGKILL PID  # 强制终止
kill -SIGSTOP PID  # 暂停
kill -SIGHUP PID   # 重载配置
kill -SIGINT PID   # 中断（Ctrl+C）

# 进程后台
ctrl+z             # 暂停放入后台
bg                # 后台继续
fg                # 前台继续
nohup cmd &       # 防断线后台
```

#### 4.4 回答模板

> Linux进程管理：ps静态、top动态、信号控制。TERM优雅、KILL强制、HUP重载配置。

---

### 5. Linux内存管理？

#### 5.1 内存查看

```bash
# 内存使用
free -h             # 人性化显示
free -m             # MB���单位
cat /proc/meminfo    # 详细

# vmstat
vmstat 1            # 每秒
vmstat 1 5          # 5次

# /proc/meminfo
MemTotal        总内存
MemFree        空闲
MemAvailable   可用（含缓存）
Buffers       缓冲区
Cached        页缓存
SwapTotal     交换分区
SwapFree      交换分区空闲
```

#### 5.2 回答模板

> free查看内存，MemAvailable才是真实可用。内存满会swap，影響性能。

---

### 6. Linux磁盘管理？

#### 6.1 分区管理

```bash
# 查看分区
fdisk -l                    # 磁盘分区
partprobe                   # 刷新分区表
lsblk                       # 块设备树
df -h                       # 文件系统使用

# 分区工具
fdisk /dev/sdb              # MBR分区
gdisk /dev/sdb              # GPT分区
parted /dev/sdb             # 交互分区
```

#### 6.2 文件系统

```bash
# 创建文件系统
mkfs.ext4 /dev/sdb1
mkfs.xfs /dev/sdb1
mkfs -t ext4 /dev/sdb1

# 挂载
mount /dev/sdb1 /mnt/data
umount /mnt/data

# 检查修复
fsck /dev/sdb1
e2fsck -fy /dev/sdb1
```

#### 6.3 回答模板

> 磁盘管理：fdisk分区、mkfs创文件系统、mount挂载。生产环境umount前需确认无进程使用。

---

### 7. Linux网络配置？

#### 7.1 网络命令

```bash
# 配置查看
ip addr                  # IP地址
ip link                  # 网卡状态
ip route                 # 路由表
ss -tlnp                # 监听端口
netstat -tlnp           # 同上（旧）

# 连通测试
ping -c 4 8.8.8.8
traceroute 8.8.8.8
mtr 8.8.8.8              # ping+traceroute

# DNS
dig domain.com
nslookup domain.com
host domain.com
```

#### 7.2 网络配置

```bash
# ip命令
ip addr add 192.168.1.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.1.1

# ifcfg（centos6）
cat /etc/sysconfig/network-scripts/ifcfg-eth0

# nmcli（centos7+）
nmcli con add con-name eth0 ifname eth0 ip4 192.168.1.10/24 gw4 192.168.1.1
```

#### 7.3 回答模板

> 网络工具：ip配置、ss查端口、ping测连通。网络配置文件CentOS7+/etc/sysconfig/network-scripts/。

---

### 8. Linux用户管理？

#### 8.1 用户命令

```bash
# 用户操作
useradd -m -s /bin/bash user1
usermod -aG group user1
userdel -r user1

# 组操作
groupadd devops
groupadd -g 1000 devuser
gpasswd -a user1 devops

# sudo权限
visudo                  # 编辑sudoers
# user1 ALL=(ALL) NOPASSWD: /bin/ls, /bin/grep
```

#### 8.2 回答模板

> 用户管理：useradd创建、usermod修改、userdel删除。sudo权限visudo配置，生产慎用root。

---

### 9. Linux软件包管理？

#### 9.1 包管理器

```bash
# Debian系（apt）
apt update
apt install nginx
apt remove --purge nginx
apt upgrade
dpkg -i xxx.deb

# RedHat系（yum/dnf）
 yum install nginx
yum remove nginx
yum update
rpm -ivh xxx.rpm

# 其他
# Alpine: apk add nginx
# Arch: pacman -S nginx
```

#### 9.2 回答模板

> 包管理：CentOS用yum/dnf，Debian用apt。生产优先yum/apt，确保安全更新。

---

### 10. Linux日志管理？

#### 10.1 日志系统

```bash
# rsyslog配置
cat /etc/rsyslog.conf

# 查看日志
/var/log/messages     # 系统日志（centos）
/var/log/syslog       # 系统日志（debian）
/var/log/auth.log    # 认证日志
/var/log/nginx/access.log  # 应用日志
/var/log/dmesg       # 启动日志

# journalctl
journalctl -xe                    # 错误详情
journalctl -u nginx.service        # 特定服务
journalctl --since "1 hour ago"   # 时间筛选
journalctl -f                     # 实时
```

#### 10.2 回答模板

> Linux日志：rsyslog收集，journalctl（CENTOS7+）查询。/var/log/目录，系统日志messages/auth.log。

---

### 11. Linux性能分析vmstat？

#### 11.1 输出解读

```bash
vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs  us  sy  id  wa st
 1  0      0 512000  10240 204800    0    0     5    10   25   50  5   2  90  3  0

# r: 运行队列
# b: 阻塞进程
# si/so: swap in/out（严重时有问题）
# us: 用户CPU
# id: 空闲CPU
```

#### 11.2 回答模板

> vmstat：si/so非0说明swap频繁需加内存。r过CPU核数说明CPU不足。id低说明CPU忙。

---

### 12. Linux性能分析iostat？

#### 12.1 输出解读

```bash
iostat -xz 1
Linux 3.10.0 (hostname)     _x86_64_        01/10/2024     _x86_64_

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.23    0.00    2.45    3.12    0.00   89.20

Device:         rrqm/s   wrqm/s     r/s     w/s   rMB/s   wMB/s avgrq-sz avgqu-sz   await  svctm  %util
sda              0.00     0.00    1.00    0.00    0.01    0.00     8.00     0.00    8.00   8.00   0.80

# await: IO等待（ms）
# svctm: 服务时间
# %util: 使用率
```

#### 12.2 回答模板

> iostat：await高说明IO慢。%util接近100%说明磁盘瓶颈。ssd无此问题。

---

## 第二章 Shell脚本编程（高频 ★★★★★）

### 13. 基础变量使用？

#### 13.1 变量定义

```bash
# 定义变量
name="张三"
age=25

# 使用变量
echo $name
echo ${name}

# 只读变量
readonly PI=3.14

# 删除变量
unset name

# 字符串
str='原样输出'   # 单引号
str2="值: $name" # 双引号展开
```

#### 13.2 回答模板

> Shell变量：name=value定义，${name}使用，单引号原样、双引号展开变量。

---

### 14. 特殊变量？

#### 14.1 参数变量

```bash
$0         # 脚本名
$1-$9      # 第1-9个参数
${10}       # 第10+
$@         # 所有参数
$*         # 所有参数（整体）
$#         # 参数个数
$?         # 上个命令退出码
$$         # 当前进程PID
$!         # 上个子进程PID
```

#### 14.2 回答模板

> Shell特殊变量：$0脚本名，$1-$9参数，$?退出码，$@所有参数。

---

### 15. 条件判断？

#### 15.1 test命令

```bash
# 文件
[ -f file ]     # 普通文件
[ -d dir ]      # 目录
[ -r file ]    # 可读

# 字符串
[ -z str ]     # 空
[ -n str ]     # 非空
[ "$a" = "$b" ]

# 数字
[ $a -eq $b ]  # equal
[ $a -ne $b ]  # not equal
[ $a -gt $b ]  # greater
[ $a -lt $b ]  # less

# 逻辑
[ -f file ] && echo exists
[ -f file ] || echo not exists
```

#### 15.2 回答模板

> Shell条件：[]或[[]]，-f文件、-d目录、-eq相等、-gt大于。&&||逻辑组合。

---

### 16. 循环语句？

#### 16.1 for循环

```bash
# 列表循环
for i in {1..5}; do
    echo $i
done

# 命令结果
for f in $(ls *.log); do
    echo $f
done

# C风格
for ((i=0;i<10;i++)); do
    echo $i
done
```

#### 16.2 while循环

```bash
while read line; do
    echo "$line"
done < file.txt

count=0
while [ $count -lt 5 ]; do
    echo $count
    count=$((count+1))
done
```

#### 16.3 回答模板

> Shell循环：for in遍历、while条件、C风格for。双括号(( ))做算术运算。

---

### 17. 函数定义？

#### 17.1 函数语法

```bash
# 定义函数
function hello() {
    echo "Hello $1"
    return 0
}

# 调用
hello World

# 返回值
result=$(hello World)
```

#### 17.2 回答模板

> Shell函数：function/name()定义，$1参数，$?捕获返回值。全局变量慎用。

---

### 18. 文本处理三剑客？

#### 18.1 grep

```bash
# 查找
grep "error" log.txt
grep -r "error" /var/log/
grep -i "error" log.txt          # 忽略大小写
grep -n "error" log.txt          # 行号
grep -c "error" log.txt          # 计数
grep -v "error" log.txt         # 不包含
grep -E "[0-9]+" log.txt        # 正则
```

#### 18.2 sed

```bash
# 替换
sed 's/error/warning/g' file
sed -i 's/error/warning/g' file # 原地修改
sed -n '5p' file               # 打印第5行
sed '1,5d' file                # 删除1-5行
sed '/error/d' file            # 删除包含行
```

#### 18.3 awk

```bash
# 打印列
awk '{print $1}' file
awk -F',' '{print $1}' csv     # 指定分隔符
awk 'NR>1' file                # 跳过表头

# 条件
awk '$3>80 {print $1,$3}' file

# 内置变量
# NR 行号 NF 列数 FS 分隔符
```

#### 18.4 回答模板

> 文本处理：grep查找sed替换awk分析。正则+列处理是文本处理核心。

---

### 19. 数组使用？

#### 19.1 数组操作

```bash
# 定义
arr=(a b c d)
arr[0]=a
arr[1]=b

# 访问
echo ${arr[0]}
echo ${arr[@]}    # 所有元素
echo ${#arr[@]}  # 长度

# 切片
echo ${arr[@]:1:2}
```

#### 19.2 回答模板

> Shell数组：arr=(a b c)定义，${arr[@]}访问，${#arr[@]}长度。双括号做比较。

---

### 20. 输入输出？

#### 20.1 read输入

```bash
echo -n "Input: "
read name
echo "Hello $name"

# 同时读取多个
read -p "Name: " name age
```

#### 20.2 重定向

```bash
cmd > file      # 覆盖
cmd >> file     # 追加
cmd 2> file     # 错误
cmd &> file     # 所有
cmd < file      # 输入

# 管道
cmd1 | cmd2
```

#### 20.3 回答模板

> Shell IO：read读输入，>覆盖>>追加，2>错误，|管道。/dev/null丢弃输出。

---

### 21. 脚本调试？

#### 21.1 调试选项

```bash
#!/bin/bash -x           # 跟踪执行
bash -x script.sh

# 选项说明：
# -x 输出每行
# -v 原始输入
# -e 遇错退出
# -u 未定义变量报错
```

#### 21.2 回答模板

> 脚本调试：bash -x跟踪，echo debug。set -e遇错退出，set -u未定义报错。

---

## 第三章 系统服务（高频 ★★★★★）

### 22. systemd服务管理？

#### 22.1 基础命令

```bash
# 服务管理
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl status nginx

# 开机启动
systemctl enable nginx
systemctl disable nginx

# 查看
systemctl list-units --type=service
systemctl list-unit-files
```

#### 22.2 服务文件

```ini
[Unit]
Description=Nginx HTTP Server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target
```

#### 22.3 回答模板

> systemd：init替代品，unit文件定义服务。start/stop/enable/disable常用。Type=forking后台服务。

---

### 23. systemd高级功能？

#### 23.1 资源限制

```ini
[Service]
MemoryMax=1G
CPUQuota=50%
TasksMax=100
IOWeight=100
```

#### 23.2 日志

```bash
journalctl -u nginx.service
journalctl -f -u nginx.service
journalctl --since "1 hour ago" -u nginx.service
```

#### 23.3 回答模板

> systemd资源：MemoryMax/CPUQuota限制。journalctl查日志，-u指定服务。

---

### 24. cron定时任务？

#### 24.1 crontab格式

```bash
# 格式
分 时 日 月 周 命令

# 示例
0 2 * * * /backup.sh        # 每天2点
0 */4 * * * /check.sh       # 每4小时
0 9-18 * * * /task.sh      # 9-18点每小时
0 0 1 * * /monthly.sh     # 每月1号0点
```

#### 24.2 服务管理

```bash
# 编辑
crontab -e                 # 当前用户
crontab -u user -e         # 指定用户

# 查看
crontab -l
crontab -l -u user

# 日志
grep CRON /var/log/syslog
```

#### 24.3 回答模板

> cron定时任务：分时日月周crontab -e编辑。日志/var/log/syslog或messages。

---

### 25. 日志rotate？

#### 25.1 logrotate配置

```bash
# 主配置
/etc/logrotate.conf

# 应用配置
/etc/logrotate.d/nginx

/var/log/nginx/*.log {
    daily              # 每天轮转
    missingok         # 缺失不报错
    rotate 14        # 保留14份
    compress         # gzip压缩
    delaycompress   # 延迟压缩上一份
    notifempty       # 空文件不轮转
    create 0640 www-data adm
    sharedscripts   # postrotate只执行一次
    postrotate
        kill -USR1 `cat /run/nginx.pid`
    endscript
}
```

#### 25.2 回答模板

> logrotate自动轮转日志。daily/weekly/monthly指定频率，rotate保留份数，compress压缩。

---

### 26. SSH远程管理？

#### 26.1 SSH命令

```bash
# 连接
ssh user@host
ssh -p 2222 user@host
ssh -i ~/.ssh/key.pem user@host

# 密钥生成
ssh-keygen -t rsa -b 4096
ssh-copy-id user@host

# 配置
~/.ssh/config
Host ali
    HostName 192.168.1.10
    Port 22
    User root
    IdentityFile ~/.ssh/key.pem
```

#### 26.2 SSH安全加固

```plaintext
SSH安全：
- 禁用密码登录（PasswordAuthentication no）
- 禁用root登录（PermitRootLogin no）
- 使用密钥
- 更改默认端口
- 限制来源IP（AllowUsers/AllowGroups）
```

#### 26.3 回答模板

> SSH：ssh-keygen生成密钥，ssh-copy-id复制公钥。生产禁用密码、更换端口。

---

### 27. FTP/NFS/Samba？

#### 27.1 NFS挂载

```bash
# 服务端
/etc/exports:
/share 192.168.1.0/24(rw,sync,no_root_squash)

exportfs -ra

# 客户端
mount -t nfs 192.168.1.10:/share /mnt/nfs
# 开机挂载
# /etc/fstab: 192.168.1.10:/share /mnt/nfs nfs defaults,_netdev 0 0
```

#### 27.2 Samba配置

```ini
[global]
workgroup = WORKGROUP
security = user

[share]
path = /samba
valid users = user
writable = yes
```

#### 27.3 回答模板

> NFS：/etc/exports配置服务端，mount挂载。Samba CIFS共享，Windows互访。

---

### 28. 防火墙firewalld？

#### 28.1 基础使用

```bash
# firewall-cmd
firewall-cmd --list-all
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# zone
firewall-cmd --permanent --zone=public --add-port=80/tcp
```

#### 28.2 回答模板

> firewalld：CentOS7防火墙，--permanent永久规则，--reload重载。常用service/http快速放行。

---

### 29. 防火墙iptables？

#### 29.1 基础规则

```bash
# 查看
iptables -L -n -v
iptables -L -n -v --line-numbers

# 基本规则
iptables -A INPUT -j ACCEPT          # 追加
iptables -I INPUT 1 -j INSERT       # 插入
-D DELETE  -D 编号
-P POLICY                         # 默认策略

# 常用
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptABLES -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

#### 29.2 NAT

```bash
# 源NAT
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE

# 目的NAT
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:8080
```

#### 29.3 回答模板

> iptables：四表五链四链。-A追加，-I插入，-D删除，-p协议，--dport端口。

---

### 30. DNS服务bind？

#### 30.1named.conf

```bash
# /etc/named.conf
options {
    listen-on port 53 { any; };
    allow-query { any; };
    recursion yes;
};

zone "example.com" IN {
    type master;
    file "example.com.zone";
};
```

#### 30.2 区域文件

```bash
# /var/named/example.com.zone
$TTL 600
@  IN  SOA dns.example.com. admin.example.com. (
        2024010101 ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        600 )
@  IN  NS  dns.example.com.
dns  IN  A   192.168.1.10
www IN  A   192.168.1.10
```

#### 30.3 回答模板

> BIND：DNS服务。named.conf主配置，正向解析zone文件。Serial序列号每次更新要+1。

---

### 31. DHCP服务？

#### 31.1 dhcpd.conf

```dhcp
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8;
}
```

#### 31.2 回答模板

> DHCP动态分配IP。range范围，router网关，dns-nameserver DNS服务器。

---

### 32. PXE网络装机？

#### 32.1 PXE流程

```plaintext
PXE引导：
1. BIOS网卡PXE发起DHCP请求
2. DHCP返回IP+TFTP服务器+pxelinux.0
3. Client从TFTP下载引导文件
4. 引导文件加载内核initrd
5. 启动安装程序
```

#### 32.2 回答模板

> PXE网络装机：DHCP/TFTP/HTTP/WDS配合。BIOS发起PXE请求引导。

---

### 33. LDAP目录服务？

#### 33.1 OpenLDAP

```bash
# 安装
yum install openldap openldap-clients

# 配置
/etc/openldap/slapd.conf

# 启动
systemctl start slapd
```

#### 33.2 回答模板

> LDAP集中认证。用户/组管理，统一登录。OpenLDAP开���实现。

---

### 34. VPN服务？

#### 34.1 OpenVPN

```bash
# 配置文件
/etc/openvpn/server.conf

port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
```

#### 34.2 IPSec

```bash
# strongSwan
# /etc/ipsec.conf
config setup
    charondebug="all"

conn %default
    authby=secret
    auto=route

conn vpn
    left=192.168.1.10
    right=192.168.1.20
    auto=add
```

#### 34.3 回答模板

> VPN：OpenVPN兼容强，IPSec效率高。远程访问选OpenVPN，站间选IPSec。

---

### 35. LAMP/LNMP环境？

#### 35.1 LAMP

```bash
# CentOS安装
yum install httpd mariadb-server php php-mysql

# 启服务
systemctl start httpd mariadb
systemctl enable httpd mariadb
```

#### 35.2 LNMP

```bash
# 一键或手动
yum install nginx php-fpm mariadb-server
# nginx配置fastcgi_pass
# php-fpm配置pool
```

#### 35.3 回答模板

> LAMP/LNMP：Web服务+Mysql+PHP。nginx+php-fpm性能更好。生产推荐LNMP。

---

### 36. Nginx反向代理？

#### 36.1 配置

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080 backup;
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_next_upstream error timeout invalid_header http_500 http_502;
    }
}
```

#### 36.2 负载均衡策略

```nginx
upstream backend {
    server 192.168.1.10 weight=3;
    server 192.168.1.11;
    ip_hash;
    fair;
}
```

#### 36.3 回答模板

> Nginx反向代理：upstream定义后端，proxy_pass转发。weight权重，ip_hash会话保持。

---

### 37. Tomcat配置？

#### 37.1 server.xml

```xml
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">
      <Host name="localhost" appBase="webapps">
      </Host>
    </Engine>
  </Service>
</Server>
```

#### 37.2 JVM参数

```bash
# catalina.sh
CATALINA_OPTS="-Xms512m -Xmx2048m -XX:+UseG1GC"
JAVA_OPTS="$JAVA_OPTS -server"
```

#### 37.3 回答模板

> Tomcat：server.xml配置server/connector/engine/host。JVMheap -Xms -Xmx，G1GC。

---

### 38. Apache配置？

#### 38.1 httpd.conf

```apache
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf

ServerAdmin admin@example.com
ServerName www.example.com:80

<Directory />
    AllowOverride None
</Directory>

<VirtualHost *:80>
    DocumentRoot "/var/www/html"
    ServerName www.example.com
</VirtualHost>
```

#### 38.2 回答模板

> Apache：httpd.conf主配置，conf.d/模块化。痛快点conf配置。MPM worker/static。

---

## 第四章 性能调优（高频 ★★★★★）

### 39. CPU高排查？

#### 39.1 排查命令

```bash
# 先定位进程
top                                 # 查看%CPU
ps -eo pcpu,pid,comm | sort -k1 -nr | head

# 定位线程
top -H -p PID                       # 查看线程CPU
perf top -p PID                     # 函数级分析
strace -c -p PID                    # 系统调用

# 定位代码
perf record -p PID -g
perf report
```

#### 39.2 常见原因

```plaintext
CPU高原因：
- 业务流量高
- GC频繁
- 死循环
- 计算密集
- DDOS攻击
```

#### 39.3 回答模板

> CPU高：top定位进程perf分析代码。Java用jstack/jmap排查GC问题。

---

### 40. 内存高排查？

#### 40.1 排查命令

```bash
# 查看内存
free -h
top -o %MEM
ps -eo pid,rss,comm | sort -k2 -nr | head

# 详细分析
pmap -x PID           # 内存映射
cat /proc/PID/status  # 进程状态

# 泄漏检测
valgrind --leak-check=full program
```

#### 40.2 常见原因

```plaintext
内存高原因：
- JVM heap过大
- 缓存无限增长
- 内存泄漏
- 连接池配置过大
```

#### 40.3 回答模板

> 内存高：RSS/VSZ区分用户/虚拟内存。jmap -heap看堆，MAT分析泄漏。

---

### 41. IO高排查？

#### 41.1 排查命令

```bash
# IO使用
iostat -xz 1
iotop                          # 按进程
pidstat -d 1                   # 进程IO

# 文件级排查
lsof +L1                      # 打开文件数
find /proc -name fd -type d -maxdepth 2 | xargs ls -l
```

#### 41.2 常见原因

```plaintext
IO高原因：
- 日志量大
- 大文件读写
- SWAP使用
- IO密集操作
```

#### 41.3 回答模板

> IO高：iostat看%util和await。ionice限制IO，ssd代替机械盘解决。

---

### 42. 网络高排查？

#### 42.1 排查命令

```bash
# 网络使用
iftop                         # 按连接
nethogs                       # 按进程
netstat -nat                  # 连接状态
ss -s                        # 统计

# 抓包
tcpdump -i eth0 host 192.168.1.10
tcpdump -i eth0 port 80
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'
```

#### 42.2 常见原因

```plaintext
网络高原因：
- DDOS攻击
- 日志推送
- 文件传输
- 爬虫
```

#### 42.3 回答模板

> 网络高：iftop看带宽，netstat看连接状态。tcpdump抓包分析，防火墙拦截。

---

### 43. 磁盘满排查？

#### 43.1 排查命令

```bash
# 查看使用
df -h
du -sh /*
du -sh /var/* 2>/dev/null | sort -hr | head

# 大文件
find / -type f -size +100M -exec ls -lh {} \;
```

#### 43.2 清理策略

```plaintext
清理空间：
- 日志轮转(logrotate)
- 临时文件(/tmp)
- 旧内核(yum autoremove)
- docker prune
- 缓存清理
```

#### 43.3 回答模板

> 磁盘满：du排序定位，find找大文件。日志轮转、定时清理防止满。

---

### 44. 负载高排查？

#### 44.1 load定义

```bash
# 系统负载
uptime
w

# top
top
load average: 0.5, 0.8, 1.0  # 1/5/15分钟
```

```plaintext
负载含义：
- CPU核数*N=单核满载
- 4核cpu负载4=100%
- 负载>CPU核数说明等待
```

#### 44.2 排查

```bash
# top找高load进程
# vmstat 1找r,b列
# 对症下药
```

#### 44.3 回答模板

> 负载>CPU核数说明有进程排队。load平均值1/5/15分钟，关注5分钟。

---

### 45. 进程异常排查？

#### 45.1 僵尸进程

```bash
# 查找僵尸
ps aux | grep 'Z'
ps -eo pid,ppid,state,comm

# 原因
# 父进程未wait子进程
# 父进程已死
```

#### 45.2 答模板

> 僵尸进程：父进程未回收。找出父进程kill或重起。长期存在有bug。

---

### 46. 链接数高排查？

#### 46.1 链接问题

```bash
# 查看链接
ss -s
netstat -an | wc -l
netstat -an | grep EST | wc -l

# TIME_WAIT
netstat -an | grep TIME_WAIT | wc -l
```

#### 46.2 处理

```bash
# TIME_WAIT重用
sysctl -w net.ipv4.tcp_tw_reuse=1
sysctl -w net.ipv4.tcp_fin_timeout=15

# 最大链接
sysctl -w net.max_conn
```

#### 46.3 回答模板

> TIME_WAIT：短链接正常，过多优化tcp参数。并发链接看ulimit -n。

---

### 47. tcpdump抓包？

#### 47.1 基础用法

```bash
# 抓包
tcpdump -i eth0
tcpdump -i eth0 host 192.168.1.10
tcpdump -i eth0 port 80
tcpdump -i eth0 tcp

# 保存
tcpdump -i eth0 -w /tmp/capture.cap

# 读取
tcpdump -r /tmp/capture.cap
tcpdump -r /tmp/capture.cap -X  # hex输出
```

#### 47.2 表达式

```bash
# 过滤
tcpdump 'host 192.168.1.10 and port 80'
tcpdump 'tcp[tcpflags] & tcp-syn != 0'
tcpdump 'tcp[tcpflags] & tcp-ack != 0'
```

#### 47.3 回答模板

> tcpdump：-i指定网卡，-w保存 -r读取，过滤host/port/tcp。分析网络问题必备。

---

### 48. strace跟踪？

#### 48.1 基础用法

```bash
# 跟踪
strace -p PID
strace -f -p PID             # 跟踪子进程
strace -c -p PID            # 统计

# 输出
strace -o output.txt -p PID
strace -t -p PID            # 时间前缀
strace -tt -p PID           # 微秒时间
```

#### 48.2 过滤

```bash
# 系统调用
strace -e trace=open,read,write -p PID
strace -e abbrev -p PID      # 简化输出
```

#### 48.3 回答模板

> strace：追踪系统调用，-c统计，-f追踪子进程。定位系统调用问题。

---

### 49. perf性能分析？

#### 49.1 基础用法

```bash
# 性能热点
perf top
perf top -p PID

# 记录分析
perf record -g
perf report

# CPU采样
perf sched latency
perf sched record -- sleep 10
```

#### 49.2 回答模板

> perf：Linux性能分析工具。top看热点，record记录分析。CPU/内存/IO全能。

---

### 50. htop/glances/monitor？

#### 50.1 htop

```bash
htop
# 彩色UI
# 鼠标操作
# 按CPU%、MEM%、TIME排序
```

#### 50.2 glances

```bash
glances
# 跨机器
glances -s               # server
glances -c client        # client
```

#### 50.3 回答模板

> 监控工具：htop交互友好、glances全能、monitor开源各有优势。

---

## 第五章 运维自动化（高频 ★★★★★）

### 51. Ansible基础？

#### 51.1 ad-hoc

```bash
# 简单命令
ansible all -m ping
ansible all -m shell -a "uptime"
ansible all -m copy -a "src=a dest=b"

# 批量执行
ansible prod -m yum -a "name=nginx state=present"
ansible prod -m systemd -a "name=nginx state=started enabled=yes"
```

#### 51.2 playbook

```yaml
- hosts: webservers
  become: yes
  tasks:
  - name: Install nginx
    yum:
      name: nginx
      state: present
  - name: Start nginx
    systemd:
      name: nginx
      state: started
      enabled: yes
```

#### 51.3 回答模板

> Ansible：幂等性批量管理。ad-hoc单命令，playbook剧本。有agent基于SSH。

---

### 52. Ansible模块？

#### 52.1 文件操作

```yaml
# copy
- copy:
    src: files/app.conf
    dest: /etc/app.conf
    mode: '0644'
    owner: app
    group: app

# template
- template:
    src: app.conf.j2
    dest: /etc/app.conf

# synchronize
- synchronize:
    src: files/
    dest: /opt/
```

#### 52.2 服务操作

```yaml
# systemd
- systemd:
    name: nginx
    state: restarted
    enabled: yes
    daemon_reload: yes

# service (老)
- service:
    name: nginx
    state: restarted
    enabled: yes
```

#### 52.3 回答模板

> Ansible模块：yum/package安装，copy/template文件，systemd/service服务，shell/command执行。

---

### 53. Ansible变量和 facts？

```yaml
# 变量
vars:
  nginx_version: "1.24"

# facts
- debug:
    msg: "{{ ansible_facts['distribution'] }} {{ ansible_facts['ansible_default_ipv4']['address'] }}"
```

```bash
# facts查看
ansible all -m setup
ansible all -m setup -a "filter=ansible_*"
```

#### 53.2 回答模板

> Facts：ansible_setup收集的系统信息。变量优先级11+层。注册变量register。

---

### 54. SaltStack基础？

#### 54.1 命令

```bash
salt '*' test.ping
salt '*' cmd.run 'uptime'
salt 'web*' pkg.install nginx
salt '*' state.apply nginx
```

#### 54.2 SLS

```yaml
# /srv/salt/nginx/init.sls
nginx:
  pkg.installed:
    - name: nginx
  service.running:
    - name: nginx
    - enable: True
```

#### 54.3 回答模板

> SaltStack：master-minion架构。salt命令行，SLS声明式。 grains/pillar传递信息。

---

### 55. Puppet基础？

#### 55.1 Manifest

```puppet
# site.pp
node 'web01.example.com' {
  class { 'nginx': }
}
```

#### 55.2 模块

```puppet
# modules/nginx/manifests/init.pp
class nginx {
  package { 'nginx':
    ensure => installed,
  }
  service { 'nginx':
    ensure => running,
    enable => true,
  }
}
```

#### 55.3 回答模板

> Puppet：DSL描述资源，catalog编译后执行。声明式，幂等性强。

---

### 56. Zabbix监控？

#### 56.1 架构

```plaintext
Zabbix：
- Zabbix Server   监控中心
- Zabbix Agent    被监控端
- Zabbix Proxy   代理
- Web Interface  Web界面
```

#### 56.2 常用key

```bash
# agent
system.cpu.load[all,avg1]
vfs.fs.size[/,used]
net.tcp.port[,80]
proc.num[nginx]
```

#### 56.3 回答模板

> Zabbix：agent收集server聚合告警。自定义key灵活，模板批量。

---

### 57. Prometheus监控？

#### 57.1 架构

```plaintext
Prometheus：
- Prometheus Server  拉取存储
- Exporters          指标导出
- Alertmanager      告警管理
- Grafana           可视化
- Pushgateway       推送
```

#### 57.2 指标类型

```plaintext
指标类型：
- Counter        累加器
- Gauge          仪表
- Histogram      直方图
- Summary       摘要
```

#### 57.3 回答模板

> Prometheus：拉模式，TSDB存储。Exporter暴露/metrics，PromQL查询，Grafana展示。

---

### 58. ELK日志收集？

#### 58.1 架构

```plaintext
ELK：
- Filebeat    日志收集
- Logstash   处理转换
- Elasticsearch 存储搜索
- Kibana     可视化
- Beats     家族
```

#### 58.2 配置

```yaml
# filebeat.yml
filebeat.inputs:
- type: log
  paths:
    - /var/log/*.log
output.logstash:
  hosts: ["localhost:5044"]
```

#### 58.3 回答模板

> ELK：Beats收集→Logstash处理→ES存储→Kibana展示。日志分析标配。

---

### 59. Grafana配置？

#### 59.1 数据源

```bash
# Add Data Source
# Prometheus http://localhost:9090
# ES http://localhost:9200
```

#### 59.2 Dashboard

```json
{
  "panels": [
    {
      "targets": [
        {
          "expr": "rate(http_requests_total[5m])"
        }
      ]
    }
  ]
}
```

#### 59.3 回答模板

> Grafana：多数据源。Dashboard模板。变量/报警/权限管理。

---

### 60. Jenkins CI/CD？

#### 60.1 Job类型

```bash
# Freestyle Job
# Pipeline Job
# Multibranch Pipeline
```

#### 60.2 Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make'
            }
        }
        stage('Test') {
            steps {
                sh 'make test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

#### 60.3 回答模板

> Jenkins：Pipeline代码化。Blue Ocean UI。Build成功发通知部署。SLAVE并行。

---

### 61. GitLab CI？

#### 61.1 .gitlab-ci.yml

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - make build

test:
  stage: test
  script:
    - make test
  only:
    - main

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  environment:
    name: production
```

#### 61.2 Runner

```bash
# 注册runner
gitlab-runner register
# executor: shell/docker
```

#### 61.3 回答模板

> GitLab CI：.gitlab-ci.yml定义流水线。Runner执行。Auto DevOps一键启用。

---

### 62. Docker Swarm？

#### 62.1 集��

```bash
# 初始化
docker swarm init --advertise-addr 192.168.1.10

# 加入
docker swarm join-token worker
docker swarm join --token xxx MANAGER_IP:2377

# 节点
docker node ls
docker node promote node2
```

#### 62.2 Service

```bash
# 部署
docker service create --name web --replicas 3 -p 80:80 nginx

# 管理
docker service ls
docker service scale web=5
docker service update --image nginx:new web
docker service rollback web
```

#### 62.3 回答模板

> Docker Swarm：简化的容器编排swarm init。service管理。K8S功能更全。

---

### 63. K3s轻量K8S？

#### 63.1 安装

```bash
# 单机
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik" sh -

# 多机
curl -sfL https://get.k3s.io | K3S_URL=https://server:6443 K3S_TOKEN=xxx sh -
```

#### 63.2 使用

```bash
kubectl get nodes
kubectl get pods -A
```

#### 63.3 回答模板

> K3s：轻量K8s。边缘计算、研发环境。单节点 <512MB内存。

---

### 64. SRE实践？

#### 64.1 SRE原则

```plaintext
SRE：
- 确定SLO
- 错误预算
- 做可靠性工作
- 减少辛劳
- 拥抱风险
```

#### 64.2 指标

```plaintext
SLO/SLI：
- 请求延迟
- 错误率
- 可用性
- 吞吐量
```

#### 64.3 回答模板

> SRE：用SLO衡量可靠性，SLI测量达成度。错误预算允许创新。

---

### 65. 蓝绿发布？

```bash
# 流量切换
# lb切blue->green
# 或者DNS切换
# 两套环境同时运行
```

#### 65.2 回答模板

> 蓝绿：两套环境，秒级切换。无停机，回滚快。成本高。

---

### 66. 滚动发布？

```bash
# k8s deployment默认
# 逐步替换
# maxSurge/maxUnavailable控制
```

#### 66.2 回答模板

> 滚动：逐个替换。旧版可回滚。对用户体验平滑。

---

### 67. 金丝雀发布？

```bash
# 流量切小部分到新版本
# 观察
# 逐步放大
# 或者Istio控制
```

#### 67.2 回答模板

> 金丝雀：小流量先上，观察无问题放大。风险小。

---

### 68. GitOps实践？

```plaintext
GitOps：
- Git做唯一真相源
- ArgoCD/Tekton同步
- 自动rollback
- drift检测
```

#### 68.2 回答模板

> GitOps：配置文件Git管理，ArgoCD同步部署。版本追溯。

---

### 69. IaC基础设施即代码？

#### 69.1 Terraform

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxx"
  instance_type = "t3.micro"
  tags = {
    Name = "web"
  }
}
```

#### 69.2 Ansible

```yaml
- hosts: all
  become: yes
  tasks:
  - import_playbook: nginx.yml
```

#### 69.3 回答模板

> IaC：Terraform/Ansible/Vagrant。版本化、幂等、可复制。不可变基础设施。

---

### 70. 日志收集架构？

#### 70.1 EFK

```plaintext
EFK = Elasticsearch + Fluentd + Kibana

Fluentd = Beats + Logstash
```

#### 70.2 Loki

```bash
# Loki日志+Grafana
# 成本低
# 索引标签
```

#### 70.3 回答模板

> 日志收集：ELK/EFK/Loki。Fluentd/Filebeat收集。Loki+Prometheus监控。

---

## 第六章 安全与防护（高频 ★★★★★）

### 71. Linux安全加固？

#### 71.1 基线检查

```bash
# 用户安全
# /etc/passwd检查
# sudoers检查
# 空密码检查

# 服务安全
# 禁用不必要的服务
# 禁用telnet/rsh/ftp

# 网络安全
# iptables规则
# tcp_wrapper
```

#### 71.2 回答模板

> 安全加固：最小安装、禁用服务、权限收紧、口令策略、网络ACL。

---

### 72. 入侵检测？

#### 72.1 检测命令

```bash
# 进程
ps auxf
ls -la /proc/*/exe

# 网络
netstat -tulnp
ss -tulnp
lsof -i

# 用户
last
lastlog
who
```

#### 72.2 日志分析

```bash
# 登录日志
/var/log/secure
/var/log/messages
/var/log/auth.log
```

#### 72.3 回答模板

> 入侵检测：异常进程、网络连接、登录日志。rootkit查rkhunter。

---

### 73. 常用安全工具？

#### 73.1 扫描工具

```bash
rkhunter            # rootkit
lynis              # 安全审计
clamav             # 病毒
aide               # 完整性
tripwire           # 完整性
```

#### 73.2 回答模板

> 安全工具：rkhunter查rootkit，lynis审计，aide/tripwire文件防篡改。

---

### 74. SELinux/AppArmor？

#### 74.1 SELinux

```bash
# 状态
getenforce
sestatus

# 模式
setenforce Enforcing
# /etc/selinux/config: SELINUX=enforcing
```

#### 74.2 布尔值

```bash
# 查看
getsebool -a
# 开启
setsebool -P httpd_can_network_connect 1
```

#### 74.3 回答模板

> SELinux：强制访问控制。enforcing宽松/permissive警告。可用布尔值细粒度控制。

---

### 75. 加密对称/非对称？

#### 75.1 对称加密

```bash
# openssl enc
openssl enc -aes-256-cbc -in file -out file.enc
openssl enc -d -aes-256-cbc -in file.enc -out file
```

#### 75.2 非对称加密

```bash
# 生成密钥
openssl genrsa -out privkey.pem 2048
openssl rsa -in privkey.pem -pubout -out pubkey.pem

# 加解密
openssl rsautl -encrypt -pubin -in file -inkey pubkey.pem -out file.enc
openssl rsautl -decrypt -inkey privkey.pem -in file.enc -out file
```

#### 75.3 回答模板

> 对称加密AES速度快，非对称RSA安全用于密钥交换。OpenSSL实现。

---

### 76. 证书SSL/TLS？

#### 76.1 生成证书

```bash
# 自签CA
openssl req -x509 -newkey rsa:2048 -keyout ca.key -out ca.crt

# 服务器证书
openssl req -newkey rsa:2048 -keyout server.key -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```

#### 76.2 HTTPS配置

```nginx
server {
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_session_timeout 1d;
    ssl_protocols TLSv1.2 TLSv1.3;
}
```

#### 76.3 回答模板

> SSL证书：自签CA或Let's Encrypt。HTTPS=TLS。nginx/apache配置。

---

### 77. 防火墙生产实践？

#### 77.1 生产规则

```bash
# 默认拒绝
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# 允许
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

#### 77.2fail2ban

```bash
# 防暴力破解
yum install fail2ban
systemctl enable fail2ban

# 配置
/etc/fail2ban/jail.local
[ sshd ]
enabled = true
port = ssh
filter = sshd
maxretry = 3
```

#### 77.3 回答模板

> 防火墙：默认拒绝、允许LIST。fail2ban防暴力破解。日志告警。

---

### 78. 备份策略？

#### 78.1 备份方案

```bash
# 完整+增量
# rSync同步
# Snapshot快照
# 云端备份
```

#### 78.2 备份命令

```bash
# tar打包
tar -cvpzf backup.tar.gz /data

# rsync差异
rsync -avz --delete /data/ /backup/

# mysqldump
mysqldump -u root -p db > db.sql
```

#### 78.3 回答模板

> 备份：3221原则（每日增、周全、月归档），离线备份。3-2-1存储。

---

### 79. 恢复演练？

#### 79.1 恢复脚本

```bash
#!/bin/bash
# 停止服务
systemctl stop app
# 恢复数据
tar -xvpfz backup.tar.gz -C /data
# 启动
systemctl start app
# 验证
curl localhost:8080/health
```

#### 79.2 回答模板

> 备份必须演练。恢复脚本化、定期测。RPO/RTO定义。

---

### 80. 权限管理最佳实践？

#### 80.1 最小权限

```bash
# 文件644
chmod 644 /etc/app/conf
# 目录755
chmod 755 /etc/app
# 禁止
chmod 000 /etc/shadow
# 用户
useradd -r -s /sbin/nologin app
chown app:app /data
```

#### 80.2 audit审计

```bash
# auditctl
auditctl -w /etc/passwd -p wa -k user_mod
auditctl -w /usr/bin -p x -k cmd_exec
ausearch -k user_mod
```

#### 80.3 回答模板

> 最小权限原则。audit审计关键文件。敏感日志留存。

---

## 第七章 综合实战（中高级 ★★★★）

### 81. CPU 100%排查实战？

#### 81.1 排查步骤

```bash
# 1. 定位进程
top
# CPU 100%。python进程

# 2. 定位线程
top -H -p python_pid
# main线程最高

# 3. 堆栈
pstack python_pid
# jstack python_pid

# 4. 代码定位
strace -p python_pid
# 大量系统调用
```

#### 81.2 解决

```python
# 死循环
while True:
    process()
```

#### 81.3 回答模板

> CPU高：top→top -H→pstack/jstack→strace。Java jstack分析线程。

---

### 82. 内存溢出排查实战？

#### 82.1 排查

```bash
# 1. 内存使用
free -h
# used 95%

# 2. Java进程
jps -ml
jstat -gcutil pid 1000

# 3. dump分析
jmap -dump:file=/tmp/heap.hprof pid

# 4. MAT分析
```

#### 82.2 原因

```java
// 内存泄漏
static List list = new ArrayList();
while(true) {
    list.add(new Object());
}
```

#### 82.3 回答模板

> 内存爆满：java进程OOM。jmap dump，MAT分析对象。代码层面修复。

---

### 83. 磁盘爆满排查实战？

#### 83.1 排查

```bash
df -h
#/ 100%

du -sh /*
# /var最大

du -sh /var/*
# /var/log最大

ls -lS /var/log | head
# app.log 10G
```

#### 83.2 解决

```bash
# 日志轮转
logrotate配置

# 清理
rm app.log
# 磁盘清理
find / -type f -mtime +30 -delete
```

#### 83.3 回答模板

> 磁盘满：排序目录定位。大文件日志删，logrotate防复发。

---

### 84. 无法SSH连接排查？

#### 84.1 排查

```bash
# 1. 网络
ping ip

# 2. 端口
nc -vz ip 22
# refused

# 3. 服务
systemctl status sshd

# 4. 防火墙
iptables -L -n | grep 22
# drop
```

#### 84.2 解决

```bash
# 物理机console
# iptables -I INPUT -p tcp --dport 22 -j ACCEPT
systemctl start sshd
```

#### 84.3 回答模板

> SSH连不上：ping→端口→服务→防火墙排查。console物理登录解决。

---

### 85. 服务启动失败排查？

#### 85.1 排查

```bash
# 1. 日志
journalctl -u nginx -n 50
# bind()failed

# 2. 端口
ss -tlnp | grep 80
# ��被��

# 3. 权限
ls -l /var/run
# socket权限
```

#### 85.2 解决了

```bash
# kill旧进程
pkill nginx
# 改端口
listen 8080
```

#### 85.3 回答模板

> 服务启动失败journalctl查看日志。端口占用kill，服务配置修复。

---

### 86. 网络延迟高排查？

#### 86.1 排查

```bash
# 1. ping延迟
ping -c 10 host

# 2. traceroute
traceroute host

# 3. DNS
nslookup host

# 4. ss连接状态
```

#### 86.2 解决

```bash
# DNS慢
vi /etc/resolv.conf
# 8.8.8.8

# 网络抖动
mtr host
```

#### 86.3 回答模板

> 延迟高：ping+traceroute定位段。DNS慢改公共DNS。网络抖动抓包。

---

### 87. 数据库慢排查？

#### 87.1 排查

```bash
# 1. slow query
mysqldumpslow slow.log

# 2. EXPLAIN
explain sql

# 3. 索引
show index from tbl

# 4. processlist
show processlist
```

#### 87.2 解决

```sql
ALTER TABLE tbl ADD INDEX idx_col(col);
```

#### 87.3 回答模板

> 慢查询：slow query log分析，EXPLAIN看执行计划，加索引调优。

---

### 88. RAID配置？

#### 88.1 RAID级别

| 级别 | 原理 | 特点 |
|------|------|------|
| 0 | 条带 | 快，无冗余 |
| 1 | 镜像 | 50%空间 |
| 5 | 校验 | 1块盘容错 |
| 10 | 0+1 | 性能和冗余 |
| 6 | 双校验 | 2块盘容错 |

#### 88.2 MDADM

```bash
# 创建RAID5
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sd[bcd]1

# 格式化
mkfs.ext4 /dev/md0

# 监控
mdadm --detail /dev/md0
```

#### 88.3 回答模板

> RAID5用3块盘1块容错。mdadm管理，/proc/mdstat监控。生产数据RAID。

---

### 89. LVM逻辑卷管理？

#### 89.1 概念

```plaintext
LVM：
- PV Physical Volume
- VG Volume Group
- LV Logical Volume
- PE Physical Extent
```

#### 89.2 操作

```bash
# 创建PV
pvcreate /dev/sdb1

# 创建VG
vgcreate vgdata /dev/sdb1

# 创建LV
lvcreate -L 10G -n lvdata vgdata

# 扩展
lvextend -L +10G /dev/vgdata/lvdata
resize2fs /dev/vgdata/lvdata
```

#### 89.3 回答模板

> LVM：灵活调整大小。PV→VG→LV→文件系统。热扩展收缩。

---

### 90. 高性能网络优化？

#### 90.1 内核参数

```bash
# sysctl.conf
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 87380 16777216
net.core.netdev_max_backlog=8000
net.ipv4.tcp_window_scaling=1
```

#### 90.2 网卡优化

```bash
# ethtool
ethtool -G eth0 rx 4096 tx 4096
ethtool -K eth0 tso on
ethtool -K eth0 gso on
```

#### 90.3 回答模板

> 网络优化：TCP窗口、拥塞控制、网卡offload。万兆网卡+ Jumbo Frame。

---

### 91. 内核参数优化？

#### 91.1 常用参数

```bash
# sysctl -w net.ipv4.tcp_syncookies=1
# sysctl -w net.ipv4.conf.all.rp_filter=1
# sysctl -w net.ipv4.conf.default.rp_filter=1
# sysctl -w kernel.sysrq=0
# sysctl -w kernel.core_uses_pid=1
```

#### 91.2 文件

```bash
# /etc/sysctl.conf
# 永久生效
sysctl -p
```

#### 91.3 回答模板

> 生产内核优化：tcp相关、kernel参数。sysctl -p生效。

---

### 92. Systemtap性能分析？

#### 92.1 使用

```bash
# 内核级追踪
yum install systemtap
stap -e 'probe kernel.function("sys_sync") { printf("sync called\n"); }'
```

#### 92.2 回答模板

> Systemtap：内核动态追踪。DTraceLinux版。无侵入。

---

### 93. 内核升级？

#### 93.1 升级

```bash
# CentOS
yum install kernel
# grub2-set-default 0
# reboot

# Ubuntu
apt install linux-image-generic-hwe-20.04
```

#### 93.2 回答模板

> 内核升级：centos yum/apt install。生产谨慎，新内核先测试。

---

### 94. 业务指标命令？

#### 94.1 命令行

```bash
#  uptime        运行时间
#  w            在线用户
#  dstat        综合性能
#  htop        交互监控
#  atop        资源Top
#  iotop       IO TOP
#  nethogs     网终流量
#  iftop       网卡流量
#  smem        内存分布
```

#### 94.2 回答模板

> 这些命令全覆盖运维场景。htop/atop/iotop/nethogs/iftop各有侧重。

---

### 95. 常见端口？

#### 95.1 端口速查

| 端口 | 服务 | 说明 |
|------|------|------|
| 21 | FTP | 文件传输 |
| 22 | SSH | 远程 |
| 23 | Telnet | 明文远程 |
| 25 | SMTP | 邮件 |
| 53 | DNS | 域名 |
| 80 | HTTP | Web |
| 110 | POP3 | 邮件收 |
| 143 | IMAP | 邮件 |
| 443 | HTTPS | 安全Web |
| 3306 | MySQL | 数据库 |
| 5432 | PostgreSQL | 数据库 |
| 6379 | Redis | 缓存 |
| 27017 | MongoDB | 数据库 |
| 9200 | ES | 搜索 |
| 5601 | Kibana | UI |
| 9090 | Prometheus | 监控 |

---

### 96. 常见日志路径？

#### 96.1 日志目录

```plaintext
/var/log/messages    系统日志
/var/log/secure     认证
/var/log/maillog   邮件
/var/log/cron      定时任务
/var/log/dmesg     启动日志
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/httpd/access_log
/var/log/httpd/error_log
```

---

### 97. 故障排查流程？

#### 97.1 排查流程

```plaintext
故障排查步骤：
1. 确认问题现象
2. 收集信息（top/ps/netstat/logs）
3. 定位根因
4. 制定方案
5. 实施修复
6. 验证恢复
7. 记录复盘
```

---

### 98. 经验教训记录？

#### 98.1 复盘

```markdown
# 故障复盘
## 时间
## 影响
## 原因分析
## 教训
## 改进措施
```

---

### 99. 高可用方案？

#### 99.1 HA方案

```plaintext
HA：
- Heartbeat（经典）
- Pacemaker + Corosync
- Keepalived（LVS/VRRP）
- HAProxy + Keepalived
```

#### 99.2 VRRP

```bash
# keepalived.conf
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    virtual_ipaddress {
        192.168.1.100
    }
}
```

---

### 100. 职业素养？

#### 100.1 核心能力

```plaintext
运维核心：
- 扎实基础
- 自动化能力
- 监控意识
- 安全意识
- 文档能力
- 沟通能力
- 学习能力
```

#### 100.2 回答模板

> 运维成长：扎实基础、文档沉淀、自动化提效、监控告警。持续学习新技术。背锅也要心态好。

---

## 附录：面试追问

1. **Linux启动流程？**
   答：BIOS→BootLoader→Kernel→Init→RunLevel

2. **TCP三次握手四次挥手？**
   答：SYN→SYN+ACK→ACK，FIN→ACK→FIN→ACK→TIME_WAIT

3. **HTTP状态码？**
   答：2xx成功，3xx重定向，4xx客户端错误，5xx服务端错误

4. **常用Shell？**
   答：Bash(sh)、Zsh、Fish。生产主力bash。

---

## 参考资料

- 《Linux鸟哥私房菜》
- 《Linux高性能服务器编程》
- 《UNIX环境高级编程》
- man手册

---

> 整理by Claude Code | Linux与运维面试高频100问