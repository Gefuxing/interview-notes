# CI/CD与DevOps 100问——持续集成/交付核心深度指南

> 本文档面向DevOps工程师，从入门到实战，包含高频面试题深度解析。
> 每个问题从「是什么」→「为什么」→「怎么实现的」「实际怎么用」四个维度讲解。

---

## 第一章 CI/CD基础（高频 ★★★★★）

### 1. 什么是CI？

#### 1.1 定义

> CI（Continuous Integration，持续集成）是一种软件开发实践，开发者频繁合并代码到主干，自动触发构建和测试。

```plaintext
CI核心要素：
- 频繁合并代码（每天多次）
- 自动化构建
- 自动化测试
- 问题早发现
```

#### 1.2 回答模板

> CI是频繁合并代码、自动构建测试的开发习惯。减少集成难度，快速发现问题。Jenkins/GitLab CI是常见工具。

---

### 2. 什么是CD？

#### 2.1 定义

> CD（Continuous Delivery，持续交付）在CI基础上，自动将代码部署到测试/预生产环境。

```plaintext
CD流程：
Code → Build → Test → Stage → Deploy to Staging
```

#### 2.2 Continuous Deployment

> 自动部署到生产（完全自动化）

```plaintext
CD完整流程：
Code → Build → Test → Stage → Production（自动）
```

#### 2.3 回答模板

> CD：持续交付自动到staging，持续部署自动到prod。CD是CI的延伸，实现一键部署。

---

### 3. CI/CD pipeline是什么？

#### 3.1 Pipeline定义

> Pipeline是定义代码从提交到部署全过程的工作流。

```groovy
// Jenkinsfile
pipeline {
    stages {
        stage('Build') {
            steps {
                sh 'make build'
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

#### 3.2 回答模板

> Pipeline定义构建→测试→部署流程。Jenkinsfile/GitLab CI YAML定义。是CI/CD的核心。

---

### 4. 为什么要CI/CD？

#### 4.1 价值

```plaintext
CI/CD价值：- 快速发布
- 减少人为错误
- 可追溯- 回滚快- 高质量
```

#### 4.2 答案模板

> CI/CD快速发布、减少手工错误、部署可追溯。传统手动部署风险高、慢、不一致。

---

### 5. 主流CI/CD工具？

#### 5.1 工具对比

| 工具 | 特点 | 费用 |
|------|------|------|
| Jenkins | 插件王、灵活 | 免费 |
| GitLab CI | 与Git集成、UI好 | 免费/付费 |
| Travis CI | GitHub集成 | 免费/付费 |
| CircleCI | 云原生 | 免费/付费 |
| Azure DevOps | 微软生态 | 免费/付费 |
| ArgoCD | GitOps | 免费 |

#### 5.2 答案模板

> Jenkins最灵活开源。GitLab CI仓库集成好。CircleCI云原生快。ArgoCD GitOps。

---

### 6. Jenkins基础？

#### 6.1 安装

```bash
# Docker
docker run -d -p 8080:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

# 或yum
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
yum install jenkins
systemctl start jenkins
```

#### 6.2 Jobs

```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```

#### 6.3 Answer template

> Jenkins：Jenkins安装，Job配置，Pipeline编写。插件丰富，企业主用。

---

### 7. GitLab CI基础？

#### 7.1 .gitlab-ci.yml

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - echo "Building..."
  artifacts:
    paths:
      - target/

test:
  stage: test
  script:
    - echo "Testing..."
  only:
    - main
    - develop
```

#### 7.2 Runner

```bash
# Register runner
gitlab-runner register \
  --url https://gitlab.com \
  --registration-token xxx \
  --executor docker \
  --docker-image docker:latest
```

#### 7.3 Answer template

> GitLab CI：.gitlab-ci.yml定义，GitLab Runner执行。Runner有shared/specific/groups三种。

---

### 8. Docker在CI中的应用？

#### 8.1 Docker化CI

```dockerfile
# Dockerfile
FROM maven:3.8 as builder
COPY . .
RUN mvn package -DskipTests

FROM openjdk:11-jre
COPY --from=builder /target/*.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```

#### 8.2 Docker in Pipeline

```yaml
build:
  stage: build
  image: maven:3.8
  script:
    - mvn package
  artifacts:
    paths:
      - target/*.jar
```

#### 8.3 Answer template

> CI中使用Docker：构建阶段Docker化，多阶段构建减少镜像。保证环境一致。

---

### 9. 单元测试与CI？

#### 9.1 单元测试集成

```yaml
test:unit:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/Coverage: \d+\.\d+%/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```

#### 9.2 Answer template

> 单元测试必须集成到CI。失败阻止后续。覆盖率阈值卡人。

---

### 10. 集成测试与CI？

#### 10.1 集成测试

