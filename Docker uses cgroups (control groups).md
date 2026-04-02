Docker uses **cgroups (control groups)** — a core feature of the Linux kernel — to **limit, prioritize, and isolate resource usage** (CPU, memory, disk I/O, etc.) for containers.

Let’s break it down clearly 👇

---

# 🧠 What are cgroups?

**cgroups = Control Groups**

They allow the OS to:

- Limit resources (e.g., max memory)
- Measure usage (e.g., CPU consumed)
- Isolate workloads (so one container doesn’t starve others)

Think of it as:

> “Resource quotas + accounting for a group of processes”

---

# 🐳 How Docker uses cgroups

When you run a container via Docker, it:

1. Creates a **cgroup**
2. Assigns the container’s processes to it
3. Applies limits based on flags you provide

---

# ⚙️ Example: Limiting resources

### Memory limit

```bash
docker run -m 512m nginx
```

- Container can only use **512 MB RAM**
- If exceeded → OOM kill

---

### CPU limit

```bash
docker run --cpus="1.5" nginx
```

- Max CPU = 1.5 cores

---

### CPU shares (relative priority)

```bash
docker run --cpu-shares=512 nginx
```

- Default = 1024
- Lower = less priority under contention

---

# 📦 Under the hood (cgroup subsystems)

Docker configures different **controllers**:

| Controller | What it controls    |
| ---------- | ------------------- |
| `cpu`      | CPU time scheduling |
| `memory`   | RAM limits          |
| `blkio`    | Disk I/O            |
| `pids`     | Number of processes |
| `cpuset`   | Which CPU cores     |

Example path:

```
/sys/fs/cgroup/memory/docker/<container-id>/
```

---

# 🔄 cgroups v1 vs v2

### cgroups v1 (older)

- Separate hierarchies per controller
- More flexible but messy

### cgroups v2 (modern, default in many distros)

- Unified hierarchy
- Better resource accounting
- Simpler model

Docker supports both, but:

- New systems → **v2 preferred**

---

# ⚡ Key concept: Isolation ≠ Virtualization

- cgroups **limit resources**
- Namespaces **isolate environment (PID, network, etc.)**

Together → containers

---

# 🧩 Quick mental model

```
Container = Namespaces (isolation)
          + cgroups (resource control)
```

---

# 🚨 Common pitfalls

### 1. No limits = dangerous

If you don’t set limits:

- Container can consume **all host memory**
- Can crash the system

---

### 2. OOM killer behavior

- Kernel may kill your container
- Not always predictable which process dies

---

### 3. CPU shares misunderstood

- Only matters when CPU is contended
- Not a hard limit

---

# 🔍 Debugging cgroups

Check container limits:

```bash
docker inspect <container>
```

Check actual usage:

```bash
docker stats
```

Inspect cgroup directly:

```bash
cat /sys/fs/cgroup/memory/<id>/memory.limit_in_bytes
```

---

# 🧠 Summary

- cgroups = **resource control layer**
- Docker uses them to enforce **CPU, memory, I/O limits**
- Essential for **multi-tenant stability**
- Works with namespaces to form containers

---

If you want, I can go deeper into:

- How Kubernetes uses cgroups
- cgroups v2 internals (like `cpu.max`, `memory.high`)
- Real production tuning strategies (very useful for backend interviews)
