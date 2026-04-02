Kubernetes **Server-Side Apply (SSA)** is great for multi-actor workflows (controllers + humans), but conflicts happen when **two field managers try to own the same field**.

Let’s break down _why conflicts happen_ and the **practical strategies to resolve them cleanly**.

---

# 🔥 Why SSA Conflicts Happen

SSA tracks **field ownership** in `metadata.managedFields`.

- Each actor (controller, kubectl, CI/CD, etc.) is a **field manager**
- Each field (e.g., `spec.replicas`) has **one owner**

👉 Conflict occurs when:

```
Controller owns spec.replicas
Developer tries to apply spec.replicas
```

Kubernetes says:

> “You’re modifying a field you don’t own.”

---

# 🧠 Key Principle

👉 **You must define clear ownership boundaries**

Without that, SSA becomes a constant conflict generator.

---

# ✅ Strategy 1: Split Ownership by Fields (Best Practice)

Design your resources so:

- 👨‍💻 Developers own:
  - `spec.template`
  - `metadata.labels`

- 🤖 Controllers own:
  - `status`
  - specific spec fields (e.g., `replicas`, autoscaling)

### Example

```yaml
# developer applies
spec:
  template:
    spec:
      containers:
        - name: app
          image: myapp:v2
```

Controller manages:

```yaml
spec:
  replicas: 5
```

👉 No overlap = no conflict

---

# ✅ Strategy 2: Use Different Field Managers

Always apply with explicit managers:

```bash
kubectl apply --server-side --field-manager=developer -f deploy.yaml
```

Controller uses:

```go
ApplyOptions{
  FieldManager: "my-controller",
}
```

👉 This makes ownership visible and debuggable.

---

# ✅ Strategy 3: Force Apply (Use Carefully)

If developer _must override controller_:

```bash
kubectl apply --server-side --force-conflicts -f deploy.yaml
```

👉 This:

- steals ownership
- overwrites controller changes

⚠️ Risk:

- Controller may revert it back → “fight loop”

---

# ✅ Strategy 4: Controllers Should Avoid Owning User Fields

Controller design rule:

> ❌ Don’t own fields users are expected to modify

Instead:

- Use **defaults only on create**
- Avoid re-applying full spec

### Bad controller behavior:

```go
Apply(fullDeploymentSpec)
```

### Good behavior:

```go
Apply(only fields controller owns)
```

---

# ✅ Strategy 5: Use Subresources (Very Important)

Controllers should write to:

```
/status
```

NOT:

```
/spec
```

Example:

```bash
kubectl patch deployment myapp --subresource=status
```

👉 Prevents spec conflicts entirely.

---

# ✅ Strategy 6: Use Patch Instead of Apply in Controllers

For controllers:

- Prefer:
  - `PATCH` (merge/strategic)

- Avoid:
  - full `Apply` unless necessary

👉 Reason:
Apply = claims ownership
Patch = modifies without ownership takeover

---

# 🔍 Debugging Conflicts

Check ownership:

```bash
kubectl get deploy myapp -o yaml
```

Look at:

```yaml
metadata:
  managedFields:
```

You’ll see:

```yaml
- manager: developer
  fields:
    f:spec:
      f:template: {}
- manager: my-controller
  fields:
    f:spec:
      f:replicas: {}
```

👉 That tells you exactly who owns what.

---

# 🧩 Real-World Pattern (Golden Setup)

| Field                  | Owner            |
| ---------------------- | ---------------- |
| `spec.template`        | Developer        |
| `spec.replicas`        | HPA / Controller |
| `status`               | Controller       |
| annotations (internal) | Controller       |
| labels (business)      | Developer        |

---

# 🚨 Anti-Patterns (Avoid These)

❌ Controller applies full object every reconcile
❌ Developers using `force-conflicts` regularly
❌ No field-manager specified
❌ Shared ownership of same field

---

# 💡 Advanced Pattern: “Intent vs Observed State”

Split:

- **User intent** → `spec.userConfig`
- **Controller output** → `spec.computedConfig`

👉 Avoids direct collision entirely.

---

# 🧭 TL;DR

If you want SSA to work smoothly:

1. **Define strict field ownership**
2. **Use different field managers**
3. **Controllers should not own user-editable fields**
4. **Use `/status` and patch where possible**
5. **Only use `force-conflicts` as last resort**

---

If you want, I can show a **real controller-runtime (Go) example** with SSA done correctly — that’s where most people get it wrong.