```yaml
test:integration:
  stage: test
  services:
    - mysql:8.0
    - redis:6.0
  variables:
    MYSQL_ROOT_PASSWORD: root
    REDIS_HOST: redis
  script:
    - npm run test:integration
```

#### 10.2 Answer template

> 集成测试用services启动依赖mysql/redis。测试完整流程。测试环境与prod一致。

---

## 第二章 DevOps基础（高频 ★★★★★）

### 11. 什么是DevOps？

#### 11.1 定义

> DevOps是开发（Development）与运维（Operations）的桥梁，打破壁垒，实现快速高质量交付。

```plaintext
DevOps文化：
- 协作
- 自动化
- 监控
- 快速反馈
```

#### 11.2 答案模板

> DevOps消弭开发和运维鸿沟。通过自动化工具链，实现持续交付。核心是文化+工具。

---

### 12. DevOps成熟度模型？

#### 12.1 级别

```plaintext
DevOps成熟度：
Level 1: 手动、脚本
Level 2: 自动化Build
Level 3: 自动化Test+Deploy
Level 4: 持续交付
Level 5: 持续优化
```

#### 12.2 答案模板

> DevOps成熟度：手动→自动化→CI→CD→DevOps文化。越往右越快越好。

---

### 13. DevOps工具链？

#### 13.1 工具全景

```plaintext
DevOps工具链：
代码：Git
构建：Maven/Gradle/npm
测试：JUnit/Selenium
构建镜像：Docker
编排：K8S
配置：Ansible/Terraform
监控：Prometheus
日志：ELK
CI：Jenkins/GitLab
制品：Nexus/Artifactory
```

#### 13.2 Answer template

> DevOps工具覆盖全生命周期：代码→构建→测试→部署→监控→日志。工具链完整性决定效率。

---

### 14. GitOps是什么？

#### 14.1 定义

> GitOps以Git为唯一真相源，用声明式Infrastructure as Code管理部署。

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: https://github.com/me/repo
    path: k8s
  destination:
    server: https://kubernetes.io
    namespace: prod
```

#### 14.2 Answer template

> GitOps：配置文件Git管，ArgoCD/Tekton同步部署。版本可追溯，drift检测。

---

### 15. IaC基础设施即代码？

#### 15.1 IaC定义

> Infrastructure as Code用代码管理基础设施，保证一致性、可复制、可审计。

```hcl
# Terraform
resource "aws_instance" "web" {
  ami           = "ami-0c55b60c9afaefe61"
  instance_type = "t3.micro"
  tags = {
    Name = "web"
  }
}
```

#### 15.2 Answer template

> IaC：Terraform/Ansible/Vagrant。版本化管理基础设施。不可变基础设施。

---

### 16. Configuration Management？

#### 16.1 配置管理工具

```bash
# Ansible
ansible all -m setup
ansible all -b -m yum -a "name=nginx state=present"
ansible-playbook -i inventory site.yml
```

#### 16.2 Answer template

> Chef/Puppet/Ansible管理配置。Ansible无agentSSH，简单。幂等性好。

---

### 17. 制品库Artifacts？

#### 17.1 Nexus/Artifactory

```bash
# Maven deploy
mvn deploy

# Gradle
publish {
    maven {
        url "https://nexus.example.com/repository/maven-releases/"
        credentials {
            username = user
            password = password
        }
    }
}
```

#### 17.2 Answer template

> Nexus/ JFrog Artifactory：存放JAR/Docker镜像。版本管理，权限控制。

---

### 18. SemVer版本语义？

#### 18.1 定义

```plaintext
SemVer: MAJOR.MINOR.PATCH
- MAJOR: 不兼容API变更
- MINOR: 向后兼容功
- PATCH: 向后兼容补丁

预发布：1.0.0-alpha.1
构建：1.0.0-beta.1+build.123
```

#### 18.2 Answer template

> SemVer版本约定。MAJOR不兼容变更，MINOR新增功能，PATCH修复。自动化版本管理。

---

### 19. 蓝绿发布？

#### 19.1 蓝绿部署

```bash
# 流量切换
# Blue: old version
# Green: new version
# LB switching
```

#### 19.2 答案模板

> 蓝绿：两套环境，秒级切换。无停机，回滚快。成本翻倍。

---

### 20. 金丝雀发布？

#### 20.1 Canary Deploy

```bash
# 流量切小比例到新版本
# 观察
# 逐步放大
# 或Istio VirtualService
```

#### 20.2 Answer template

> 金丝雀：小流量先上，观察放大。风险小，适合关键业务。Argo Rollouts支持。

---

## 第三章 自动化构建测试（高频 ★★★★★）

### 21. 构建加速策略？

#### 21.1 缓存

```yaml
cache:
  paths:
    - .m2/repository
    - node_modules/
  key: ${CI_COMMIT_REF_SLUG}
