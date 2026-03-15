**云原生（Cloud Native）**是一种**设计、构建和运行应用程序的方法**，专门为了在云环境中高效运行而设计。简单说：

> **云原生 = 利用云的能力（弹性、自动化、分布式）来开发和运行软件的架构理念。**

它不是单一技术，而是一整套**技术体系 + 架构思想 + 工程实践**。

---

## 一、云原生的核心思想

云原生的目标是让应用具有以下能力：

1. **弹性扩展（Elasticity）**
   流量变大 → 自动扩容
   流量变小 → 自动缩容

2. **高可用（High Availability）**
   单个实例挂了不会影响整体服务。

3. **快速交付（Fast Delivery）**
   开发 → 测试 → 部署可以自动化。

4. **可观测（Observability）**
   系统运行状态可以被监控、追踪、分析。

---

## 二、云原生的四大核心技术

### 1️⃣ 容器（Container）

把应用和依赖打包成一个镜像。

最常见工具：

- Docker

优点：

- 环境一致
- 快速部署
- 轻量隔离

例如：

```
App + runtime + dependencies → Docker image
```

---

### 2️⃣ 容器编排（Orchestration）

当你有 **几十到几千个容器**时，需要自动管理。

最核心系统：

- Kubernetes

Kubernetes可以：

- 自动部署
- 自动扩容
- 自动恢复
- 负载均衡
- 滚动更新

例如：

```
用户流量 ↑
Kubernetes 自动创建更多容器
```

---

### 3️⃣ 微服务架构（Microservices）

传统架构：

```
一个巨大应用
```

云原生架构：

```
订单服务
支付服务
用户服务
库存服务
```

每个服务：

- 独立部署
- 独立扩展
- 独立开发

---

### 4️⃣ DevOps & CI/CD

开发 → 自动部署

常见工具：

- Jenkins
- GitHub Actions
- Argo CD

流程：

```
code commit
   ↓
CI build
   ↓
test
   ↓
container image
   ↓
deploy to kubernetes
```

---

## 三、云原生技术栈（完整生态）

典型云原生架构：

```
Application
     │
Microservices
     │
Containers (Docker)
     │
Orchestration (Kubernetes)
     │
Service Mesh (Istio)
     │
Observability (Prometheus + Grafana)
     │
Cloud Infrastructure (AWS / GCP / Azure)
```

常见组件：

- Istio
- Prometheus
- Grafana

---

## 四、为什么企业都在做云原生？

因为它解决了传统系统的问题：

| 传统架构 | 云原生     |
| -------- | ---------- |
| 部署慢   | 自动化部署 |
| 扩容困难 | 自动扩容   |
| 单点故障 | 高可用     |
| 更新困难 | 滚动更新   |

典型使用公司：

- Netflix
- Uber
- Airbnb

---

## 五、一句话理解

**传统软件**

```
软件适配服务器
```

**云原生**

```
软件为云环境而设计
```

---

💡 如果你在学习 **系统设计 / Kubernetes / Golang 后端**（从你最近问的 k8s、docker、raft 看），我也可以给你讲：

- **云原生底层原理（容器 + cgroups + namespace）**
- **Kubernetes 架构（scheduler / etcd / controller）**
- **云原生系统设计面试八股**

这些是很多 **大厂分布式系统面试核心内容**。
