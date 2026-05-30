# Docker与K8S 100问——容器技术核心深度指南

> 本文档面向容器技术学习者，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 Docker基础（高频 ★★★★★）

### 1. 什么是Docker？

#### 1.1 定义

> Docker是一个开源的容器化平台，让开发者可以打包应用及其依赖到一个可移植的容器中。

```plaintext
Docker核心组件：
- Docker Daemon        守护进程，管理容器/镜像/网络
- Docker Client       CLI客户端
- Docker Registry    镜像仓库（Docker Hub、私有仓库）
- Docker Image      只读模板
- Docker Container  镜像的运行实例
```

#### 1.2 回答模板

> Docker是容器化技术，把应用和依赖打包成镜像，运行在隔离的容器中。相较VM更轻量、启动更快、资源占用更少。

---

### 2. Docker与虚拟机的区别？

#### 2.1 对比

| 特性 | Docker容器 | 虚拟机 |
|------|----------|--------|
| 隔离级别 | 进程级 | 操作系统级 |
| 启动时间 | 秒级 | 分钟级 |
| 镜像大小 | MB级 | GB级 |
| 资源占用 | 共享内核 | 独立内核 |
| 性能开销 | 低 | 高 |

#### 2.2 回答模板

> Docker是进程级隔离，共享宿主机内核，所以更轻量；VM是操作系统级隔离，完全独立。容器启动秒级，镜像MB级；VM启动分钟级，镜像GB级。

---

### 3. Docker镜像层级？

#### 3.1 镜像原理

```dockerfile
# 镜像分层结构
FROM ubuntu:20.04        # 基础层
RUN apt-get update     # 层1
RUN apt-get nginx   # 层2
COPY ./app /app     # 层3
CMD ["nginx"]      # 层4
```

```plaintext
UnionFS联合文件系统：
- 分层叠加
- 共享底层
- 增量存储
- COW（Copy on Write）
```

#### 3.2 回答模板

> Docker镜像是分层结构，每层只存增量。UnionFS叠加只读层运行时合并。Docker优化层数减少存储和启动时间。

---

### 4. Dockerfile常用指令？

#### 4.1 指令详解

```dockerfile
FROM          # 基础镜像
RUN           # 构建时执行命令
COPY          # 复制文件
ADD           # 复制（可解压URL）
WORKDIR       # 工作目录
ENV           # 环境变量
EXPOSE        # 暴露端口
USER         # 指定用户
VOLUME        # 卷
ENTRYPOINT   # 入口（不被覆盖）
CMD         # 默认命令（可被覆盖）
```

#### 4.2 最佳实践

```dockerfile
# 多阶段构建
FROM golang:1.20 AS builder
WORKDIR /build
COPY . .
RUN go build -o app

FROM alpine:latest
WORKDIR /app
COPY --from=builder /build/app .
ENTRYPOINT ["./app"]
```

#### 4.3 回答模板

> Dockerfile指令：FROM基础、RUN执行、COPY复制、WORKDIR目录、ENV环境、EXPOSE端口、CMD命令。多阶段构建减少镜像体积。

---

### 5. Docker网络模式？

#### 5.1 网络模式

```bash
# bridge（默认）: NAT映射
# host: 共享主机网络
# none: 无网络
# container: 共享其他容器网络
```

#### 5.2 回答模板

> Docker四种网络：bridge默认NAT、host共享主机网络、none无网络、container共享容器网络。生产用自定义bridge或overlay。

---

### 6. Docker存储卷？

#### 6.1 卷的类型

```bash
# 命名卷
docker volume create myvolume
docker run -v myvolume:/data ...

# 绑定挂载
docker run -v /host/path:/container/path ...

# tmpfs（内存）
docker run --tmpfs /tmp ...
```

#### 6.2 回答模板

> Docker存储：有命名卷、绑定挂载、tmpfs内存卷。持久化用命名卷，开发调试用绑定挂载���

---

### 7. Dockerfile优化技巧？

#### 7.1 优化原则

```dockerfile
# 1. 减少层数
RUN apt-get update && apt-get install -y nginx && rm -rf /var/lib/apt/*

# 2. 调整指令顺序（利用缓存）
COPY package.json .    # 变更少的放前面
RUN npm install
COPY . .

# 3. 使用多阶段构建
# 4. 使用.mini基础镜像（alpine）
# 5. .dockerignore排除无关文件
```

#### 7.2 回答模板

> Dockerfile优化：减少层数、调整指令顺序、利用缓存、多阶段构建、使用alpine基础镜像。

---

### 8. Docker Compose？

#### 8.1 定义

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:80"
  db:
    image: postgres:14
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

#### 8.2 回答模板

> Docker Compose用YAML定义多容器应用。一个命令运行整个栈。适合开发、测试、单机器部署。

---

## 第二章 Kubernetes基础（高频 ★★★★★）

### 9. 什么是Kubernetes？

#### 9.1 定义

> Kubernetes（K8S）是Google开源的容器编排平台，自动部署、扩缩容、管理容器化应用。

```plaintext
K8S核心能力：
- 自我修复
- 水平扩展
- 服务发现
- 负载均衡
- 自动部署回滚
- 配置管理
- 存储编排
```

#### 9.2 回答模板

> K8S是容器编排平台，实现容器自动化管理。自愈、扩缩容、服务发现、滚动更新是企业级容器运维标配。

---

### 10. K8S架构组成？

#### 10.1 核心组件

```plaintext
Master节点（Control Plane）：
├── kube-apiserver       API入口
├── kube-controller-manager 控制器
├── kube-scheduler       调度器
└── etcd               存储

Worker节点：
├── kubelet             容器运行时接口
├── kube-proxy         网络代理
└── Container Runtime  容器引擎
```

#### 10.2 回答模板

> K8S架构：Master管理（API Server、Controller、Scheduler、etcd），Worker运行（kubelet、kube-proxy、Container Runtime）。

---

### 11. Pod是什么？

#### 11.1 定义

> Pod是K8S最小部署单元，一个Pod包含一个或多个共享网络的容器。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
```

```plaintext
Pod特性：
- 共享网络（localhost通信）
- 共享存储
- 共享Linux命名空间
- 原子性部署/扩缩容
```

#### 11.2 回答模板

> Pod是K8S最小单元，同Pod内容器共享网络和存储，localhost互访。Production用Deployment管理Pod。

---

### 12. Pod生命周期？

#### 12.1 状态

```
Pending    → Running → Succeeded
                      → Failed
                      → Unknown