```

#### 21.2 并行

```yaml
test:parallel:
  script:
    - npm run test:unit -- --parallel
    - npm run test:e2e -- --parallel
```

#### 21.3 Answer template

> 构建加速：依赖缓存、并行测试、增量构建、Docker层缓存。优化缩短CI时间。

---

### 22. 多阶段构建？

#### 22.1 Jenkins多阶段

```groovy
pipeline {
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

#### 22.2 Answer template

> 多阶段：Build→Test→Deploy。Stage门禁控制，失败停止。Branch条件。

---

### 23. 环境管理？

#### 23.1 环境类型

```plaintext
环境层级：
- Local/Dev   开发
- Test        测试
- Staging/QA  预发布- Prod         生产
```

#### 23.2 Answer template

> 环境：DEV/Test/Staging/Prod。DEV本地，Staging与Prod一致。环境隔离是基础。

---

### 24. 密码密钥管理？

#### 24.1 密钥注入

```yaml
# Jenkins credentials
withCredentials([string(credentialsId: 'db-password', variable: 'DB_PASSWORD')]) {
    sh 'run-db.sh'
}
```

#### 24.2 Vault

```bash
# HashiCorp Vault
export PASSWORD=$(vault kv get -field=password secret/db)
```

#### 24.3 Answer template

> 密钥不写代码-git。Jenkins credentials/Vault/云KMS。泄漏是大事。

---

### 25.  webhook触发？

#### 25.1 Webhook配置

```yaml
# GitLab webhook
- project: 'myproject'
  trigger:
    strategy: when_parent_status_success
```

#### 25.2 Answer template

> Webhook自动触发CI。Push/Merge触发。避免轮询浪费资源。

---

### 26. 分支策略？

#### 26.1 GitFlow

```plaintext
GitFlow分支：
- main/master   主干
- develop     开发- feature/*    功能分支- release/*    发布
- hotfix/*    热修复
```

#### 26.2 TrunkBased

```plaintext
Trunk Based：
- 直接commit trunk
- 短期分支hotfix
- 持续集成
```

#### 26.3 Answer template

> GitFlow适合正式发布项目，Trunk适合快速迭代。团队选适合的。

---

### 27. 合并策略Merge Request？

#### 27.1 MR流程

```bash
# Create MR
git push -o merge_request.create -o merge_request.target=main

# Code Review
# Auto CI
# Approve
# Merge
```

#### 27.2 Answer template

> MR.code Review+CI自动检查。通过后合并。保护main/trunk。

---

### 28. 代码审查Code Review？

#### 28.1 检查项

```markdown
代码审查清单：
- 功能完整
- 可读性
- 测试覆盖
- 性能影响
- 安全
- 日志
```

#### 28.2 Answer template

> MR必须有Reviewer。通过门槛：至少1人+CI通过+测试覆盖+安全扫描。

---

### 29. 自动化回滚？

#### 29.1 回滚脚本

```bash
# 回滚
./rollback.sh v1.0.0
# Jenkins
rollback: currentRevision(execute: false)
```

#### 29.2 Answer template

> CD必须带自动回滚能力。能快速恢复。RB上有价值的数据不回滚。

---

### 30. 制品签名？

#### 30.1 内容签名

```bash
# Cosign (Sigstore)
cosign sign --key cosign.key image:tag
cosign verify --key cosign.pub image:tag
```

#### 30.2 Answer template

> 镜像签名保证来源可信。Cosign/Notary。供应链安全必备。

---

## 第四章 容器化DevOps（高频 ★★★★★）

### 31. Docker化构建？

#### 31.1 多阶段构建

```dockerfile
# 构建镜像
FROM maven:3.8 AS builder
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src .
RUN mvn package -DskipTests

FROM openjdk:11-jre-slim
COPY --from=builder /target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

#### 31.2 Answer template

> 多阶段构建：builder编译，runtime运行。镜像小，安全漏洞少。

---

### 32. Docker镜像构建优化？

#### 32.1 Dockerfile优化

```dockerfile
# 优化
FROM alpine  # 基础镜像小
COPY . .    # 代码放最后
RUN rm -rf /var/cache   # 清理缓存
```

#### 32.2 Answer template

> 生产Dockerfile：alpine基础、分层优化、清理缓存。Dockerfile审查。

---

### 33. 私有镜像仓库？

#### 33.1 Harbor

```bash
# Docker Compose
curl -s -o docker-compose.yml https://raw.githubusercontent.com/goharbor/harbor/main/docker-compose.yml
docker-compose up -d
```

#### 33.2 Amazon ECR

```bash
# Push
aws ecr get-login-password | docker login --username AWS --password-stdin xxx.dkr.ecr.us-east-1.amazonaws.com
docker push xxx.dkr.ecr.us-east-1.amazonaws.com/repo:image
```

#### 33.3 Answer template

> 私有仓库：Harbor自建、ECR/ACR/GCR云托管。保证镜像安全。

---

### 34. K8s Deployent自动化？

#### 34.1 kubectl apply

```bash
kubectl apply -f deployment.yaml
kubectl set image deployment/myapp app=myapp:v2.0
kubectl rollout status deployment/myapp
```

#### 34.2 Answer template

> K8S部署：kubectl apply/update。Deployment管理Replicaset。Rolling update。

---

### 35. Helm Chart？

#### 35.1 Helm使用

```bash
# Install
helm install myapp ./chart

# Upgrade
helm upgrade myapp ./chart

# Rollback
helm rollback myapp 1
```

#### 35.2 Chart结构

```plaintext
chart/
├── Chart.yaml
├── values.yaml
└── templates/
```

#### 35.3 Answer template

> Helm Chart包管理。values.yaml配置参数。Helm是K8s包管理王者。

---

### 36. Kustomize？

#### 36.1 Kustomize结构

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
configMapGenerator:
  - name: app-config
    literals:
      - IMG=mynImage:v1.0
```

#### 36.2 Answer template

> Kustomize：Overlay方式。环境替换。不用模板语法，更简单。

---

### 37. Skaffold？

#### 37.1 Skaffold定义

```yaml
# skaffold.yaml
apiVersion: skaffold/v2
kind: Config
deploy:
  kubectl: {}
build:
  local: {}
profiles:
  - name: dev
    build:
      tagPolicy:
        imageStrategy:
          prefix: dev
```

#### 37.2 Answer template

> Skaffold：开发→上线自动化。本地watch改动，自动部署。快速迭代。

---

### 38. Kaniko构建镜像？

#### 38.1 Kaniko

```yaml
# Dockerfile
FROM gcr.io/kaniko-project/executor:latest
--context gcs://bucket/
--destination gcr.io/project/image:tag
```

#### 38.2 Answer template

> Kaniko：K8S内构建镜像。无需Docker daemon。安全。企业用。

---

### 39. BuildKit？

#### 39.1 Dockerfile

```bash
# 启用BuildKit
export DOCKER_BUILDKIT=1
docker build .
```

#### 39.2 Answer template

> BuildKit：并行构建、层缓存、产物压缩。多阶段构建优化。Docker daemon支持。

---

### 40. 镜像扫描？

#### 40.1 Trivy

```bash
trivy image myimage
trivy image --severity HIGH,CRITICAL myimage
```

#### 40.2 Clair

```yaml
# Clair in CI
-clair:
  image: quay.io/coreos/clair:latest
  arg:
    - -config-file=/etc/clair/config.yaml
```

#### 40.3 Answer template

> 镜像扫描Trapv/Clair/GRYPE。CI必须卡住高危漏洞。供应链安全。

---

## 第五章 监控与反馈（高频 ★★★★★）

### 41. 什么是SRE？

#### 41.1 SRE定义

> Site Reliability Engineering（网站可靠性工程）用软件工程思维解决运维问题。

```plaintext
SRE核心：
- 确定SLO
- 错误预算
- 做可靠性工作
- 拥抱风险
```

#### 41.2 答案模板

> SRE：用软件思维做运维。SLO衡量可靠性，SLI测量达成度。Google提出。

---

### 42. SLI/SLO/Error Budget？

#### 42.1 定义

```plaintext
SLI: Service Level Indicator  指标
  - 可用性
  - 延迟
  - 吞吐量

SLO: Service Level Objective 目标
  - 可用性>=99.95%

Error Budget: 错误预算
  - 0.05% 不可用的时间
```

#### 42.2 代码模板

> SLI是测量指标，SLO是目标，Error Budget是可接受故障时间。时间=100%-SLO%。

---

### 43. Golden Signals？

#### 43.1 四大黄金信号

```plaintext
Golden Signals：
- Latency 延迟
- Traffic 流量
- Errors 错误
- Saturation 饱和度
```

#### 43.2 答案模板

> 金牌信号：延迟/流量/错误/饱和度。监控这4个，SLI就够了。

---

### 44. 监控金字塔？

#### 44.1 层级

```
金字塔：
  用户体验
    ↓
  业务指标
    ↓
  应用指标
    ↓
  基础设施
```

#### 44.2 Answer template

> 监控从下往上：基础设施→应用→业务→用户体验。下面是根因，上面是症状。

---

### 45. Prometheus指标类型？

#### 45.1 类型

```plaintext
指标类型：
- Counter    累加
- Gauge     当前值
- Histogram 直方图
- Summary  摘要
```

#### 45.2 例子

```go
// Counter
httpRequestsTotal  // 请求总数

// Gauge
gcPauseDurationSeconds  // GC暂停

// Histogram
httpRequestDurationBucket
```

#### 45.3 Answer template

> Counter增长、Gauge波动、Histogram/Summary统计延迟。选错类型分析难。

---

### 46. Grafana仪表盘？

#### 46.1 Dashboard

```json
{
  "panels": [
    {
      "title": "CPU使用率",
      "targets": [
        {
          "expr": "rate(process_cpu_seconds_total{job=\"app\"}[5m])"
        }
      ]
    }
  ]
}
```

#### 46.2 Answer template

> Grafana：仪表盘可视化。模板变量。面板可复用。运维大屏必备。

---

### 47. AlertManager告警？

#### 47.1 定义

```yaml
groups:
- name: app
  rules:
  - alert: HighLatency
    expr: http_request_duration_seconds{quantile="0.99"} > 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High latency detected"
```

#### 47.2 Answer template

> 告警规则：Prometheus rule定义。AlertManager聚合去重发通知。抑制防风暴。

---

### 48. Logging最佳实践？

#### 48.1 结构化日志

```json
{
  "level": "info",
  "msg": "request processed",
  "trace_id": "abc123",
  "latency_ms": 25
}
```

#### 48.2 ELK Stack

```plaintext
ELK：
- Beats收集
- Logstash处理
- ES存储
- Kibana查
```

#### 48.3 Answer template

> 结构化JSON日志。ELK/Loki存储。便于搜索分析。日志即追踪。

---

### 49. 分布式追踪？

#### 49.1 Jaeger

```go
// tracer
tracer := opentracing.GlobalTracer()
span := tracer.StartSpan("operation")
defer span.Finish()
```

#### 49.2 协议

```plaintext
追踪协议：
- OpenTracing
- OpenTelemetry (统一)
```

#### 49.3 Answer template

> 分布式追踪：OpenTelemetry。链路追踪定位慢请求。上下游溯源。

---

### 50. 反馈环？

#### 50.1 反馈流程

```
Code → CI → Deploy → Monitor → Alert → Fix → Code
```

#### 50.2 Answer template

> DevOps反馈环越短越好。快速发现、快速响应。监控+告警是闭环。

---

## 第六章 自动化运维（高频 ★★★★★）

### 51. Ansible Playbook？

#### 51.1 Playbook

```yaml
- hosts: webservers
  become: yes
  vars:
    nginx_version: "1.24"
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

#### 51.2 Answer template

> Ansible Playbook：YAML声明配置。幂等执行。简单强大。

---

### 52. Terraform工作流？

```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

#### 52.2 Answer template

> Terraform：init→plan→apply→destroy。state管理状态。lock防并发。

---

### 53.  IaC安全实践？

#### 53.1 代码扫描

```bash
# tfsec
tfsec .

# Checkov
checkov -d .

# Snyk
snyk iac test .
```

#### 53.2 Answer template

> IaC 安全：tfsec/checkov扫描。CI中Gate。基础设施也要安全审计。

---

### 54. 密钥管理实践？

#### 54.1 Vault

```bash
# 读取
vault kv get secret/db

# 使用
VAULT_TOKEN="$(vault token create -field=token -policy=my-policy)"
```

#### 54.2 Answer template

> 密钥集中Vault管理。动态凭据。审计日志。泄漏可轮循。

---

### 55. GitOps工具ArgoCD？

#### 55.1 ArgoCD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argo-cd
    path: guestbook
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

#### 55.2 Answer template

> ArgoCD：GitOps CRD。自动同步。Prune清理垃圾。SelfHeal自愈。

---

### 56. Argo Rollouts？

#### 56.1 Canary

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: guestbook
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 10m}
      - setWeight: 30
      - pause: {duration: 10m}
      - setWeight: 100
