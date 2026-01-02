# 🔷 What is the Argo CD Hub-and-Spoke Model?

The **Hub-and-Spoke model** is a **multi-cluster Argo CD architecture** where:

* **One central Argo CD (Hub)** manages
* **Multiple Kubernetes clusters (Spokes)**

Each spoke cluster runs applications, but **Argo CD is controlled from one central place**.

---

## 🔷 Why This Model Exists

In real companies:

* You have **many clusters**

  * Dev
  * QA
  * UAT
  * Prod
* Sometimes across:

  * Different regions
  * Different cloud providers
* You want:

  * Central control
  * Strong security
  * Clear separation of environments

➡️ Hub-and-Spoke solves this.

---

## 🔷 High-Level Architecture

```
                 Git Repositories
                        |
                        v
                 +----------------+
                 |   Argo CD HUB   |
                 | (Management)   |
                 +----------------+
                   |      |      |
                   v      v      v
              +--------+ +--------+ +--------+
              | Dev    | | QA     | | Prod   |
              | Spoke  | | Spoke  | | Spoke  |
              +--------+ +--------+ +--------+
             (K8s)      (K8s)      (K8s)
```

---

## 🔷 Hub Cluster (Control Plane)

### 🔹 What runs in the Hub:

* Argo CD components:

  * API Server
  * Repo Server
  * Application Controller
  * Redis
* Central RBAC
* Central authentication (SSO, LDAP)

### 🔹 Responsibilities:

* Connect to **all Git repositories**
* Manage **Application definitions**
* Control **who can deploy where**
* Enforce **policies**

📌 The Hub **does NOT run workloads** (best practice).

---

## 🔷 Spoke Clusters (Execution Plane)

### 🔹 What runs in each Spoke:

* Business applications
* Kubernetes workloads
* Optional:

  * No Argo CD installed
  * Or a lightweight Argo CD agent

### 🔹 Responsibilities:

* Run application Pods
* Expose services
* Send status back to Hub

📌 Spokes are **isolated from each other**.

---

## 🔷 How Hub Manages Spokes (Key Mechanism)

Argo CD Hub:

* Registers spoke clusters using:

  * Kubernetes contexts
  * Service accounts
  * TLS authentication

Example:

```bash
argocd cluster add prod-cluster
```

This creates:

* A **ServiceAccount** in the spoke
* RBAC permissions
* Secure communication channel

---

## 🔷 Application Deployment Flow

1. Developer commits code to Git
2. Git contains:

   * App manifests
   * Environment-specific configs
3. Argo CD Hub:

   * Detects Git change
   * Determines target cluster
4. Hub deploys app to:

   * Dev / QA / Prod spoke
5. Spoke runs the workload
6. Status reported back to Hub

---

## 🔷 Application Placement Example

### Git Structure

```
apps/
 ├── guestbook/
 │    ├── base/
 │    ├── overlays/
 │         ├── dev/
 │         ├── qa/
 │         └── prod/
```

### Argo CD Applications

* Same app
* Different destination clusters

---

## 🔷 App-of-Apps Pattern (Used with Hub-Spoke)

The Hub often uses **App-of-Apps**:

* One **parent app** per environment
* Child apps per microservice

Benefits:

* Scalable
* Clean separation
* Easy onboarding

---

## 🔷 Security Benefits (Very Important)

### 🔐 Strong Isolation

* Dev cannot touch Prod
* Separate credentials per spoke

### 🔐 Least Privilege

* Hub has limited access to spokes
* Spokes expose only required permissions

### 🔐 Central Audit

* All changes tracked in Git
* Full deployment history

---

## 🔷 Hub-Spoke vs Single Cluster Argo CD

| Feature            | Single Cluster | Hub-Spoke        |
| ------------------ | -------------- | ---------------- |
| Number of clusters | One            | Many             |
| Control            | Local          | Central          |
| Security           | Basic          | Enterprise-grade |
| Scaling            | Limited        | High             |
| Governance         | Weak           | Strong           |

---

## 🔷 Real-World Use Case (Bank / Enterprise)

Example:

* Hub cluster in **management VPC**
* Spokes in:

  * Dev VPC
  * QA VPC
  * Prod VPC
* Strict firewall rules:

  * Only Hub → Spoke allowed
* No direct kubectl access to Prod

---

## 🔷 Advantages of Hub-and-Spoke

✅ Centralized management
✅ Scales to dozens of clusters
✅ Strong security boundaries
✅ Easy compliance & audit
✅ GitOps consistency

---

## 🔷 Challenges / Limitations

⚠️ Hub is critical (needs HA)
⚠️ Initial setup complexity
⚠️ Requires careful RBAC design

---

## 🔷 Interview One-Line Summary

> **The Argo CD Hub-and-Spoke model uses a central Argo CD instance to manage deployments across multiple Kubernetes clusters, providing strong security, centralized governance, and scalable GitOps operations.**

---

## 🔷 When to Use Hub-and-Spoke

* Multi-environment setups
* Multi-region deployments
* Regulated industries
* Large microservice platforms

---

## ✅ Final Mental Model

* **Hub = Brain / Control Tower**
* **Spokes = Execution Clusters**
* **Git = Single Source of Truth**
* **Argo CD = GitOps Engine**
---