```

```plaintext
Pod生命周期：
1. Pending    创建调度的过程
2. Running   至少一个容器运行
3. Succeeded 所有容器成功结束
4. Failed   至少一个容器失败
5. Unknown  节点失联
```

#### 12.2 探针

```yaml
livenessProbe:  # 存活探针（重启容器）
  httpGet:
    path: /health
    port: 8080
readinessProbe:  # 就绪探针（加入Service）
  httpGet:
    path: /ready
    port: 8080
startupProbe:   # 启动探针（启动完成后启用其他探针）
  httpGet:
    path: /start
    port: 8080
```

#### 12.3 回答模板

> Pod五状态：Pending调度中、Running运行中、Succeeded成功、Failed失败、Unknown失联。三种探针：liveness存活、readiness就绪、startup启动。

---

### 13. ReplicaSet与Deployment？

#### 13.1 定义

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
```

```plaintext
ReplicaSet：保证指定数量Pod副本
Deployment：声明式更新，管理ReplicaSet
```

#### 13.2 更新策略

```yaml
spec:
  strategy:
    type: RollingUpdate    # 默认滚动更新
    rollingUpdate:
      maxSurge: 1        # 最多超出期望数
      maxUnavailable: 0   # 最少可用数
```

#### 13.3 回答模板

> Deployment声明式管理Pod，实现滚动更新、最大可用。Replicaset保证副本数，生产必须用Deployment。

---

### 14. Service？

#### 14.1 定义

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP    # ClusterIP/NodePort/LoadBalancer/ExternalName
```

```plaintext
Service类型：
- ClusterIP      集群内部IP（默认）
- NodePort      宿主机端口
- LoadBalancer  云厂商LB
- ExternalName  DNS别名
```

#### 14.2 工作原理

```plaintext
Service发现：
- kube-proxy更新iptables/IPVS
- Endpoint同步Pod变化
- DNS解析Service IP
```

#### 14.3 回答模板

> Service为Pod提供统一入口和负载均衡。ClusterIP集群内、NodePort宿主机端口、LoadBalancer云LB。kube-proxy维护转发规则。

---

### 15. Ingress？

#### 15.1 定义

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-svc
            port:
              number: 80
```

#### 15.2 Ingress Controller

```plaintext
Ingress实现：
- Nginx Ingress Controller
- Traefik
- Envoy
- 云厂商ALB
```

#### 15.3 回答模板

> Ingress提供HTTP/HTTPS路由，基于域名或路径转发到Service。需要Ingress Controller（如Nginx）生效。

---

### 16. ConfigMap与Secret？

#### 16.1 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
    key: value
  file.conf: |
    server.port=8080
```

#### 16.2 Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=     # base64编码
  password: cGFzc3dvcmQ=
```

#### 16.3 使用方式

```yaml
env:
- name: CONFIG_PATH
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: config.yaml

volumeMounts:
- name: config
  mountPath: /etc/config
  readOnly: true
```

#### 16.4 回答模板

> ConfigMap存配置，Secret存敏感信息（base64编码）。环境变量或卷挂载方式使用。生产密钥用Vault或KMS。

---

### 17. Volume存储？

#### 17.1 EmptyDir

```yaml
volumes:
- name: cache
  emptyDir: {}
```

#### 17.2 PersistentVolume（PV）+ PersistentVolumeClaim（PVC）

```yaml
# PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  hostPath:
    path: /data

# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
```

#### 17.3 回答模板

> K8S存储：EmptyDir临时存储、HostPath节点存储、PV/PVC持久化存储。PVC动态制备需StorageClass。

---

### 18. 污点和容忍？

#### 18.1 Taint/Toleration

```bash
# 标记节点
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 key=value:NoExecute
kubectl taint nodes node1 key=value:PreferNoSchedule
```

```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

#### 18.2 应用场景

```plaintext
污点场景：
- 专用节点（GPU/SSD）
- 维护驱逐（NoExecute）
- 资源紧张时低优先级（PreferNoSchedule）
```

#### 18.3 回答模板

> Taint标记节点不可调度，Toleration让Pod容忍。用于专用节点、维护驱逐、差异化调度。

---

### 19. 亲和性调度？

#### 19.1 NodeAffinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

#### 19.2 PodAffinity/AntiAffinity

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
      matchLabels:
        app: web
      topologyKey: kubernetes.io/hostname
```

#### 19.3 回答模板

> 亲和性调度：nodeAffinity节点亲和、podAffinity Pod亲和、podAntiAntiAffinity反亲和。控制Pod共置或分散。

---

### 20. 资源配额？

#### 20.1 ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"
    limits.cpu: "8"
    requests.memory: "4Gi"
    limits.memory: "8Gi"
```

#### 20.2 LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limits
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "200m"
      memory: "128Mi"
    type: Container
```

#### 20.3 回答模板

> ResourceQuota命名空间级总量限制，LimitRange单个Pod默认上限。防止资源耗尽必须设置。

---

## 第三章 K8S进阶（高频 ★★★★★）

### 21. HorizontalPodAutoscaler（HPA）？

#### 21.1 定义

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

#### 21.2 工作原理

```plaintext
HPA流程：
1. Metrics Server收集指标
2. 计算期望副本数
3. 调用Scale接口调整
4. Deployment响应扩容
```

#### 21.3 回答模板

> HPA基于CPU/内存利用率自动扩缩容Pod。需要Metrics Server，K8S 1.18+支持自定义指标。

---

### 22. StatefulSet？

#### 22.1 定义

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp
spec:
  serviceName: myapp
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

#### 22.2 特性

```plaintext
StatefulSet特性：
- 固定序号标识
- 稳定网络标识
- 稳定存储（PVC）
- 有序部署/扩缩容
- 有序更新
```

#### 22.3 回答模板

> StatefulSet有状态应用：稳定网络ID、稳定存储、有序扩缩。适用于数据库、消息队列等。Deployment用于无状态。

---

### 23. DaemonSet？

#### 23.1 定义

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
      - name: agent
        image: log-agent:1.0
```

#### 23.2 使用场景

```plaintext
DaemonSet场景：
- 日志采集（Filebeat、Fluentd）
- 监控（Prometheus Node Exporter）
- 网络插件（CNI）
- 存储插件（CSI）
```

#### 23.3 回答模板

> DaemonSet每个节点运行一个Pod。用于日志、监控、网络插件等系统级组件。

---

### 24. Job与CronJob？

#### 24.1 Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: task
        image: mytask:1.0
```