```

#### 56.2 Answer template

> Argo Rollouts：金丝雀/蓝绿/实验。自动化分析和进度。替代Deployment。

---

### 57. Tekton？

#### 57.1 Tasks

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build
spec:
  steps:
  - name: build
    image: maven:3.8
    script: |
      mvn clean package
```

#### 57.2 Pipelines

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: pipeline
spec:
  tasks:
  - name: build
    taskRef:
      name: build
  - name: deploy
    taskRef:
      name: deploy
```

#### 57.3 Answer template

> Tekton：K8S native CI。Task定义步骤。Pipeline串联。GitOps友好。

---

### 58. Jenkins Shared Libraries？

#### 58.1 定义

```groovy
// vars/buildMaven.groovy
def call() {
    sh 'mvn clean package'
}
```

```groovy
// Jenkinsfile
@Library('shared-library') _
buildMaven()
```

#### 58.2 Answer template

> Shared Libraries：复用Pipeline代码。版本化管理。企业必备。

---

### 59. Jenkins Agent？

#### 59.1 分布式构建

```groovy
// Agent标签
pipeline {
    agent { label 'docker' }
    stages {
        stage('Build') {
            steps {
                sh 'docker build .'
            }
        }
    }
}
```

#### 59.2 Answer template

> Jenkins Agent：Master/Slave架构。标签分类执行。K8S agent动态伸缩。

---

### 60.  SonarQube代码审查？

#### 60.1 Scanner

```bash
# Maven
mvn sonar:sonar

