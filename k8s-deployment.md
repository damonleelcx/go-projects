Below is a **practical, production-style guide** to deploying Kubernetes for a distributed system **without Minikube**. This is how Kubernetes is commonly deployed in real environments (cloud, bare-metal, or VMs).

---

## 1️⃣ Choose Your Deployment Model (Very Important)

Since you’re not using Minikube, you’ll deploy a **real Kubernetes cluster**. The main options:

### Option A — Managed Kubernetes (Recommended)

You **don’t install Kubernetes yourself**; the provider manages control plane.

| Provider     | Service |
| ------------ | ------- |
| AWS          | EKS     |
| GCP          | GKE     |
| Azure        | AKS     |
| DigitalOcean | DOKS    |

✅ Best for reliability, scaling, security
❌ Less low-level control

---

### Option B — Self-Managed on VMs or Bare Metal

You install Kubernetes manually.

| Tool        | Use Case                   |
| ----------- | -------------------------- |
| **kubeadm** | Standard, production-grade |
| k3s         | Lightweight clusters       |
| Rancher     | Enterprise management      |

✅ Full control
❌ More operational responsibility

---

## 2️⃣ Architecture of a Real Distributed Kubernetes Cluster

```
                ┌─────────────────┐
                │  Control Plane  │
                │ (API, Scheduler)│
                └─────────────────┘
                        │
     ┌──────────────────┼──────────────────┐
     │                  │                  │
┌───────────┐   ┌───────────┐     ┌───────────┐
│ Worker 1  │   │ Worker 2  │ ... │ Worker N  │
│  Pods     │   │  Pods     │     │  Pods     │
└───────────┘   └───────────┘     └───────────┘
```

* **Control plane** manages cluster state
* **Workers** run distributed services (microservices, jobs)

---

## 3️⃣ Deploy Kubernetes Using `kubeadm` (Most Common Non-Minikube Setup)

### Step 1: Prepare All Nodes

On **every VM or machine**:

```bash
sudo swapoff -a
sudo apt update
sudo apt install -y containerd
```

Enable container runtime:

```bash
sudo systemctl enable containerd
sudo systemctl start containerd
```

---

### Step 2: Install Kubernetes Components

On **all nodes**:

```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### Step 3: Initialize Control Plane

On the **master node**:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Configure kubectl:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### Step 4: Install a Pod Network (REQUIRED)

Without a CNI, **pods cannot communicate**.

Example: **Flannel**

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

Other popular CNIs:

* Calico (advanced networking & policies)
* Cilium (eBPF-based, high performance)

---

### Step 5: Join Worker Nodes

On each worker:

```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

---

## 4️⃣ Deploy Distributed Applications (Microservices)

Example: **Go microservices** (relevant to your Golang work)

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user
  template:
    metadata:
      labels:
        app: user
    spec:
      containers:
      - name: user
        image: user-service:latest
        ports:
        - containerPort: 8080
```

Kubernetes will:

* Spread replicas across nodes
* Restart failed pods
* Load balance traffic

---

### Service (Internal Load Balancing)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user
  ports:
    - port: 80
      targetPort: 8080
```

---

## 5️⃣ Achieving Distributed System Guarantees

| Problem           | Kubernetes Feature |
| ----------------- | ------------------ |
| Service discovery | DNS + Services     |
| Load balancing    | Services + Ingress |
| Failure recovery  | ReplicaSets        |
| Scaling           | HPA                |
| Config sync       | ConfigMaps         |
| Secrets           | Secrets            |
| Rolling updates   | Deployments        |
| Isolation         | Namespaces         |

---

## 6️⃣ Expose Services Externally (No Minikube)

### Option A — LoadBalancer (Cloud)

```yaml
spec:
  type: LoadBalancer
```

### Option B — Ingress Controller

Install **NGINX Ingress**:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Ingress example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
```

---

## 7️⃣ Monitoring & Reliability (Production-Grade)

Minimum recommended stack:

* **Prometheus + Grafana** – metrics
* **Loki / ELK** – logs
* **Horizontal Pod Autoscaler**
* **Readiness & Liveness probes**

---

## 8️⃣ If You Want Cloud-Specific Instructions

I can give **exact steps** for:

* AWS EKS
* GCP GKE
* Azure AKS
* DigitalOcean
* Bare-metal + kubeadm
* Lightweight k3s cluster

Just tell me:
👉 **Cloud or self-hosted?**
👉 **How many nodes?**
👉 **Is this for learning or production?**

This setup is how real distributed systems run in industry 🚀


Yes — **that’s exactly right** 👍
In a real Kubernetes cluster (non-Minikube), nodes are **separate servers**.

---

## 🔹 Standard Kubernetes Cluster Layout

### 🧠 Control Plane (Master Node)

Deployed on **one or more servers**

* Runs:

  * `kube-apiserver`
  * `scheduler`
  * `controller-manager`
  * `etcd`
* **Does NOT run application workloads** (by default)
* Responsible for **cluster coordination**

Example:

```
Server A → Control Plane (Master)
```

---

### ⚙️ Worker Nodes

Each worker runs on a **different server**

* Runs:

  * `kubelet`
  * `container runtime`
  * `kube-proxy`
* Hosts:

  * Your **pods**
  * Your **microservices**

Example:

```
Server B → Worker Node
Server C → Worker Node
Server D → Worker Node
```

---

## 🔹 Typical Real-World Topologies

### 1️⃣ Small / Learning Cluster

```
1 Control Plane
2–3 Worker Nodes
```

---

### 2️⃣ Production (High Availability)

```
3 Control Plane Nodes (HA)
5+ Worker Nodes
```

Why 3 masters?

* etcd uses quorum
* survives 1 master failure

---

### 3️⃣ Cloud-Managed Kubernetes (EKS / GKE / AKS)

```
Control Plane → Managed by Cloud Provider
Worker Nodes → Your VMs
```

You don’t even see the master servers — but they still exist.

---

## 🔹 Networking Between Servers

All nodes must:

* Be able to **talk to each other**
* Have:

  * Open ports (6443, 10250, etc.)
  * Same internal network or VPN

Kubernetes networking enables:

* Pod-to-pod communication across servers
* Service discovery via DNS

---

## 🔹 Why This Is a Distributed System

Because:

* Services run on **multiple machines**
* Requests are **load-balanced**
* Nodes can fail without downtime
* Pods are rescheduled automatically

Example:

```
Client
  ↓
LoadBalancer
  ↓
Service
  ↓
Pods on multiple servers
```

---

## 🔹 Important Clarification

| Component               | Same server?            |
| ----------------------- | ----------------------- |
| Control plane + workers | ❌ (production)          |
| Multiple workers        | ❌                       |
| Pods in same deployment | ❌ (spread across nodes) |

> Kubernetes **intentionally spreads workloads** across servers for fault tolerance.

---

## 🔹 Can They Be on the Same Server?

Yes — **only for learning/testing**:

* Single-node cluster
* Master also acts as worker

❌ Not recommended for real distributed systems.

---

## 🔹 Summary

✔ Master node → one or more servers
✔ Worker nodes → different servers
✔ All connected via network
✔ Kubernetes orchestrates everything

---

If you want, I can:

* Draw an **exact architecture diagram**
* Show a **3-master HA setup**
* Map this to **AWS / GCP / bare metal**
* Show how your **Go microservices** fit in

Just tell me what you want next 👌