#### 24.2 CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mycron
spec:
  schedule: "0 * * * *"    # 每小时
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: job
            image: myjob:1.0
          restartPolicy: OnFailure
```

#### 24.3 回答模板

> Job一次性任务，CronJob定时任务。restartPolicy控制失败策略：Always、OnFailure、Never。

---

### 25. SecurityContext？

#### 25.1 Pod级

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
```

#### 25.2 容器级

```yaml
containers:
- name: app
  securityContext:
    readOnlyRootFilesystem: true
    allowPrivilegeEscalation: false
    capabilities:
      drop:
      - ALL
```

#### 25.3 回答模板

> SecurityContext控制容器安全：非root运行、只读根文件系统、禁用特权提升、能力集控制。

---

### 26. Pod安全策略（PSP）？

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name:restricted
spec:
  privileged: false
  runAsUser:
    rule: MustRunAsNonRoot
  seLinux:
    rule: RunAsAny
  volumes:
  - configMap
  - emptyDir
  - secret
```

#### 26.2 回答模板

> PSP定义Pod安全标准：禁止特权提升、非root运行、限制存储卷。K8S 1.25+移除，用OPA Gatekeeper替代。

---

### 27. 网络策略？

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api
    ports:
    - protocol: TCP
      port: 5432
```

#### 27.2 回答模板

> NetworkPolicy控制Pod流量出入。默认拒绝，需显式声明白名单。微分段安全必备。

---

### 28. ServiceAccount与RBAC？

#### 28.1 ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
```

#### 28.2 RBAC

```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: myrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: myrolebinding
subjects:
- kind: ServiceAccount
  name: myapp-sa
  namespace: default
roleRef:
  kind: Role
  name: myrole
  apiGroup: rbac.authorization.k8s.io
```

#### 28.3 回答模板

> RBAC基于角色授权：Role（命名空间级）、ClusterRole（集群级）、RoleBinding绑定。最小权限原则。

---

### 29. Helm包管理器？

#### 29.1 Chart结构

```plaintext
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   └── deployment.yaml
└── charts/
```

#### 29.2 命令

```bash
helm install myapp ./myapp
helm upgrade myapp ./myapp
helm rollback myapp 1
helm uninstall myapp
```

#### 29.3 回答模板

> Helm是K8S包管理，类似yum/apt。Chart定义应用资源模板，values.yaml配置参数，实现一键部署升级。

---

### 30. Operator与CRD？

#### 30.1 CRD

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.example.com
spec:
  group: example.com
  names:
    kind: MyApp
    listKind: MyAppList
    plural: myapps
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
```

#### 30.2 Operator

```plaintext
Operator模式：
- 自定义资源（CR）
- 自定义控制器
- 协调逻辑
- 封装运维知识
```

#### 30.3 回答模板

> Operator自定义资源和控制器，封装复杂应用运维。使用场景：数据库、消息队列、备份恢复。

---

## 第四章 存储与网络（中高级 ★★★★）

### 31. CSI存储驱动？

#### 31.1 接口

```plaintext
CSI（Container Storage Interface）：
- Identity Service
- Controller Service
- Node Service
```

#### 31.2 常见驱动

```plaintext
CSI驱动：
- NFS
- Ceph RBD
- iSCSI
- AWS EBS
- GCP PD
- Azure Disk
```

#### 31.3 回答模板

> CSI标准化存储接口。主流云厂商存储、社区驱动都支持CSI。StorageClass动态制备。

---

### 32. CNI网络插件？

#### 32.1 主流CNI

```plaintext
CNI插件：
- Flannel        VXLAN叠加网络
- Calico        BGP网络策略
- Cilium        eBPF网络策略
- Weave Net    加密叠加
- AWS VPC CNI  VPC原生
```

#### 32.2 选型

```plaintext
选择依据：
- 性能：Calico/Cilium
- 简单：Flannel
- 安全：Flannel WireGuard/Cilium
- 云厂商：VPC CNI
```

#### 32.3 回答模板

> CNI管理K8S网络。Flannel简单、Calico性能和安全好。云上可用VPC CNI减少网络开销。

---

### 33. Service Mesh？

#### 33.1 Istio架构

```plaintext
Istio组件：
- Control Plane（istiod）
- Data Plane（Envoy Sidecar）
```

#### 33.2 核心功能

```plaintext
Istio能力：
- 流量管理（VirtualService、DestinationRule）
- 可观测性（Prometheus、Grafana、Jaeger）
- 安全（mTLS）
- 策略控制（Rate Limit）
```

#### 33.3 回答模板

> Istio是ServiceMesh方案，Envoy Sidecar代理流量。提供服务治理、可观测、安全能力。适合微服务治理。

---

### 34. K8S资源对象关系？

#### 34.1 对象关系

```plaintext
资源层次：
Deployment → ReplicaSet → Pod
      ↓
Service（EndPoints） → Pod
      ↓
ConfigMap/Secret
```

#### 34.2 回答模板

> K8S对象有层级：Deployment管理RS，RS管Pod，Service暴露Pod。网络通过Endpoints关联。

---

### 35. 调度器如何工作？

#### 35.1 调度流程

```plaintext
调度阶段：
1. Predicate 过滤（节点筛选）
2. Priority 排序（权重计算）
3. Bind   绑定（更新NodeName）
```

#### 35.2 调度算法

```plaintext
Predicate过滤：
- PodFitsResources
- HostFitsPorts
- MatchNodeSelector
- NoDiskConflict

Priority排序：
- LeastRequestedCPU
- BalancedResourceAllocation
- ImageLocality
```

#### 35.3 回答模板

> K8S调度器：Predicate过滤不符合节点，Priority权重排序，选高分节点Bind。影响因子：资源、标签、亲和性。

---

## 第五章 运维与实战（中高级 ★★★★）

### 36. K8S集群高可用部署？

#### 36.1 Master高可用

```plaintext
HA部署：
- 多Master节点（3或5）
- etcd集群
- VIP/LB透传API Server
- 选主机制
```

#### 36.2 Worker高可用

```plaintext
Worker HA：
- 打散到多可用区
- Pod反亲和
- HPA自动扩容
```

#### 36.3 回答模板

> K8S高可用：Master 3节点etcd集群，Worker打散多AZ，配合HPA实现应用层高可用。

