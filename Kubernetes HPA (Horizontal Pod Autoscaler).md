**Kubernetes HPA (Horizontal Pod Autoscaler)** 是 Kubernetes 中用于 **自动横向扩展 Pod 数量** 的机制。它会根据资源使用情况（CPU / Memory）或自定义指标，自动增加或减少 Pod 副本数，从而应对负载变化。⚙️📈

---

# 一、HPA 的核心作用

HPA 会持续监控应用指标，并动态调整 Deployment / StatefulSet 的 Pod 数量。

基本逻辑：

```
负载 ↑  → Pod 数量 ↑
负载 ↓  → Pod 数量 ↓
```

例如：

| CPU 使用率 | Pod 数量 |
| ---------- | -------- |
| 30%        | 2        |
| 70%        | 3        |
| 90%        | 5        |

这样可以实现：

- 自动扩容
- 自动缩容
- 节省资源
- 提升系统稳定性

---

# 二、HPA 工作原理

HPA 通过 **Metrics API** 获取指标：

```
Pod Metrics
   │
   ▼
Metrics Server
   │
   ▼
HPA Controller
   │
   ▼
Scale Deployment
```

组件：

| 组件           | 作用              |
| -------------- | ----------------- |
| metrics-server | 收集 CPU / Memory |
| HPA Controller | 计算是否扩容      |
| Deployment     | 被扩容对象        |

常见指标：

- CPU usage
- Memory usage
- QPS
- 自定义 metrics

---

# 三、HPA 自动扩容算法

核心公式：

```
desiredReplicas = ceil(
  currentReplicas × currentMetric / targetMetric
)
```

例如：

```
当前 Pod = 3
目标 CPU = 50%
当前 CPU = 80%

desiredReplicas = ceil(3 × 80 / 50)
                = ceil(4.8)
                = 5
```

HPA 会扩容到 **5 个 Pod**。

---

# 四、HPA 使用示例

## 1 安装 metrics-server

很多集群默认没有：

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

验证：

```bash
kubectl top nodes
kubectl top pods
```

---

## 2 创建 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
```

注意：

⚠️ HPA **必须设置 requests**。

---

## 3 创建 HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web

  minReplicas: 2
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

含义：

```
CPU 平均超过 50%
自动扩容
最大 10 pods
最小 2 pods
```

---

# 五、命令方式创建 HPA

也可以直接 CLI：

```bash
kubectl autoscale deployment web \
  --cpu-percent=50 \
  --min=2 \
  --max=10
```

查看：

```bash
kubectl get hpa
```

输出：

```
NAME      REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS
web-hpa   Deployment/web   60%/50%   2         10        4
```

---

# 六、HPA 支持的 Metrics

Kubernetes HPA v2 支持：

### 1 Resource metrics

最常用

```
CPU
Memory
```

---

### 2 Pod metrics

例如：

```
requests per second
```

---

### 3 Object metrics

例如：

```
Ingress QPS
```

---

### 4 External metrics

来自外部系统：

- Prometheus
- Kafka
- Redis
- CloudWatch

例如：

```
Kafka queue length
```

---

# 七、生产级 HPA 架构

很多公司会这样：

```
Prometheus
     │
     ▼
Prometheus Adapter
     │
     ▼
Custom Metrics API
     │
     ▼
HPA
```

常见 stack：

- Prometheus
- Grafana
- prometheus-adapter

可以基于：

```
QPS
Kafka lag
Queue length
```

自动扩容。

---

# 八、HPA + Cluster Autoscaler

很多人会混淆两个东西：

| 类型               | 扩容对象 |
| ------------------ | -------- |
| HPA                | Pod      |
| Cluster Autoscaler | Node     |

流程：

```
流量 ↑
  ↓
HPA 扩 Pod
  ↓
Node 不够
  ↓
Cluster Autoscaler 扩 Node
```

---

# 九、HPA 常见问题

### 1 不扩容

原因：

```
没有设置 resources.requests
```

---

### 2 metrics-server 没装

检查：

```
kubectl top pods
```

---

### 3 扩容慢

HPA 默认：

```
sync period = 15s
```

---

# 十、大厂实践

很多公司 HPA 触发条件：

| 触发指标     | 场景    |
| ------------ | ------- |
| CPU          | Web API |
| Memory       | Java    |
| QPS          | 网关    |
| Kafka lag    | 消费者  |
| Queue length | Worker  |

例如：

```
Kafka lag > 1000
→ 扩容 consumer pods
```

---

💡如果你做 **分布式 / GPU / AI 平台**（你之前问过 GPU scheduler），HPA 还可以结合：

- 推理 QPS
- GPU utilization
- 队列长度

做 **AI 推理自动扩容**。

---

如果你愿意，我可以继续讲一个 **高级面试级问题**：

**如何设计一个比 Kubernetes HPA 更智能的自动扩容系统（大厂架构）**
包括：

- 预测扩容（AI / ARIMA）
- 冷启动问题
- GPU 推理扩容
- 秒级扩容系统 🚀
