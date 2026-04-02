在用 **Kafka 做并发处理** 时，幂等性（Idempotency）是个核心问题，本质是：**同一条消息被重复消费，也只会产生一次业务效果**。

下面我按工程实践给你拆清楚（偏 Golang / 分布式系统视角）：

---

# 一、为什么 Kafka 会出现重复消费？

即使你配置了：

- at-least-once（默认）
- 手动 commit offset

仍然会有重复，比如：

- consumer 处理成功，但 **offset 还没提交就挂了**
- rebalance 导致重复消费
- producer 重试导致重复写入

👉 所以：**Kafka 本身不保证业务幂等，只保证“至少一次”**

---

# 二、常见幂等方案（从简单到强一致）

## 1️⃣ 业务唯一 ID（最核心方案）

给每条消息一个 **全局唯一 ID（biz_id）**

比如：

```json
{
  "order_id": "123456",
  "event_id": "uuid-xxx"
}
```

消费时做：

```text
if event_id 已处理:
    忽略
else:
    执行业务
    记录 event_id
```

### 存储方式

- Redis（高性能）
- DB（强一致）
- 本地缓存（短期去重）

👉 这是最推荐、最通用方案

---

## 2️⃣ 数据库唯一约束（强一致）

直接利用数据库：

```sql
INSERT INTO orders (order_id, ...)
VALUES (...)
ON CONFLICT DO NOTHING;
```

或 MySQL：

```sql
INSERT IGNORE INTO ...
```

👉 优点：

- 天然幂等
- 不需要额外系统

👉 缺点：

- 写 DB 成本高
- 高并发下有锁竞争

---

## 3️⃣ Redis 去重（高性能）

用 Redis 做幂等控制：

```bash
SETNX event_id 1
```

Golang 示例：

```go
ok, _ := redis.SetNX(ctx, eventID, 1, time.Hour).Result()
if !ok {
    return // 已处理
}
```

👉 注意：

- 一定要设置 TTL（避免内存爆）
- 适合短期幂等（如 24h 内）

---

## 4️⃣ Kafka 事务（Exactly Once）

Kafka 提供：

- **Idempotent Producer**
- **Transactional Producer**
- **EOS（Exactly Once Semantics）**

配置：

```properties
enable.idempotence=true
transactional.id=xxx
```

👉 适用场景：

- Kafka → Kafka（流处理）
- 不适合 DB / 外部系统

👉 关键点：
Kafka EOS ≠ 业务幂等
只保证：

> “写 Kafka + 提交 offset”原子性

---

## 5️⃣ 消费+落库+提交 offset 原子化（推荐架构）

核心思想：

```text
处理数据 + 幂等记录 + 提交 offset = 一个事务
```

例如（MySQL）：

```sql
BEGIN;

-- 幂等检查
INSERT INTO processed_events(event_id) VALUES(...);

-- 业务操作
UPDATE account SET balance = balance - 100;

COMMIT;
```

👉 如果插入失败（重复），直接 rollback

---

## 6️⃣ 分区内顺序 + 单线程消费

Kafka 保证：

> 一个 partition 内是有序的

所以可以：

- 同一个 key（如 order_id）→ 同一 partition
- 单线程消费

👉 这样可以避免：

- 并发写冲突
- 状态错乱

---

# 三、工程实践推荐组合（很重要）

我给你一个现实可落地的组合 👇

### ⭐ 推荐方案（通用）

```
Kafka + Redis(SetNX) + DB
```

流程：

```text
1. 消费消息
2. Redis SETNX(event_id)
   - 失败 → 丢弃（重复）
3. 执行业务（DB）
4. 提交 offset
```

---

### ⭐ 强一致方案（金融级）

```
Kafka + DB 唯一约束 + 本地事务
```

流程：

```text
1. 开事务
2. 插入 event_id（唯一索引）
3. 执行业务
4. commit
5. 提交 offset
```

---

### ⭐ 高吞吐方案

```
Kafka + 分区路由 + 内存去重 + 批处理
```

---

# 四、Golang 实战建议

你做 Golang + Kafka，可以这样设计：

## Consumer 关键点

- 使用 **Sarama / franz-go**
- 手动 commit offset
- 控制并发：

```go
semaphore := make(chan struct{}, 100) // 限流

for msg := range messages {
    semaphore <- struct{}{}
    go func(m *sarama.ConsumerMessage) {
        defer func(){ <-semaphore }()
        handle(m)
    }(msg)
}
```

👉 但注意：

- 幂等必须在 handle 里保证
- 不要依赖 Kafka 保证

---

# 五、常见坑（面试高频）

### ❌ 误区 1：Kafka 可以保证幂等

👉 错，只保证“消息层面”

---

### ❌ 误区 2：只靠 offset

👉 offset 只是消费进度，不是业务状态

---

### ❌ 误区 3：用内存 map 去重

👉 重启就丢了

---

### ❌ 误区 4：Redis 不设 TTL

👉 内存爆炸

---

# 六、一句话总结

👉 Kafka 并发幂等的本质是：

> **“用唯一ID + 外部存储（Redis/DB）做去重，而不是依赖 Kafka 本身”**

---

如果你愿意，我可以帮你：

- 画一个 **Kafka 幂等架构图**
- 或给你一份 **完整 Golang demo（带 Redis + DB）**
- 或讲讲 **Kafka 如何做到“零丢失 + 幂等 + 顺序”三者平衡**