---

### 37. 资源限制最佳实践？

#### 37.1 设置原则

```yaml
resources:
  requests:      # 调度依据
    cpu: "100m"
    memory: "128Mi"
  limits:        # 上限
    cpu: "500m"
    memory: "512Mi"
```

#### 37.2 实践建议

```plaintext
资源设置：
- requests必须设（调度依据）
- limits建议设（防止失控）
- 初始值参考已有数据
- 压测调优
```

#### 37.3 回答模板

> 资源request用于调度，limit防止失控。初始值参考现有应用数据，压测后优化。生产必须设置。

---

### 38. 健康检查最佳实践？

#### 38.1 探针设置

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
failureThreshold: 3
```

#### 38.2 实践要点

```plaintext
健康检查要点：
- liveness检测进程存活
- readiness检测流量就绪
- 避免过早探活失败
- 日志记录探活失败原因
```

#### 38.3 回答模板

> liveness保活、readiness引流。就绪后才接收流量，避免启动慢导致探活失败。initialDelaySeconds重要。

---

### 39. 日志采集方案？

```plaintext
方案选择：
- 本地日志 + DaemonSet（Elastic/Fluentd）
-Sidecar + 抽水（Filebeat）
- 业务直接写（Kafka/ES）
```

#### 39.2 实践建议

```plaintext
日志采集：
- 控制日志级别
- 结构化日志（JSON）
- 标准输出stderr
- 日志轮转（max-size/max-file）
```

#### 39.3 回答模板

> K8S日志：容器stdout/stderr输出，DaemonSet采集。生产用Fluentd/Elastic/Filebeat。

---

### 40. 监控体系？

#### 40.1 Prometheus架构

```plaintext
Prometheus：
- scraper拉取指标
- TSDB存储时序数据
- PromQL查询
- AlertManager告警
- Grafana可视化
```

#### 40.2 K8S指标

```plaintext
K8S监控指标：
- 资源（CPU/Memory/磁盘/网络）
- 应用（QPS/延迟/错误）
- K8S对象（Pod/Deployment/Service）
```

#### 40.3 回答模板

> K8S监控用Prometheus+Grafana。Exporters采集指标，AlertManager告警。关键指标：CPU、内存、延迟、错误率。

---

### 41. 滚动更新与回滚？

#### 41.1 更新流程

```bash
# 滚动更新
kubectl set image deployment/myapp app=myapp:2.0
kubectl rollout status deployment/myapp

# 查看历史
kubectl rollout history deployment/myapp

# 回滚
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=2
```

#### 41.2 实践注意

```plaintext
滚动更新注意：
- 保持可用性（maxUnavailable）
- 逐步替换（maxSurge）
- 健康检查配合
- 金丝雀发布
```

#### 41.3 回答模板

> kubectl rollout管理发布。滚动更新保持可用，maxSurge/maxUnavailable控制节奏。一键回滚。

---

### 42. 金丝雀发布？

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
---
apiVersion: v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1      # 小流量
  selector:
    matchLabels:
      app: myapp
      track: canary
```

```bash
# 流量切换
kubectl scale deployment myapp-v2 --replicas=5
kubectl scale deployment myapp-v1 --replicas=0
```

#### 42.2 回答模板

> 金丝雀：小比例新版本先上，观察无问题后逐步放大。可用Deployment或Istio VirtualService控制流量。

---

### 43. 多集群管理？

#### 43.1 多集群方案

```plaintext
多集群方案：
- Rancher
- kubefed（Federation）
- ArgoCD（GitOps）
- OCM（MultiCluster）
```

#### 43.2 应用场景

```plaintext
多集群场景：
- 多地域部署
- 流量分发
- 容灾切换
- 混合云
```

#### 43.3 回答模板

> 多集群管理用Rancher/OCM。异地多活流量切换需Global DNS/LB配合。

---

### 44. GitOps实践？

#### 44.1 工作流

```plaintext
GitOps流程：
1. 代码入Git
2. ArgoCD/Flux同步
3. 自动部署
4.  drift检测
```

#### 44.2 回答模板

> GitOps：Git为单一真相源，ArgoCD自动同步。实现声明式基础设施，版本可追溯。

---

### 45. 备份与恢复？

#### 45.1 etcd备份

```bash
# 备份
ETCDCTL_API=3 etcdctl snapshot save backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 恢复
ETCDCTL_API=3 etcdctl snapshot restore backup.db \
  --data-dir=/var/lib/etcd
```

#### 45.2 应用状态备份

```plaintext
备份方案：
- PVC快照（VolumeSnapshot）
- Velero应用备份
- 数据库级备份
```

#### 45.3 回答模板

> etcd集群关键，定期备份。应用数据用PVC快照或Velero。数据库还需应用级逻辑备份。

---

### 46. K8S安全最佳实践？

#### 46.1 镜像安全

```plaintext
镜像安全：
- 最小化基础镜像
- 减少非必要工具
- 多阶段构建减少攻击面
- 定期扫描漏洞
```

#### 46.2 运行时安全

```plaintext
运行时安全：
- 非root运行
- 只读filesystem（ReadOnlyRootFilesystem）
- 禁止特权容器
- 网络策略
- 审计日志
```

#### 46.3 回答模板

> 安全实践：镜像最小化、非root运行、只读根文件系统、禁止特权、网络策略、RBAC最小权限。

---

### 47. 故障排查思路？

#### 47.1 排查步骤

```bash
# 1. Pod状态
kubectl get pod -o wide
kubectl describe pod <pod>

# 2. 日志
kubectl logs <pod> -f
kubectl logs <pod> --previous

# 3. 容器内
kubectl exec -it <pod> -- /bin/sh

# 4. 事件
kubectl get events --sort-by='.lastTimestamp'

# 5. 资源
kubectl top pod
kubectl top node
```

#### 47.2 常见问题

```plaintext
常见故障：
- CrashLoopBackOff（启动失败）
- ImagePullBackOff（拉取失败）
- OOMKilled（内存超限）
- Pending（资源不足）
- Evicted（节点压力驱逐）
```

#### 47.3 回答模板

> 故障排查：describe查看Events，logs看日志，exec进入容器。常见问题：启动失败、拉取失败、资源不足。

---

### 48. 问题诊断命令速查？

