# 🔷 Argo CD Architecture (Detailed Explanation)

**Argo CD follows a controller-based architecture and runs entirely inside a Kubernetes cluster.**
It continuously watches Git repositories and keeps the cluster state in sync with the desired state defined in Git.

---

## 🔷 High-Level Architecture Flow

```
Git Repository
      ↓
 Argo CD Repo Server
      ↓
Argo CD Application Controller
      ↓
Kubernetes API Server
      ↓
Kubernetes Cluster (Pods, Services, etc.)
```

---

## 🔷 Core Components of Argo CD

Argo CD has **four main components**, each with a specific responsibility.

---

## 1️⃣ Argo CD API Server

### 🔹 What it does:

* Exposes **Web UI**
* Provides **CLI access (`argocd` command)**
* Handles **authentication and authorization**
* Acts as the **entry point** for users

### 🔹 Key responsibilities:

* Login via GitHub, GitLab, SSO, or local users
* RBAC enforcement
* Receives user actions (sync, rollback, refresh)

📌 **Users interact ONLY with the API Server**, not directly with other components.

---

## 2️⃣ Repository Server (Repo Server)

### 🔹 What it does:

* Connects to **Git repositories**
* Fetches Kubernetes manifests
* Renders templates

### 🔹 Supported formats:

* Plain YAML
* Helm charts
* Kustomize
* Jsonnet

### 🔹 Example:

If your repo has:

```text
k8s/
 ├── deployment.yaml
 ├── service.yaml
```

Repo Server:

* Pulls the repo
* Converts everything into **plain Kubernetes manifests**

📌 Repo Server **does NOT deploy anything**, it only prepares manifests.

---

## 3️⃣ Application Controller (Most Important Component)

### 🔹 What it does:

* Core brain of Argo CD
* Continuously compares:

  * **Desired state (Git)**
  * **Live state (Kubernetes cluster)**

### 🔹 Key responsibilities:

* Detects drift
* Syncs changes
* Self-heals resources
* Prunes deleted resources
* Updates application health and sync status

### 🔹 Example:

* Git says replicas = 3
* Cluster has replicas = 2
* Controller detects drift
* Controller fixes it automatically

📌 This component makes Argo CD **pull-based**.

---

## 4️⃣ Redis

### 🔹 What it does:

* Caching layer
* Improves performance
* Stores:

  * Application state
  * Metadata
  * Comparison results

📌 Redis is **not mandatory**, but recommended for large setups.

---

## 🔷 Kubernetes API Server (External but Critical)

* Argo CD communicates with Kubernetes via **Kubernetes API Server**
* Uses **Service Accounts**
* Applies manifests using Kubernetes APIs

📌 Argo CD never uses `kubectl` manually.

---

## 🔷 Argo CD Application (Logical Component)

An **Application** is a **CRD (Custom Resource Definition)**.

It defines:

* Which Git repo
* Which path
* Which branch
* Which cluster & namespace

### Example fields:

* `source.repoURL`
* `source.path`
* `source.targetRevision`
* `destination.server`
* `destination.namespace`

The Application Controller reads this object and acts accordingly.

---

## 🔷 Authentication & Authorization Flow

### Authentication:

* LDAP
* GitHub / GitLab
* SSO (OIDC)
* Local users

### Authorization:

* Kubernetes RBAC
* Argo CD RBAC policies

📌 Ensures **secure access** in enterprise environments (banks, MNCs).

---

## 🔷 Sync & Drift Detection Flow

1. Developer commits changes to Git
2. Repo Server fetches latest commit
3. Application Controller compares states
4. If different:

   * Syncs automatically (if enabled)
   * Or waits for manual approval
5. Health status updated in UI

---

## 🔷 Architecture Diagram (Text-Based)

```
User / CLI / UI
       ↓
   API Server
       ↓
-------------------------
|   Application Controller   |
-------------------------
       ↓
Kubernetes API Server
       ↓
Kubernetes Cluster
       ↑
-------------------------
|     Repo Server           |
-------------------------
       ↑
    Git Repository
```

---

## 🔷 Why This Architecture is Powerful

* **Pull-based** → more secure
* **Git as source of truth**
* **No direct cluster access for developers**
* **Easy rollback**
* **Scalable for microservices**

---

## 🔷 Interview One-Line Architecture Summary

> **Argo CD uses a controller-based architecture where the Application Controller continuously reconciles the desired state from Git with the live state of the Kubernetes cluster using the Kubernetes API.**

---

## 🔷 Real-World Use Case (Your CI/CD Project)

* Jenkins updates Git with new image tag
* Argo CD Repo Server fetches changes
* Application Controller deploys to cluster
* Redis caches state
* API Server shows status

This matches **enterprise GitOps architecture**.
