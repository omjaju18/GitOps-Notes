# 🚀 Argo CD Installation & Setup (Step-by-Step Guide)

## 🔷 What We Are Doing

We will:

1. Install Argo CD on a Kubernetes cluster
2. Access Argo CD UI & CLI
3. Connect a Git repository
4. Create an Application
5. Deploy an app using GitOps

---

## 🔹 Prerequisites

Make sure you have:

* A running **Kubernetes cluster**

  * Minikube / Kind / EKS / AKS / GKE
* `kubectl` installed and configured
* Internet access to pull manifests
* Basic Kubernetes knowledge

Check cluster access:

```bash
kubectl get nodes
```

---

## 🔹 Step 1: Create Argo CD Namespace

Argo CD runs **inside Kubernetes**, so first create a namespace.

```bash
kubectl create namespace argocd
```

📌 Why?

* Keeps Argo CD isolated
* Easy management and security

---

## 🔹 Step 2: Install Argo CD

Apply the official Argo CD manifests:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### What this installs internally:

* API Server
* Repo Server
* Application Controller
* Redis
* Services, ConfigMaps, Secrets

Check pods:

```bash
kubectl get pods -n argocd
```

Wait until all pods are **Running**.

---

## 🔹 Step 3: Access Argo CD UI

By default, Argo CD server is **ClusterIP** (internal only).

### Option 1: Port Forward (Most Common)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open browser:

```
https://localhost:8080
```

---

## 🔹 Step 4: Get Admin Password

Argo CD auto-creates an admin password in a secret.

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

Login:

* **Username:** `admin`
* **Password:** (output of command)

---

## 🔹 Step 5: Install Argo CD CLI (Optional but Recommended)

### Linux:

```bash
curl -sSL -o argocd \
https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

chmod +x argocd
sudo mv argocd /usr/local/bin/
```

Verify:

```bash
argocd version
```

---

## 🔹 Step 6: Login Using CLI

```bash
argocd login localhost:8080
```

📌 CLI is useful for:

* Automation
* Scripting
* CI/CD pipelines

---

## 🔹 Step 7: Connect Git Repository

Argo CD needs access to your Git repo.

### Public Repo

No credentials needed.

### Private Repo (Example: GitHub)

```bash
argocd repo add https://github.com/USERNAME/REPO.git \
  --username YOUR_USERNAME \
  --password YOUR_GITHUB_TOKEN
```

Verify:

```bash
argocd repo list
```

---

## 🔹 Step 8: Create an Application (Core Setup)

An **Application** tells Argo CD:

* Where the code is
* Where to deploy it

### Example Command

```bash
argocd app create my-app \
  --repo https://github.com/USERNAME/REPO.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --revision main
```

### What happens:

* Repo Server pulls manifests
* Controller compares with cluster
* App appears in UI

---

## 🔹 Step 9: Sync the Application

### Manual Sync

```bash
argocd app sync my-app
```

### Enable Auto Sync

```bash
argocd app set my-app \
  --sync-policy automated \
  --self-heal \
  --prune
```

📌 Now Git changes deploy automatically.

---

## 🔹 Step 10: Verify Deployment

Check app status:

```bash
argocd app list
```

Check Kubernetes:

```bash
kubectl get pods -n default
```

---

## 🔹 Step 11: Update App (GitOps in Action)

1. Change image version in Git
2. Commit & push

```bash
git commit -am "Update app version"
git push
```

👉 Argo CD automatically syncs and updates pods.

---

## 🔹 Step 12: Rollback Application

Rollback to a previous version:

```bash
argocd app rollback my-app
```

Or use UI → **History & Rollback**

---

## 🔷 Optional Production Configurations

### Change Service Type to LoadBalancer

```bash
kubectl edit svc argocd-server -n argocd
```

### Enable SSO / RBAC

* Integrate LDAP / OIDC
* Define RBAC policies

---

## 🔷 Common Installation Issues & Fixes

| Issue             | Fix                     |
| ----------------- | ----------------------- |
| Pods not running  | Check node resources    |
| UI not accessible | Use port-forward        |
| Repo auth failed  | Check token permissions |
| App OutOfSync     | Check YAML & namespace  |

---

## 🔷 Interview One-Line Summary

> **Argo CD is installed by deploying its controller components inside a Kubernetes cluster, connecting it to Git repositories, and defining Applications that continuously synchronize the cluster state with Git using a pull-based GitOps model.**

---

## ✅ Final Mental Model

1. Install Argo CD
2. Access UI / CLI
3. Connect Git
4. Create Application
5. Enable auto-sync
6. Git controls deployments 🚀