| 命令 | 用途 |
|------|------|
| kubectl get pods -o wide | Pod状态 |
| kubectl describe pod xxx | 详情事件 |
| kubectl logs xxx | 当前日志 |
| kubectl logs xxx --previous | 上次重启日志 |
| kubectl exec -it xxx -- sh | 进入容器 |
| kubectl top pod/node | 资源使用 |
| kubectl get events | 最近事件 |
| kubectl cp ns/pod:path ./ | 复制文件 |

---

### 49. 云原生存储方案？

#### 49.1 对象存储

```plaintext
云存储：
- AWS S3 / MinIO
- Azure Blob
- GCP Cloud Storage
```

#### 49.2 块存储

```plaintext
云盘：
- AWS EBS
- Azure Managed Disk
- GCP PD
```

#### 49.3 文件存储

```plaintext
文件存储：
- AWS EFS
- Azure Files
- GCP Filestore
```

#### 49.4 回答模板

> 云存储分类：对象（S3/MinIO）、块（EBS/AZ云盘）、文件（EFS）。按场景选择。

---

### 50. 降本增效实践？

#### 50.1 成本优化

```plaintext
优化手段：
- 合理requests/limits（压测后调整）
- Spot实例/Preemptible VM
- 资源预留+弹性
- 删除闲置资源
```

#### 50.2 实践建议

```plaintext
降低成本：
- 生产用Spot节省70%
- 非_prod低成本资源
- HPA自动弹性伸缩
- 定期审计资源
```

#### 50.3 回答模板

> K8S成本：Spot实例省70%，合理资源配置，HPA自动伸缩，定期清理闲置资源。

---

## 第六章 集成与生态（中高级 ★★★★）

### 51. K8S与CI/CD集成？

#### 51.1 GitLab CI

```yaml
deploy:
  stage: deploy
  script:
    - kubectl set image deployment/myapp app=$IMAGE_TAG
    - kubectl rollout status deployment/myapp
  only:
    - master
```

#### 51.2 Jenkins

```groovy
stage('Deploy K8S') {
  steps {
    sh 'kubectl set image deployment/myapp app=${IMAGE_TAG}'
    sh 'kubectl rollout status deployment/myapp'
  }
}
```

#### 51.3 回答模板

> CI/CD集成：Jenkins/GitLabRunner调用kubectl部署。image tag方式触发滚动更新。

---

### 52. Prometheus持久化？

#### 52.1 存储配置

```yaml
prometheus:
  retention: 15d
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: ssd
        resources:
          requests:
            storage: 50Gi
```

#### 52.2 回答模板

> Prometheus存储用PVC，TSDB数据保留15天。数据量大考虑联邦或远程存储。

---

### 53. Operator开发框架？

#### 53.1 Kubebuilder

```bash
# 创建项目
kubebuilder init --domain example.com
kubebuilder create api --group webapp --version v1 --kind App

# 生成代码
make generate
make build
```

#### 53.2 OperatorSDK

```bash
# 创建项目
operator-sdk new myapp --operator-sdk
operator-sdk add api --api-version=myapp.io/v1alpha1 --kind=MyApp
operator-sdk add controller --api-version=myapp.io/v1alpha1 --kind=MyApp
```

#### 53.3 回答模板

> Operator开发用Kubebuilder或OperatorSDK。自定义CRD+Controller协调，完成自动化运维。

---

### 54. 服务网格选型？

#### 54.1 Istio vs Linkerd

| 特性 | Istio | Linkerd |
|------|-------|-------|
| 架构 | Sidecar Proxy | Per-node Proxy |
| 性能 | 高 | 更高 |
| 功能 | 丰富 | 简洁 |
| 社区 |活跃 | 活跃 |

#### 54.2 回答模板

> Istio功能最全，Linkerd更轻量简单。生产选型依团队能力和复杂度。

---

### 55. Knative无服务器？

#### 55.1 定义

```plaintext
Knative：
- Serving  - 函数即服务
- Eventing - 事件驱动
- Build    - 构建（已废弃）
```

#### 55.2 自动扩缩容

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: myfunc
spec:
  template:
    spec:
      containers:
      - image: myfunc:1.0
```

#### 55.3 回答模板

> Knative基于K8S的Serverless，流量为0自动缩容。按请求计费，适合函数和事件驱动场景。

---

### 56. K8S与微服务架构？

#### 56.1 微服务架构

```plaintext
微服务+K8S：
- 每个微服务Deployment
- Service内部通信
- Ingress/Service暴露
- ConfigMap配置
- HPA自动弹性
```

#### 56.2 回答模板

> 微服务+K8S：Deployment管理各服务，跨服务通过Service通信。配置管理CM，环境区分ns。流量用Ingress/Istio。

---

### 57. 云厂商K8S服务对比？

| 厂商 | 服务名称 | 特点 |
|------|----------|------|
| AWS | EKS | 托管master，按节点付费 |
| GCP | GKE | Autopilot全托管 |
| Azure | AKS | 与Azure集成 |
| 阿里云 | ACK | 与阿里云生态 |

#### 57.2 自建vs托管

```plaintext
自建：
- 完全控制
- 运维成本高
- 需要专家

托管：
- 免运维
- SLA保证
- 成本相对高
```

#### 57.3 回答模板

> 云托管EKS/GKE/AKS省运维，SLA保证。自建完全掌控，适合有团队的公司。

---

### 58. K8S版本管理？

#### 58.1 版本策略

```plaintext
K8S版本：
- N N-1 N-2策略
- 奇数版本测试
- 偶数版本生产
```

#### 58.2 升级流程

```bash
# 升级检查
kubeadm upgrade plan

# 升级master
kubeadm upgrade apply v1.xx.x

