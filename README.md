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

### Install Argo CD CLI
```
brew install argocd       # macOS
choco install argocd-cli  # Windows
```

### Login to Argo CD via CLI
```
argocd login localhost:8080 --username admin --password <password>
```

### Sync the App

```bash
argocd app sync guestbook
```

---


### Check Services in `argocd` Namespace

```bash
kubectl get svc -n argocd
```

Look for something like:

```
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
guestbook-ui    NodePort   10.101.13.213    <none>        80:30987/TCP     5m
```

---

### Expose and Open in Browser

Run:

```bash
minikube service guestbook-ui -n argocd
```

This will:

* Automatically open the browser to the guestbook app.
* Or show a URL like `http://127.0.0.1:30987`.

---

### ✅ (Optional) If It’s a ClusterIP, Use Port Forwarding

If the service is **ClusterIP** instead of NodePort, forward manually:

```bash
kubectl port-forward svc/guestbook-ui 8080:80 -n argocd
```

Then open your browser at:

```
http://localhost:8080
```

---

### ✅ Bonus: Verify the Pod

Make sure your app pod is running:

```bash
kubectl get pods -n argocd -l app=guestbook-ui
```

---

## 💻 5. **Hands-On Demo Flow**

* Show current state of the cluster: no guestbook app yet.
* Apply the `Application` manifest via `kubectl apply`.
* Open Argo CD UI → observe the app get synced automatically.
* Modify a value in Git (e.g., replica count), commit, and push.
* Watch Argo CD detect the change and auto-sync it.

---

### Cleanup the resources

---

### ✅ 1. **Stop the Argo CD Port Forward** (CLI access or UI)

If you ran this:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Just press **`Ctrl+C`** in that terminal. This only stops the local port forwarding — Argo CD itself is still running in the cluster.

---

### ✅ 2. **Stop Argo CD Deployment** (temporary shutdown)

To stop Argo CD entirely in your cluster, you can scale down the deployments:

```bash
kubectl scale deployment argocd-server -n argocd --replicas=0
kubectl scale deployment argocd-repo-server -n argocd --replicas=0
kubectl scale deployment argocd-application-controller -n argocd --replicas=0
kubectl scale deployment argocd-dex-server -n argocd --replicas=0
```

> This will **stop all Argo CD components** without uninstalling anything.

To bring it back:

```bash
kubectl scale deployment argocd-server -n argocd --replicas=1
# repeat for the others as needed
```

---

### ✅ 3. **Completely Uninstall Argo CD**

If you're done with Argo CD:

```bash
kubectl delete namespace argocd
```

Or if you installed it using a manifest:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## 🔧 **Other Use Cases with Argo CD**

### 1. **Multi-Environment Management**

* Manage `dev`, `qa`, `stage`, and `prod` clusters with Git branches or folder-based structure.
* Easily promote changes between environments using Git commits.

### 2. **Helm/Kustomize Integration**

* Supports Helm charts and Kustomize natively.
* Great for templated apps or managing per-environment config with overlays.

### 3. **Multi-Cluster Deployment**

* Deploy to multiple Kubernetes clusters from one Argo CD instance.
* Ideal for managing regional workloads or hybrid cloud setups.

### 4. **Drift Detection & Auto Healing**

* Argo CD continuously compares the live cluster state to the Git repo.
* If someone manually changes a resource (kubectl edit), Argo CD will alert or revert (if self-heal is enabled).

### 5. **App-of-Apps Pattern**

* Bootstrap complex applications made of many sub-applications.
* Example: a "platform" app that deploys monitoring, logging, networking, and your apps.

### 6. **Auditing and Compliance**

* Since all changes flow through Git, you have a full audit trail.
* Integrates well with policy tools like OPA/Gatekeeper for enforcing standards.

### 7. **RBAC and SSO Integration**

* Argo CD supports enterprise authentication systems like SSO, LDAP, OIDC.
* Fine-grained RBAC for teams and environments.

---


