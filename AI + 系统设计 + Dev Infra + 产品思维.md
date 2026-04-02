很好，这类岗位已经不是“普通后端”面试了，本质是 **AI + 系统设计 + Dev Infra + 产品思维** 的综合考察。下面我帮你把面试材料整理成一套可以直接用的体系（含讲稿思路 + 技术深挖点）。

---

# 一、30秒 / 3分钟自我介绍（直接可用）

### ✅ 30秒版本（电梯）

> 我是李承熙，主要方向是 Golang + 分布式系统 + AI Agent。过去几年我主要做高并发后端和基础设施，比如 Kafka、云原生、任务调度系统。
> 最近我重点在做 AI Agent 相关工作，包括 agent orchestration、工具调用、以及低延迟意图识别。
> 我对 AI coding agent 非常感兴趣，尤其是从“prompt → 自动代码 → CI → PR”的闭环系统，我也做过一些相关的架构设计和原型实践。

---

### ✅ 3分钟版本（推荐）

结构建议：**背景 → 能力 → AI Agent → why you**

> 我有 3+ 年后端经验，主要使用 Golang，专注于分布式系统和云原生架构，比如 Kafka、并发控制、任务系统这些。
>
> 在工程上我比较擅长：
>
> - 高并发系统设计（Kafka、幂等、分区并发）
> - 多 goroutine / actor-like 模型
> - 工具链集成（CI/CD、任务系统）
>
> 最近我把重心转向 AI Agent：
>
> - 做过 agent workflow 设计（多 agent 轮询 + 顺序保证）
> - 探索过低延迟 intent recognition（小模型 + routing）
> - 研究过 coding agent 的执行闭环（tool use + sandbox + code execution）
>
> 我对这个岗位感兴趣是因为：
>
> - 它不只是用 AI，而是**重构软件工程生产方式**
> - 涉及 agent runtime、devbox sandbox、CI integration，这些正好是我的优势区
>
> 我希望能把 AI agent 从“demo”做成“工程基础设施”。

---

# 二、核心项目讲法（必须准备）

你要准备一个“AI Coding Agent System Design”的项目（哪怕是你设计的，也可以）

---

## 🎯 项目：AI Coding Agent（重点讲这个）

### 1️⃣ 一句话定义

> 一个从自然语言需求 → 自动生成代码 → 跑测试 → 提交 PR 的端到端系统

---

## 2️⃣ 架构图（面试要能讲出来）

```
User Prompt
    ↓
Planner Agent（任务拆解）
    ↓
Executor Agent（代码生成 + tool use）
    ↓
Sandbox Devbox（隔离执行）
    ↓
CI Runner（test / lint）
    ↓
PR Generator（提交代码）
```

---

## 3️⃣ 关键模块拆解（重点）

### ✅ 1. Agent Orchestration（核心）

你可以这样说：

> 我会用一个 **planner-executor 架构**：

- Planner：拆任务（比如：修改 API + 写测试）
- Executor：逐步执行（调用工具）

👉 进阶说法：

- 用 **state machine / DAG workflow**
- 支持 retry / rollback

---

### ✅ 2. Devbox Sandbox（面试重点）

> 我会设计一个隔离执行环境，避免 agent 破坏主环境

方案：

- container（Docker / Firecracker）
- 限制：
  - CPU / 内存
  - 网络（只允许 git / package registry）

- ephemeral 环境（每次执行销毁）

👉 加分点：

- snapshot（加速启动）
- pre-baked image（提高 agent latency）

---

### ✅ 3. Tool / Skills System（超级重点）

> 我会把所有工具抽象为统一接口，让 agent 可调用

比如：

| Tool        | 功能    |
| ----------- | ------- |
| code_search | 搜代码  |
| file_edit   | 改文件  |
| run_test    | 跑测试  |
| create_pr   | 提交 PR |

👉 关键设计：

- JSON schema tool interface
- function calling / tool calling
- 权限控制（避免误操作）

---

### ✅ 4. Blueprint System（面试亮点🔥）

> 把常见任务抽象成 workflow template

例如：

#### Bug Fix Blueprint

```
1. 搜索相关代码
2. 复现 bug
3. 修改代码
4. 跑测试
5. 提交 PR
```

#### Feature Blueprint

```
1. 分析需求
2. 修改接口
3. 写测试
4. 更新文档
```

👉 面试加分说法：

> 类似 “Prompt → Workflow 编译器”

---

### ✅ 5. Evaluation / Metrics（面试必问）

你要主动说：

> 我会做 agent observability：

核心指标：

- 成功率（task success rate）
- CI 通过率
- PR 被 merge 率
- 平均执行时间
- retry 次数

👉 加分：

- trace（类似 OpenTelemetry）
- replay system（debug agent）

---

# 三、面试官一定会问的问题（提前准备）

---

## ❓ Q1：如何保证 agent 不乱改代码？

答：

- 限制 tool 权限（只允许某些目录）
- diff review（生成 patch 而不是直接 commit）
- 强制 CI gate
- human-in-the-loop（关键步骤）

---

## ❓ Q2：如何提高成功率？

答三点：

1. better planning（task decomposition）
2. better context（code search + embedding）
3. retry + reflection（self-correction）

---

## ❓ Q3：为什么需要 sandbox？

答：

> AI agent 本质是不可信执行体（untrusted code generator）

风险：

- rm -rf
- 数据泄露
- dependency 污染

---

## ❓ Q4：如何设计 memory？

答：

- short-term：当前任务 context
- long-term：
  - code embedding
  - 历史 PR
  - 成功案例

---

## ❓ Q5：多 agent 如何保证顺序？

结合你之前问题可以答：

> 用 event queue + version control

- 每个 step 有 version
- 使用 optimistic concurrency control
- 或 workflow engine（Temporal）

---

# 四、加分点（你可以主动提）

---

## 🚀 1. 低延迟 Agent

> 用小模型做 routing，大模型只做复杂任务

---

## 🚀 2. Agent 自我进化

> 用失败案例做 fine-tune / RAG

---

## 🚀 3. CI/CD 深度融合

> Agent 直接作为 CI participant

---

## 🚀 4. 类似这些工具（可以提）

- Claude Code
- Codex
- Cursor
- Goose

👉 但重点不是名字，是你说：

> 我会做 abstraction layer，避免 vendor lock-in

---

# 五、你可以反问面试官（非常关键）

问这些：

1. 你们的 agent 是偏“辅助开发”还是“全自动 PR”？
2. sandbox 是自己做还是用现成方案？
3. agent 成功率现在大概是多少？
4. 是否有统一的 tool schema？
5. 有没有 agent evaluation pipeline？

---

# 六、如果你想更强（我可以帮你继续）

我可以帮你下一步做这些：

✅ 模拟一轮真实面试（我当面试官）
✅ 帮你画一张“AI Coding Agent 架构图”（面试可直接画）
✅ 帮你写一个“高质量 system design 答案（像 Staff Engineer）”
✅ 或直接帮你 mock 一场 **1小时完整面试**

---

如果你只选一个下一步，我建议：

👉 **我们做一轮 mock interview（非常有用）**