# 🔷 Argo CD Hub–Spoke Workflow (Step by Step)

## 🎯 Goal

* **1 Hub cluster** → runs Argo CD (management)
* **2 Spoke clusters** → run applications (Dev & Prod)
* **Git** → single source of truth
* **Hub deploys apps to Spokes**

---

## 🔹 Step 0: Prerequisites

You need:

* kubectl configured for **multiple clusters**
* Cluster contexts:

  ```bash
  kubectl config get-contexts
  ```

Example:

```text
hub-cluster
dev-cluster
prod-cluster
```

---

## 🔹 Step 1: Install Argo CD on the HUB Cluster

Switch to hub cluster:

```bash
kubectl config use-context hub-cluster
```

Install Argo CD:

```bash
kubectl create namespace argocd

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for pods:

```bash
kubectl get pods -n argocd
```

---

## 🔹 Step 2: Access Argo CD

Port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Get admin password:

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

Login:

```bash
argocd login localhost:8080
```

---

## 🔹 Step 3: Register Spoke Clusters with the Hub

### ➤ Add DEV Spoke Cluster

```bash
argocd cluster add dev-cluster
```

### ➤ Add PROD Spoke Cluster

```bash
argocd cluster add prod-cluster
```

What happens internally:

* Argo CD creates a **ServiceAccount** in the spoke
* Adds **RBAC permissions**
* Stores credentials securely in the hub

Verify:

```bash
argocd cluster list
```

---

## 🔹 Step 4: Git Repository Structure (Very Important)

Example Git repo:

```
gitops-repo/
├── apps/
│   └── guestbook/
│       ├── base/
│       │   ├── deployment.yaml
│       │   └── service.yaml
│       └── overlays/
│           ├── dev/
│           │   └── kustomization.yaml
│           └── prod/
│               └── kustomization.yaml
```

Git = **single source of truth**

---

## 🔹 Step 5: Create Namespace in Spoke Clusters

```bash
kubectl config use-context dev-cluster
kubectl create namespace guestbook

kubectl config use-context prod-cluster
kubectl create namespace guestbook
```

---

## 🔹 Step 6: Create Argo CD Application for DEV (From Hub)

Switch back to hub:

```bash
kubectl config use-context hub-cluster
```

Create app:

```bash
argocd app create guestbook-dev \
  --repo https://github.com/omjaju18/gitops-repo.git \
  --path apps/guestbook/overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace guestbook \
  --revision main
```

Enable auto-sync:

```bash
argocd app set guestbook-dev \
  --sync-policy automated \
  --self-heal \
  --prune
```

---

## 🔹 Step 7: Create Argo CD Application for PROD

```bash
argocd app create guestbook-prod \
  --repo https://github.com/omjaju18/gitops-repo.git \
  --path apps/guestbook/overlays/prod \
  --dest-name prod-cluster \
  --dest-namespace guestbook \
  --revision main
```

Enable auto-sync:

```bash
argocd app set guestbook-prod \
  --sync-policy automated \
  --self-heal \
  --prune
```

---

## 🔹 Step 8: Sync Applications

Manual sync (optional):

```bash
argocd app sync guestbook-dev
argocd app sync guestbook-prod
```

Check status:

```bash
argocd app list
```

---

## 🔹 Step 9: Verify Workloads in Spoke Clusters

DEV:

```bash
kubectl config use-context dev-cluster
kubectl get pods -n guestbook
```

PROD:

```bash
kubectl config use-context prod-cluster
kubectl get pods -n guestbook
```

✔ Apps are running
✔ Hub did not run workloads
✔ Spokes run workloads

---

## 🔹 Step 10: Update Application (GitOps in Action)

Change image tag in Git:

```yaml
image: guestbook:v2
```

Commit:

```bash
git commit -am "update guestbook image"
git push
```

What happens:

1. Git updated
2. Hub detects commit
3. Argo CD syncs
4. Spoke clusters update automatically

🚀 **No kubectl apply needed**

---

## 🔹 Step 11: Rollback (Very Important)

Rollback to previous version:

```bash
argocd app rollback guestbook-prod
```

Or via UI → **History & Rollback**

---

## 🔹 Step 12: App-of-Apps (Enterprise Pattern)

Parent app manages child apps:

```yaml
apps/
├── dev/
│   ├── guestbook.yaml
│   ├── payments.yaml
│   └── orders.yaml
```

Hub manages **everything from one app**.

---

## 🔹 Complete Flow Summary (Simple Words)

1. Hub installs Argo CD
2. Hub connects to spoke clusters
3. Git stores Kubernetes YAML
4. Hub reads Git
5. Hub deploys to spokes
6. Spokes run applications
7. Git change → auto deploy

---

## 🔹 One-Line Interview Answer

> “In the Argo CD hub–spoke model, a central Argo CD instance manages deployments across multiple Kubernetes clusters by pulling application definitions from Git and synchronizing them securely to spoke clusters.”

---

## 🔹 Common Interview Follow-Ups

**Q: Where do workloads run?**
→ Spoke clusters

**Q: What happens if hub goes down?**
→ Apps keep running, deployments pause

**Q: Is kubectl access needed in prod?**
→ No, Git + Argo CD only

---