# Gradle
gradle sonarqube
```

#### 60.2 Quality Gates

```plaintext
SonarQube质量门：
- Bug 0
- Vulnerabilities 0
- Coverage >=80%
- Duplication <3%
```

#### 60.3 Answer template

> SonarQube：代码静态分析。质量问题CI卡住。质量门禁。

---

## 第七章 安全DevOps（高频 ★★★★★）

### 61. DevSecOps？

#### 61.1 定义

> 把安全融入DevOps各阶段。

```plaintext
DevSecOps阶段：
- 代码安全 SAST
- 镜像安全
- 运行时安全
- 供应链安全
```

#### 61.2 Answer template

> DevSecOps：安全。左移早期发现。Security as Code。安全自动化。

---

### 62. SAST？

#### 62.1 静态分析

```bash
# SonarQube
mvn sonar:sonar

# Semgrep
semgrep --lang go --config auto .
```

#### 62.2 Answer template

> SAST：SonarQube/Semgrep/Checkmarks。CI Gate卡Bug。

---

### 63. DAST？

#### 63.1 动态测试

```bash
# OWASP ZAP
zap-baseline.py -t https://example.com

# Nuclei
nuclei -u https://example.com
```

#### 63.2 Answer template

> DAST：ZAP/Nuclei爬虫。运行时注入测试。OWASP Top 10卡。

---

### 64. SCA依赖扫描？

#### 64.1 工具

```bash
# npm audit
npm audit

