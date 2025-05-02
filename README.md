### GitOps Made Easy: Deploying Kubernetes Apps with Argo CD
---

## 🧩 1. **What is GitOps?**

### ✅ Concept:

* GitOps is a way of doing Continuous Deployment using Git as the **source of truth**.
* Your Kubernetes manifests (YAMLs) live in Git.
* Any change is made via a Git commit → automatically applied to the cluster.

### ✅ Core Principles:

* **Versioned and declarative**: Git holds the desired state.
* **Pull-based deployment**: Tools (like Argo CD) pull changes from Git and apply them.
* **Audit and rollback**: Git history becomes your audit trail.

### 🧠 Example:

> “Imagine rolling back a deployment just by reverting a Git commit — that’s GitOps in action.”

---

## 🚀 2. **Why Argo CD? (vs manual `kubectl apply`)**

### ❌ Problems with `kubectl`:

* Manual steps — prone to human error.
* Hard to track changes across environments.
* No visibility into current vs desired state.

### ✅ Advantages of Argo CD:

* **Declarative + automatic sync** from Git to cluster.
* **Web UI and CLI** to monitor and manage apps.
* **Rollback support** by reverting Git commits.
* **Self-healing**: detects drift from Git and reverts automatically (if enabled).
* Works with Helm, Kustomize, plain YAML, etc.

---

## ⚙️ 3. **Installing Argo CD (on Minikube)**

### 🔧 Prerequisites:

* Minikube installed and running:

  ```bash
  minikube start
  ```

* Create `argocd` namespace and install Argo CD:

  ```bash
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

* Port forward the UI:

  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```

* Get initial admin password:

  ```bash
  kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
  ```

---

## 📦 4. **Creating and Syncing Your First App (e.g., Guestbook)**

### 🛠️ Sample Git Repo:

Use [https://github.com/argoproj/argocd-example-apps](https://github.com/argoproj/argocd-example-apps) or your own repo with manifests.

### 📝 Example `app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 👇 Apply it:

```bash
kubectl apply -f app.yaml -n argocd
```

Then sync using:

```bash
argocd app sync guestbook
```

---

## 💻 5. **Hands-On Demo Flow**

* Show current state of the cluster: no guestbook app yet.
* Apply the `Application` manifest via `kubectl apply`.
* Open Argo CD UI → observe the app get synced automatically.
* Modify a value in Git (e.g., replica count), commit, and push.
* Watch Argo CD detect the change and auto-sync it.

---