# 升级node
kubectl cordon node
kubectl drain node
升级kubelet
kubectl uncordon node
```

#### 58.3 回答模板

> 用N或N-1稳定版本生产。升级先小规模测试，节点依次升级。保持etcduv备份。

---

### 59. 容���规划？

#### 59.1 估算方法

```plaintext
容量规划：
- QPS * 单Pod处理能力 = 所需Pod数
- Pod内存 * Pod数 + 系统预留 = 总内存
- 峰值2倍buffer
```

#### 59.2 实践公式

```plaintext
估算案例：
- 单Pod能抗1000 QPS
- 峰值50000 QPS
- 需要 50000/1000 * 1.2 = 60 Pod
- 1G内存/Pod → 60GB+系统预留
```

#### 59.3 回答模板

> 容量规划：峰值为基准，按单Pod能力核算，加20%-50% buffer。持续监控调整。

---

### 60. 集群联邦（Federation）？

#### 60.1 定义

```plaintext
Federation：
- 跨集群统一管理
- 同步资源
- 全局DNS/LB
```

#### 60.2 回答模板

> Federation跨集群联邦，适合多数据中心统一管理。KubeFed是官方方案。复杂场景可选Rancher。

---

## 第七章 综合实战（高级 ★★★）

### 61. 生产级K8S集群搭建？

#### 61.1 最小规模

```plaintext
最小生产：
- Master 3台（etcd同 Kubenetes）
- Worker ≥ 3台
- 跨AZ
- LB前端
```

#### 61.2 硬件建议

```plaintext
硬件选择：
- Master: 4核心/16GB/100GB SSD
- Worker: 8核心/32GB起（依工作负载）
- SSD系统盘
```

#### 61.3 回答模板

> 生产最小3 Master+3 Worker，跨AZ。用托管版更省心。硬件根据负载选型。

---

### 62. 日志ELK Stack部署？

```yaml
# Elasticsearch StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: elasticsearch
  replicas: 3
  ...
```

```bash
# Fluentd DaemonSet
kubectl apply -f fluentd-daemonset.yaml
```

#### 62.2 回答模板

> ELK Stack：Elasticsearch存储，Fluentd采集，Logstash处理，Kibana展示。小规模用Pod，大规模拆分集群。

---

### 63. Prometheus高可用部署？

#### 63.1 Thanos架构

```plaintext
Thanos：
- Sidecar - 罐装TSDB
- Store - 检索长期存储
- Query - 查询聚合
- Ruler - 告警规则
- Compact - 压缩
```

#### 63.2 回答模板

> Prometheus HA：联邦或Thanos。Thanos统一查询，长期存储。

---

### 64. 数据库K8S部署注意？

#### 64.1 MySQL/PostgreSQL

```yaml
# RDS优于自建
# 必须用Statefulset
# 高性能存储
# 持久化存储
# 资源限制严格
```

#### 64.2 实践建议

```plaintext
数据库部署：
- 首选云RDS（托管可靠）
- 非要用K8S用Operator
- 生产级存储IOPS保障
- 数据多副本
```

#### 64.3 回答模板

> 数据库部署：生产用云RDS，不建议K8S自建。必须时用Operator+高性能存储。

---

### 65. 消息队列K8S部署？

#### 65.1 Kafka on K8S

```yaml
# Kafka Strimzi Operator
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    replicas: 3
```

#### 65.2 回答模板

> Kafka on K8S用Strimzi Operator。支持KRaft模式。生产建议优先云托管（AWS MSK、Confluent）。

---

### 66. CI/CD工具选型？

#### 66.1 工具对比

| 工具 | 优点 | 缺点 |
|------|------|------|
| ArgoCD | GitOps原生、易用 | 功能相对少 |
| Flux | GitOps、Helm集成 | 配置复杂 |
| Jenkins | 插件丰富、历史久 | 配置重 |
| Tekton | 云原生、可扩展 | 学习曲线 |
| GitLab CI | 与GitLab集成 | 受限于GitLab |

#### 66.2 回答模板

> CI/CD选型：新项目用ArgoCD/Tekton，老项目Jenkins。GitOps ArgoCD最流行。

---

### 67. 混合云K8S方案？

#### 67.1 方案架构

```plaintext
混合云：
- 公有云EKS/GKE/ACK
- IDC物理机/VM
- VPC对等连接
- Cluster API统一纳管
```

#### 67.2 回答模板

> 混合云：核心业务IDC，数据云上备份。林zie多集群纳管，VPC Peering互联。

---

### 68. API网关K8S部署？

#### 68.1 网关选择

```plaintext
API Gateway：
- Kong（Plugin丰富、活跃）
- Traefik（轻量、云原生）
- Nginx（性能高）
- Envoy（ServiceMesh融合）
```

#### 68.2 Kong部署

```bash
helm install kong kong/kong \
  --set ingressController.enabled=true
```

#### 68.3 回答模板

> API网关选型：熟悉度优先。Kong插件丰富，Traefik轻量简单，Envoy与Mesh融合。

---

### 69. 多租户K8S方案？

#### 69.1 隔离方案

```plaintext
多租户隔离：
- Namespace隔离（名字空间）
- ResourceQuota + LimitRange
- NetworkPolicy隔离
- RBAC权限控制
```

#### 69.2 硬多租户

```plaintext
硬多租户：
- 独立集群
- 安全容器（gVisor/Kata）
- 节点亲和
```

#### 69.3 回答模板

> 多租户：Namespace级软隔离，硬多租户用独立集群。ResourceQuota+NetworkPolicy配合。

---

### 70. K8S备份恢复灾备？

#### 70.1 Velero备份

```bash
# 安装
velero install --provider aws --backup-location-config region=us-east-1

# 备份
velero backup create daily-backup --schedule "0 2 * * *"

# 恢复
velero restore create --from-backup daily-backup
```

#### 70.2 回答模板

> 备份工具Velero。应用+PV快照结合，定时备份 + 演练验证。

---

### 71. Node问题排查？

#### 71.1 NotReady状态

```bash
# 检查
kubectl describe node <node>
journalctl -u kubelet -n 100

# 常见原因
# - kubelet失联
# - 磁盘pressure
# - 内存pressure
# - network问题
```

#### 71.2 回答模板

> Node NotReady：检查kubelet日志和资源状态。常见磁盘/内存压力、网络问题。

---

### 72. Pod问题排查？

#### 72.1 CrashLoopBackOff

```bash
# 排查
kubectl logs <pod> --previous
kubectl describe pod <pod>

# 常见原因
# - 启动命令错误
# - 依赖服务不可用
# - healthcheck太早
# - OOM
```

#### 72.2 回答模板

> CrashLoopBackOff：logs + describe看原因。依赖失败、启动太慢、健康检查失败、OOM。

---

### 73. 网络问题排查？

#### 73.1 Pod连不上

```bash
# DNS解析
kubectl exec -it <pod> -- nslookup service

# 连通性
kubectl exec -it <pod> -- ping <pod-ip>

# 端口
kubectl exec -it <pod> -- telnet service:port
kubectl exec -it <pod> -- curl localhost:port
```

#### 73.2 回答模板

> 网络排查：DNS→连通性→端口。CoreDNS是常见问题。NetworkPolicy写错也是原因。

---

### 74. 存储挂载问题？

#### 74.1 PVC Pending

```bash
# 查看原因
kubectl describe pvc <pvc>