# Snyk
snyk test

# GitHub Dependabot
```

#### 64.2 Answer template

> SCA：检查依赖漏洞。NPM audit/Snyk。CI必须跑。

---

### 65. 容器安全最佳实践？

#### 65.1 Dockerfile

```dockerfile
# 非ROOT
RUN addgroup -S app && adduser -S app -G app
USER app

# 只读
READONLYRootFilesystem TRUE

# 不许特权
privileged: FALSE
```

#### 65.2 Answer template

> 容器安全：非root运行、只读fs、禁止特权、基本镜像扫描。

---

### 66. Kubernetes安全？

#### 66.1 RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

#### 66.2 NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

#### 66.3 Answer template

> K8S安全：RBAC最小权限、NetworkPolicy隔离、PodSecurityPolicy。

---

### 67. Secrets管理K8S？

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

#### 67.2 Answer template

> K8S Secret：base64编码。云厂商KMS外部存。生产用Vault。

---

### 68. 审计日志？

#### 68.1 K8S Audit

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
- --audit-log-flags
- --audit-policy-file=/etc/kubernetes/audit-policy.yaml
```

#### 68.2 Answer template

> Audit记录所有API请求。日志存ES长期保留。合规必需。

---

### 69. OPA Gatekeeper？

#### 69.1 定义

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-labels
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Namespace"]
  parameters:
    labels:
    - key: environment
```

#### 69.2 Answer template

> OPA Gatekeeper：策略即代码。约束验证。合规自动化。

---

### 70. 供应链安全SLSA？

#### 70.1 Levels

```plaintext
SLSA级别：
1. 手工记录
2. 托管构建+源码
3. 安全的构建流程
4. 密封构建+标签
```

#### 70.2 Answer template

> SLSA：Supply-chain Levels for Artifacts。供应链安全框架。Cosign做签名。

---

## 第八章 云原生DevOps（中高 ★★★★）

### 71. 云原生CI/CD？

#### 71.1 定义

```plaintext
云原生CI/CD：
- K8S Platform
- Container Native
- Microservice Oriented
- Auto Scaling
```

#### 71.2 Answer template

> 云原生：K8S构建和运行。无服务器Serverless。函数即服务Faas。

---

### 72. Serverless部署？

#### 72.1 AWS Lambda

```yaml
# serverless.yml
service: myapp
provider:
  name: aws
  runtime: nodejs18.x
functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
```

#### 72.2 Answer template

> Serverless Framework：一键部署Lambda/Function。自动扩缩。按调用计费。

---

### 73. Service Mesh？

#### 73.1 Istio

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        x-canary:
          exact: "true"
    route:
    - destination:
        host: myapp-v2
      weight: 10
```

#### 73.2 Answer template

> Istio：灰度发布、金丝雀、熔断。流量精细控制。Sidecar透明。

---

### 74. 事件驱动架构？

#### 74.1 Event Driven

```plaintext
事件驱动：
- Event Producer -> Event -> Consumer
- At-least-once delivery
- Idempotent processing
- Event sourcing
```

#### 74.2 Answer template

