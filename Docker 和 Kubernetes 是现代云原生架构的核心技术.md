**Docker 和 Kubernetes** 是现代云原生架构的核心技术，但它们解决的问题不同：

- Docker：**容器运行时（Container Runtime）**，负责创建和运行容器
- Kubernetes：**容器编排系统（Container Orchestration）**，负责管理大量容器

下面从 **底层原理** 角度拆开讲。

---

# 一、Docker 的底层原理

Docker 本质上并不是虚拟机，而是 **Linux 容器技术的封装**。
它主要依赖 Linux 的几个核心能力：

### 1️⃣ Namespace（资源隔离）

Namespace 用来 **隔离进程视图**。

每个容器看到的系统都是独立的。

常见 namespace：

| Namespace | 作用         |
| --------- | ------------ |
| PID       | 进程隔离     |
| NET       | 网络隔离     |
| MNT       | 文件系统挂载 |
| IPC       | 进程通信     |
| UTS       | hostname     |
| USER      | 用户权限     |

例如：

容器内执行：

```
ps aux
```

只能看到 **容器自己的进程**。

实际上宿主机仍然能看到所有进程。

---

### 2️⃣ cgroups（资源限制）

cgroups 用于 **限制资源使用量**：

- CPU
- 内存
- IO
- 网络

例如：

```
docker run -m 512m nginx
```

就会在 cgroup 中限制该容器：

```
memory.limit = 512MB
```

这样容器不能用超过这个内存。

---

### 3️⃣ Union File System（分层文件系统）

Docker 镜像采用 **分层结构**。

例如：

```
Ubuntu base
  ↓
Python layer
  ↓
App layer
```

使用 **UnionFS / OverlayFS** 叠加。

优点：

- 镜像复用
- 下载更快
- 构建更快

Dockerfile：

```
FROM ubuntu
RUN apt install python
COPY app.py /
```

会生成 **三层镜像**。

---

### 4️⃣ Copy-on-Write（写时复制）

容器运行时：

```
Image (read only)
        ↓
Container layer (write)
```

容器修改文件时：

- 原镜像不变
- 新文件写入 container layer

这就是 **Copy-on-Write**。

---

### 5️⃣ Docker 架构

Docker 主要组件：

```
Docker CLI
    ↓
Docker Daemon (dockerd)
    ↓
containerd
    ↓
runc
    ↓
Linux kernel (namespace + cgroups)
```

说明：

- `dockerd` 管理容器
- `containerd` 管理生命周期
- `runc` 创建容器

---

# 二、Kubernetes 的底层原理

Kubernetes 是 **容器编排系统**。

核心解决问题：

- 自动部署
- 自动扩缩容
- 服务发现
- 自愈
- 负载均衡

---

# 三、Kubernetes 架构

K8s 由 **Control Plane + Worker Node** 组成。

### Control Plane（控制平面）

核心组件：

| 组件               | 作用         |
| ------------------ | ------------ |
| API Server         | 集群入口     |
| Scheduler          | 调度 Pod     |
| Controller Manager | 状态控制     |
| etcd               | 存储集群状态 |

架构：

```
        API Server
            |
     -----------------
     |       |       |
 Scheduler Controller etcd
```

---

### Worker Node

每个 Node 运行：

| 组件              | 作用     |
| ----------------- | -------- |
| kubelet           | 管理 Pod |
| kube-proxy        | 网络     |
| container runtime | 运行容器 |

结构：

```
Node
 ├─ kubelet
 ├─ kube-proxy
 └─ container runtime
```

---

# 四、Kubernetes 最核心的概念：Pod

Pod 是 **K8s 最小调度单位**。

一个 Pod 可以包含：

```
Pod
 ├─ container A
 └─ container B
```

特点：

- 共享 network
- 共享 volume
- 生命周期绑定

---

# 五、Kubernetes 调度流程

用户部署应用：

```
kubectl apply -f app.yaml
```

流程：

```
1 用户 → API Server
2 API Server → etcd 保存配置
3 Scheduler 选择 Node
4 kubelet 创建 Pod
5 container runtime 启动容器
```

完整流程：

```
User
  ↓
kubectl
  ↓
API Server
  ↓
etcd (存状态)
  ↓
Scheduler
  ↓
Node
  ↓
kubelet
  ↓
Docker/containerd
```

---

# 六、Kubernetes 自愈机制

K8s 的核心思想：

**Desired State（期望状态）**

例如：

```
replicas: 3
```

如果：

```
实际：2
```

Controller 会自动：

```
创建新的 Pod
```

恢复到：

```
3
```

---

# 七、Kubernetes 网络原理

K8s 要求：

**所有 Pod 互相可通信**

实现方案：

- Overlay Network
- CNI 插件

常见 CNI：

- Flannel
- Calico
- Cilium

网络结构：

```
Pod → veth pair → bridge → Node network → other Node → Pod
```

---

# 八、Service 负载均衡

K8s 使用 **Service** 实现服务发现。

例如：

```
service
  ↓
kube-proxy
  ↓
iptables / ipvs
  ↓
Pod
```

请求：

```
Client
   ↓
Service IP
   ↓
kube-proxy
   ↓
Pod1 / Pod2 / Pod3
```

---

# 九、Docker + Kubernetes 关系

关系非常简单：

```
Docker 负责：运行容器
Kubernetes 负责：管理容器
```

架构：

```
Kubernetes
     ↓
Container Runtime
     ↓
Docker / containerd
     ↓
Linux Kernel
```

现在很多 K8s 已经 **不直接使用 Docker**，而是：

- containerd
- CRI-O

---

# 十、一句话理解

Docker：

> **解决如何运行一个容器**

Kubernetes：

> **解决如何管理成千上万个容器**

---

如果你愿意，我可以再给你讲一个 **真正互联网公司级别的 Kubernetes 架构图（非常清晰）**，包括：

- Pod
- Deployment
- Service
- Ingress
- HPA
- CI/CD

很多工程师看完那张图 **10分钟就理解整个云原生体系**。