# 常见原因
# - StorageClass不存在
# - 存储不够
# - 亲和问题
```

#### 74.2 回答模板

> PVC Pending：describe查看Events。常见StorageClass不存在、存储不足、节点亲和。

---

### 75. 证书过期问题？

#### 75.1 证书检查

```bash
# 检查有效期
kubeadm alpha certs check-expiration

# 续期
kubeadm alpha certs renew all
```

#### 75.2 回答模板

> 证书一年过期。kubeadm alpha certs renew续期。或升级集群自动刷新。

---

### 76. K8S升级最佳实践？

#### 76.1 升级步骤

```plaintext
升级流程：
1. 阅读升级文档
2. 备份etcd
3. 测试环境验证
4. 先升级Master
5. 再升级Node
6. 验证功能
```

#### 76.2 回答模板

> K8S版本按N-1生产。备份etcd，小规模测试，逐节点升级，观察验证。

---

### 77. Docker Dockerfile最佳实践？

#### 77.1 安全实践

```dockerfile
# 非root
FROM alpine
RUN addgroup -S app && adduser -S app -G app
USER app

# 只读
COPY --chown=app:app . /app
USER app
```

#### 77.2 优化实践

```dockerfile
# 层数、缓存
# 多阶段
# .dockerignore
```

#### 77.3 回答模板

> 多阶段构建减少镜像，非root运行，只读文件系统，.dockerignore优化。

---

### 78. K8S应用健康检查设计？

#### 78.1 三类探针

```yaml
# liveness
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 3
  periodSeconds: 10

# readiness
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

# startup
startupProbe:
  httpGet:
    path: /start
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

#### 78.2 回答模板

> 探针设计核心：liveness保活、readiness引流、startup处理启动慢。initialDelaySeconds避免误判。

---

### 79. K8S命名空间设计？

#### 79.1 命名空间划分

```plaintext
Namespace划分：
- default      默认
- kube-system  系统
- dev         开发环境
- test        测试环境
- staging     预发布
- prod        生产环境
```

#### 79.2 回答模板

> NS按环境/业务划分。不同环境用不同Cluster或NS + ResourceQuota隔离。

---

### 80. 服务发现设计？

#### 80.1 DNS服务发现

```plaintext
K8S DNS：
- FQDN: <service>.<namespace>.svc.cluster.local
- 简写: <service>
- 跨namespace: <service>.<namespace>
```

#### 80.2 其他方案

```plaintext
服务发现方案：
- K8S DNS（默认）
- Headless Service + DNS
- Consul（外部服务）
```

#### 80.3 回答模板

> 服务发现用K8S DNS，FQDN命名规则。外部服务Consul或ExternalName Service。

---

### 81. Ingress Controller选型？

#### 81.1 主流选择

```plaintext
Ingress选型：
- Nginx Ingress（官方、功能全）
- Traefik（简单、支持多后端）
- 云ALB/ELB（厂商集成）
```

#### 81.2 对比

| 特性 | Nginx | Traefik |
|------|-------|--------|
| 性能 | 高 | 中 |
| 配置 | 复杂 | 简单 |
| 插件 | 多 | 中 |
| 维护 | 活跃 | 活跃 |

#### 81.3 回答模板

> Nginx Ingress功能最全最稳定。Traefik更简单。生产用云厂商ALB省运维。

---

### 82. 日志等级选择？

#### 82.1 日志级别

```plaintext
日志级别：
- DEBUG 调试
- INFO  正常
- WARN  警告
- ERROR 错误
```

#### 82.2 实践建议

```plaintext
日志实践：
- 生产INFO
- DEBUG仅开发
- 结构化JSON
- 采样ERROR
```

#### 82.3 回答模板

> K8S日志：JSON格式结构化。生产INFO级，异常ERROR。采样避免日志风暴。

---

### 83. 配置管理方案？

#### 83.1 配置方案

```plaintext
配置管理：
- ConfigMap（小配置）
- Secret（敏感）
- Vault（大/敏感）
- etcd+API（动态）
```

#### 83.2 实践

```plaintext
配置更新：
- ConfigMap热更新（可配置）
- Secret总是重启
- K8S 1.16+ ConfigMap热更新
```

#### 83.3 回答模板

> 配置管理：ConfigMap存放，敏感用Secret，大规模Secrets用Vault。注意热更新。

---

### 84. 安全加固 checklist？

#### 84.1 K8S安全

- [ ] RBAC最小权限
- [ ] 不用特权容器
- [ ] 只读根文件系统
- [ ] 禁止 privilege-escalation
- [ ] 网络策略严格
- [ ] PodSecurityPolicy/OPA
- [ ] 审计日志开启

#### 84.2 运行时安全

- [ ] 镜像签名验证
- [ ] 漏洞扫描
- [ ] Falco异常检测
- [ ] 密钥Vault管理

#### 84.3 回答模板

> 安全checklist：RBAC最小权限、禁止特权容器、只读文件系统、网络策略、审计日志、漏洞扫描.

---

### 85. Cost监控分析？

#### 85.1 监控工具

```plaintext
成本监控：
- Kubecost（Kubernetes成本）
- GCP Billing
- AWS Cost Explorer
```

#### 85.2 报告

```plaintext
成本维度：
- 按Namespace
- 按Label
- 按Service
- 按Pod
```

#### 85.3 回答模板

> 成本工具Kubecost。Tag(label)/Namespace分摊成本，优化资源request。

---

### 86. K8S与Serverless区别？

#### 86.1 架构对比

| 特性 | K8S | Serverless |
|------|-----|-------------|
| 资源 | 独占 | 共享 |
| 计费 | 资源 | 请求 |
| 启动 | 秒级 | 毫秒级 |
| 管理 | 完全 | 很少 |

#### 86.2 选型

```plaintext
场景选择：
- 常驻服务 → K8S
- 突发流量 → K8S+HPA
- 冷启动可接受 → Serverless
```

#### 86.3 回答模板

> K8S完全掌控Serverless更省心。按业务特点选型，可混合部署。

---

### 87. K8S与ServiceNow集成？

#### 86.1 ITSM集成

```plaintext
集成场景：
- 工单系统自动创建
- 告警接入ITSM
- CMDB同步
- 审批流程
```

#### 87.2 回答模板