> 事件驱动：解耦。CloudEvents规范。异步处理。Kafka是事实标准。

---

### 75. Chaos Engineering？

#### 75.1 Chaos Monkey

```bash
# Litmus Chaos
kubectl apply -f https://hub.litmuschaos.io/yaml/master/experiments/cases/pod-kill/pod-kill.yaml
```

#### 75.2 Answer template

> Chaos Engineering混沌：主动故障注入。验证韧性。生产必须试。

---

### 76. Feature Flags？

#### 76.1 定义

```javascript
// LaunchDarkly
if (ldclient.boolVariation("new-feature")) {
  // 10% traffic
}
```

#### 76.2 Answer template

> Feature Flags：热开关。灰度发布。Kill Switch。无需重新部署。

---

### 77. A/B Testing？

#### 77.1 流量分配

```yaml
# Istio
- route:
  - destination:
      host: myapp-v1
    weight: 80
  - destination:
      host: myapp-v2
    weight: 20
```

#### 77.2 Answer template

> AB Testing：对比不同版本效果。流量分配。金丝雀类似。

---

### 78. 渐进式交付？

#### 78.1 Flagger

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: app
spec:
  targetRef:
    APIVersion: apps/v1
    Kind: Deployment
    Name: app
  progressDeadline:
  AnalysisInterval: 1m
  maxWeight: 50
  steps:
  - setWeight: 10
  - pause: {duration: 10m}
  - setWeight: 30
```

#### 78.2 Answer template

> Flagger：Argo Flagger进渐式交付。自动分析和回滚。比Argo Rollouts易用。

---

### 79. 多云部署？

#### 79.1 Cross Cloud

```yaml
# Terraform
provider "aws" {
  region = "us-east-1"
}
provider "azurerm" {
  location = "eastus"
}
```

#### 79.2 Answer template

> 多云部署：Terraform抽象层。降低厂商锁定。但复杂性高。

---

### 80. GitOps多集群？

#### 80.1 多集群管理

```yaml
# Argo CD ApplicationSet
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: clusters
spec:
  generator:
    clusters:
      selector:
        env: prod
  template:
    spec:
      source:
        repoURL: https://github.com/me/repo
        path: k8s/{{name}}
      destination:
        server: '{{server}}'
