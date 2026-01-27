在 **Redis 中缓存热点链接（Hot URLs）**，核心目标是：**高并发下减轻数据库 / 后端服务压力，同时保证数据可用性与稳定性**。下面从**常见问题 → 缓存策略 → 具体实现方案**系统说明。

---

## 一、热点链接缓存面临的典型问题

1. **缓存穿透**

* 请求的 key 不存在，直接打到数据库
* 热点攻击时数据库容易被打爆

2. **缓存击穿**

* 某个热点 key 过期瞬间，大量并发请求同时访问 DB

3. **缓存雪崩**

* 大量 key 在同一时间过期，导致 DB 瞬时压力激增

4. **热点 Key 问题**

* 单个 key QPS 极高，Redis 单线程瓶颈
* 网络带宽 / CPU 成为瓶颈

---

## 二、缓存热点链接的核心策略（重点）

### 1️⃣ 热点 Key 永不过期 + 异步更新（强烈推荐）

**适用场景**：

* 短链接、文章详情页、商品页
* 数据允许“最终一致性”

**策略**

* Redis 中热点链接 **不设置过期时间**
* 后台定时或监听更新数据
* 防止击穿 & 雪崩

```text
key: hot:url:{id}
value: url_detail_json
TTL: 永久
```

**优点**

* 不会击穿
* QPS 极高也稳定

**缺点**

* 数据不是强一致

---

### 2️⃣ 逻辑过期（Logical Expire）⭐

**适用场景**：

* 高并发热点链接
* 需要自动刷新

**做法**

* value 中存储一个 `expireTime`
* Redis key 本身不过期

```json
{
  "data": {...},
  "expireTime": 1700000000
}
```

**读取逻辑**

```go
if now < expireTime {
    return data
}
try async refresh() // 只允许一个线程
return old data
```

**优势**

* 防击穿
* 无需加锁阻塞请求

---

### 3️⃣ 缓存击穿防护（互斥锁）

**适用场景**

* 热点 key 设置 TTL
* 数据必须准实时

```go
val := redis.Get(key)
if val == nil {
    if tryLock(key) {
        val = db.Get()
        redis.Set(key, val, ttl)
        unlock(key)
    } else {
        sleep(50ms)
        retry()
    }
}
```

**缺点**

* 高并发下线程阻塞
* 不如逻辑过期稳定

---

### 4️⃣ 本地缓存 + Redis（二级缓存）🔥

**适用**

* 极端热点（QPS 10W+）
* 单个链接被疯狂访问

```text
L1: Caffeine / Guava / sync.Map
L2: Redis
L3: DB
```

**流程**

1. 先查本地缓存（纳秒级）
2. 未命中查 Redis
3. Redis miss 才查 DB

**优势**

* 极大降低 Redis 压力
* 减少网络 IO

---

### 5️⃣ 热点 Key 拆分（Key Sharding）

**适用**

* 单 key QPS 超过 Redis 单线程瓶颈

```text
hot:url:123:1
hot:url:123:2
hot:url:123:3
```

**读**

* 随机 hash 取一个

**写**

* 同步更新多个 key

---

## 三、防穿透策略（热点链接必做）

### ✅ 1. 缓存空值

```text
key: hot:url:999
value: NULL
TTL: 30s
```

### ✅ 2. Bloom Filter（推荐）

* 请求前先判断是否存在
* 不存在直接返回

```text
BloomFilter.contains(urlId) == false → 拒绝
```

---

## 四、热点链接完整推荐方案（工业级）

### 🔥 推荐组合方案

> **本地缓存 + Redis 逻辑过期 + 异步刷新 + Bloom Filter**

```text
请求
 ↓
Bloom Filter
 ↓
本地缓存（L1）
 ↓
Redis（逻辑过期）
 ↓
DB（异步刷新）
```

### 适合场景

* 短链接服务
* 内容详情页
* 商品详情页
* 营销活动链接

---

## 五、Redis 配置与优化建议

* 使用 **Redis Cluster**
* `maxmemory-policy allkeys-lru`
* 热点 key 单独实例
* 监控：

  * QPS
  * 热 key
  * 命中率

---

## 六、示例：热点短链接缓存设计

```text
Key: short:url:{code}
Value:
{
  "originUrl": "...",
  "expireTime": 1700000000
}
TTL: -1
```

**访问流程**

1. 校验 Bloom
2. 本地缓存
3. Redis 返回
4. 若逻辑过期 → 异步刷新

---

## 七、总结（面试/实战金句）

> **真正的热点缓存不是“加 TTL”，而是“让它永远不失效，只在后台更新”。**

如果你愿意，我可以：

* 帮你设计 **短链接 Redis 热点缓存架构图**
* 给你一套 **Go + Redis 逻辑过期完整代码**
* 或针对 **面试题** 帮你总结标准答案