> IT运营集成ITSM：告警生成工单、CMDB同步、资源变更审批。

---

### 88. K8S培训计划？

#### 88.1 培训路径

```plaintext
学习路径：
1. Docker基础（Dockerfile/docker-compose）
2. K8S概念（Pod/Service/Deployment）
3. 实操部署（单服务）
4. 网络存储（Volume/CNI）
5. 安全（Rbac/NetworkPolicy）
6. 运维（监控/日志/备份）
```

#### 88.2 回答模板

> K8S学习路线：Docker→K8S基础→部署实战→网络存储→安全加固→运维体系。

---

### 89. 持续学习资源？

#### 89.1 官方文档

```plaintext
K8S文档：
- https://kubernetes.io/zh/
- https://kubernetes.io/docs/tasks/
- https://kubernetes.io/docs/concepts/
```

#### 89.2 社区

```plaintext
学习资源：
- Kubernetes中国社区
- KubeSphere
- 云原生技术博客
- 极客时间K8S课程
```

#### 89.3 回答模板

> 资源：官方文档+KubeSphere+国内社区+实践。最有价值：官方Tutorial和CKA课程。

---

### 90. 生产问题速查表？

| 症状 | 可能原因 | 排查命令 |
|------|--------|----------|
| Pod创建卡住Pending | 资源不足/调度失败 | describe |
| Pod一直在 Waiting | 镜像拉取失败 | describe |
| Pod不断重启 | 应用启动失败/crash | logs --previous |
| Service无法访问 | Endpoint为空/discovery | describe svc |
| Pod无回应 | healthcheck/资源 | top/describe |
| Node NotReady | kubelet问题 | journalctl |

---

### 91. 企业落地关键因素？

#### 91.1 关键成功因素

```plaintext
K8S落地：
- 高层支持
- 团队学习成本
- 基础设施
- 运维经验
- 监控体系
- 变更流程
```

#### 91.2 回答模板

> 企业落地：基础设施先行，培训团队，建立SRE规范，渐进式迁移。

---

### 92. 容器化迁移流程？

#### 92.1 迁移步骤

```plaintext
迁移流程：
1. 评估现有应用
2. 容器化改造（Dockerfile）
3. K8S部署模板（Deployment）
4. 配置管理（ConfigMap/Secret）
5. 存储方案
6. 测试环境验证
7. 生产灰度
8. 监控观察
```

#### 92.2 回答模板

> 迁移流程：容器化→K8S部署→配置存储→测试→灰度→监控，一步步来。

---

### 93. 为什么用K8S?

#### 93.1 优势

```plaintext
K8S价值：
- 自动化运维
- 弹性伸缩
- 负载均衡
- 快速部署
- 版本回滚
- 可移植性
```

#### 93.2 回答模板

> K8S自动化运维、弹性伸缩、快速部署，企业数字化基座。迁移ROI显著。

---

### 94. K8S问题排查思路？

#### 94.1 排查框架

```
1. 确认问题
   kubectl get/status

2. 定位组件
   Events/Logs

3. 分析原因
   Resource/Network/Storage

4. 解决验证
   apply/test
```

#### 94.2 回答模板

> 排查思路：从表象到根本。Events最有用，logs次之，exec进容器验证。

---

### 95. 性能优化实践？

#### 95.1 网络优化

```plaintext
网络优化：
- CNI选型（Calico Cilium）
- Service会话保持
- DNS缓存
- Ingress优化
```

#### 95.2 存储优化

```plaintext
存储优化：
- SSD存储
- IOPS预留
- 读写分离
- 缓存策��
```

#### 95.3 回答模板

> 性能优化：CNI选型低延迟、存储用SSD、合理HPA阈值。网络是瓶颈。

---

### 96. 容量管理最佳实践？

#### 96.1 容量管理

```plaintext
容量管理：
- 按业务预估
- 峰值2倍buffer
- 持续监控
- 定期review
- HPA弹性
```

#### 96.2 回答模板

> 容量管理：预估+2倍buffer+持续监控+定期review+配合HPA弹性伸缩。

---

### 97. K8S故障恢复指南？

#### 97.1 故障级别

```plaintext
故障级别：
- P0: 集群全down
- P1: 单集群不可用
- P2: 部分Pod异常
- P3: 性能下降
```

#### 97.2 恢复步骤

```plaintext
恢复：
1. 确认故障范围
2. 启用应急预案
3. 止损恢复
4. 根因分析
5. 改进预防
```

#### 97.3 回答模板

> 故障分级：P0/P1立即止损，止血优先。复盘改进，定期演练。

---

### 98. K8S监控告警实践？

#### 98.1 核心告警

```plaintext
告警项：
- Node利用率>80%
- Pod重启>阈值
- Pod不ready
- etcd延迟
- API Server错误率
```

#### 98.2 回答模板

> 监控告警：资源使用、Pod状态、API延迟、核心组件。Golden Signal。

---

### 99. K8S稳定性SLO？

#### 99.1 常见SLO

```plaintext
SLO设定：
- 可用性 99.95%（4h 27m 月度停机）
- 延迟 P99 < 200ms
- 成功率 > 99.9%
```

#### 99.2 回答模板

> SLO：可用性99.9-99.95%，延迟P99指标。建立SLI→SLO→Error Budget。

---

### 100. 未来发展趋势？

#### 100.1 技术趋势

```plaintext
K8S趋势：
- Serveless（Knative）
- Edge K8S
- GitOps普及
- Policy-as-Code
- WASM runtime
- eBPF观测
```

#### 100.2 回答模板

> K8S方向：Serverless边缘计算、GitOps自动化、policy安全、eBPF观测。

---

## 附录：面试追问题

1. **K8S与Docker的区别？**
   答：Docker容器化，K8S编排管理。Docker打包，K8S部署调度。

2. **Pod内的容器如何通信？**
   答：localhost互访，共享网络命名空间。

3. **K8S如何实现自愈？**
   答：Controller检测Pod状态，异常自动重启重建。

4. **Service发现原理？**
   答：kube-proxy维护iptables/IPVS规则，DNS解析Service IP。

5. **为什么不用Docker Swarm？**
   答：K8S社区活跃、功能丰富、生态完善、生产级验证。

---

## 参考资料

- 《Kubernetes in Action》
- 《Docker — 容器云部署实战》
- Kubernetes官方文档
- CNCF云原生基金会

---

> 整理by Claude Code | Docker与K8S面试高频100问