```

#### 80.2 Answer template

> GitOps多集群：ArgoCD ApplicationSet。集群Template。自动部署。

---

## 第九章 运维最佳实践（中高 ★★★★）

### 81. 部署频率vs变更前置时间？

#### 81.1 DORA指标

```plaintext
DORA四指标：
- 部署频率   高
- 变更前置时间 短
- 变更失败率 低
- MTTR  MTTR恢复时间短
```

#### 81.2 Answer template

> DORA指标：DevOpsResearchAndAssessment。DevOps能力对标。

---

### 82. 部署流水线实践？

#### 82.1 最佳实践

```plaintext
最佳实践：
- 分阶段 gate
- 快速失败
- 环境一致性- 并行执行
- 制品Binaries
- 幂等
```

#### 82.2 Answer template

> Pipeline最佳实践：快速失败、环境一致、二进制产物。质量内建。

---

### 83. 变更管理流程？

#### 83.1 标准流程

```plaintext
变更流程：
1. Review  同行评审
2. Test    自动化测试
3. Stage  预发布验证
4. Deploy 灰度/蓝绿
5. Monitor 监控观察
6. Rollback 预案
```

#### 83.2 Answer template

> 变更必须有Review。灰度发布。出现问题回滚。监控观察。

---

### 84. 可观测性实践？

#### 84.1 三大支柱

```plaintext
可观测性：
- Metrics   指标
- Logs     日志
- Traces   链路
```

#### 84.2 Answer template

> 可观测性：指标+日志+链路。OpenTelemetry统一。问题快速定位。

---

### 85. 事件响应流程？

#### 85.1 事件级别

```plaintext
事件级别：
- SEV0  全网影响
- SEV1  重大影响
- SEV2  部分影响
- SEV3  小影响
```

#### 85.2 Answer template

> 事件分级：P0/P1紧急止血。事后复盘。SLA承诺。

---

### 86. MTTR优化？

#### 86.1 恢复流程

```
发现 → 确认 → 止血 → 恢复 → 复盘
```
```plaintext
MTTR优化：
- 监控告警快- 应急手册- 自动化回滚
- 演练
```

#### 86.2 Answer template

> MTTR优化：监控覆盖、自动化回滚、故障演练。止血优先。

---

### 87. 能力成熟度评估？

#### 87.1 CALMS模型

```plaintext
CALMS：
- Culture  文化
- Automation 自动化
- Lean     精益
- Measure  测量
- Sharing 分享
```

#### 87.2 Answer template

> CALMS：DevOps评估框架。文化根基。度量改进。

---

### 88. 文档即代码？

#### 88.1 Doc as Code

```markdown
docs/
├── README.md
├── architecture.md
├── ops/
│   └── runbooks/
└── incidents/
```

#### 88.2 Answer template

> 文档Git管。Runbook操作手册。事故复盘文档。知识沉淀。

---

### 89. 培训与游戏化？

#### 89.1 Learning

```plaintext
学习方式：
- 内部培训
- War Game
- kata练习
- CERT
```

#### 89.2 Answer template

> 团队学习：内部培训、Game Days战争游戏。玩中学。不只是培训。

---

### 90. 持续改进？

#### 90.1 回顾会议

```markdown
# Retrospective
## What went well?
## What didn't?
## Action items
```

#### 90.2 Answer template

> 持续改进：Retro回顾。Action Items落实。度量持续。

---

## 第十章 实战综合（高级 ★★★）

### 91. 公司CI/CD落地案例？

#### 91.1 实施路径

```plaintext
落地步骤：
1. 现状评估
2. 目标设定
3. 工具选型
4. 小范围试点
5. 逐步推广
6. 持续优化
```

#### 91.2 Answer template

> CI/CD落地：小范围开始。成功案例宣传。全公司推广。持续优化。

---

### 92. 创业公司DevOps选型？

#### 92.1 推荐方案

```plaintext
Startup stack：
- GitHub/GitLab
- GitHub Actions/GitLab CI
- Vercel/Netlify
- Railway/Render
- Railway/Render
```

#### 92.2 Answer template

> 创业:GitHub+Actions,Vercel部署起步。少维护，快速迭代。

---

### 93. 大企业DevOps选型？

#### 93.1 Enterprise

```plaintext
Enterprise Stack：
- GitLab EE
- Jenkins Enterprise
- Artifactory
- Jira+Confluence
```

#### 93.2 Answer template

> 大企业：安全、合规、审计。GitLab EE+Jenkins EE+Artifactory。

---

### 94. 银行金融DevOps实践？

#### 94.1 监管要求

```plaintext
金融要求：
- 审计追溯- 权限分离- 变更审批- 合规检查
```

#### 94.2 Answer template

> 金融行业：强监管。审计日志、权限分离、变更审批。双签。

---

### 95. 互联网公司实践？

#### 95.1 快节奏

```plaintext
互联网玩法：
- 快速迭代
- 小步快跑
- 灰度发布
- A/B Test
```

#### 95.2 Answer template

> 互联网：日级发布。Feature Flag。金丝雀。大规模自动化。

---

### 96. 政企单位DevOps？

#### 96.1 政务

```plaintext
政务特点：
- 安全可控
- 国产化
- 等级保护
- 内外网分离
```

#### 96.2 Answer template

> 政务：信创国产化。等级保护。内网部署。合规严。

---

### 97. Kubernetes GitOps？

#### 97.1 方案

```plaintext
K8S GitOps：
- Code → Git
- ArgoCD Sync
- Drift Detection
- Auto Heal
```

#### 97.2 Answer template

> K8S GitOps：ArgoCD/Tekton同步。Drift自动检测。Self-Healing自愈。

---

### 98. 成本优化策略？

#### 98.1 成本

```plaintext
成本优化：
- Right sizing
- Spot instances
- Reserved
- Auto scaling
```

#### 98.2 Answer template

> DevOps成本：Right Sizing资源。Spot节省70%。非prod少开。

---

### 99. 合规与审计？

#### 99.1compliance

```plaintext
合规：
- SOX
- PCI-DSS
- HIPAA
- GDPR
```

#### 99.2 Answer template

> 合规审计：代码变更可追溯。日志保留。RBAC权限。SOC2。

---

### 100. 未来DevOps趋势？

#### 100.1 trends

```plaintext
DevOps趋势：
- Platform Engineering
- Internal Developer Portal
- IDP内部开发者平台
- AI辅助运维
- FinOps成本
```

#### 100.2 Answer template

> DevOps趋势：平台工程、IDP、AI辅助、FinOps。开发者体验第一。

---

## 附录：常见面试题

1. **介绍一下公司的CI/CD流程？**
   答：从代码提交讲起，描述完整流水线

2. **CI/CD中如何保证安全性？**
   答：代码扫描+容器扫描+密钥管理+Rbac

3. **如何处理部署失败？**
   答：自动回滚+告警+日志定位

4. **你们用什么工具做配置管理？**
   答：Ansible/Terraform/Helm

5. **如何实现蓝绿部署？**
   答：两组环境+LB切换

6. **监控哪些指标？**
   答：基础设施+应用+业务+DORA四指标

---

## 参考资料

- 《The DevOps Handbook》
- 《Accelerate》
- 《Site Reliability Engineering》
- Google SRE Book

---

> 整理by Claude Code | CI/CD与DevOps面试高频100问