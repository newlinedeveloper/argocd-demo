**3 powerful Argo CD use cases** beyond just basic app deployment 
---

## 🚀 Use Case 1: **Multi-Environment Deployment (Dev → QA → Prod)**

### 🧠 Goal:

Deploy the same app to **multiple environments**, each with different config (replicas, resources, etc.), managed via Git.

### 📁 Git Structure (monorepo):

```
apps/
  guestbook/
    dev/
      kustomization.yaml
      deployment.yaml
    qa/
      kustomization.yaml
      deployment.yaml
    prod/
      kustomization.yaml
      deployment.yaml
```

### 🧪 Demo Steps:

1. Install Kustomize version of guestbook app in Git.
2. Create 3 Argo CD apps:

   ```yaml
   metadata:
     name: guestbook-dev
   spec:
     source:
       path: apps/guestbook/dev
   ---
   metadata:
     name: guestbook-prod
   spec:
     source:
       path: apps/guestbook/prod
   ```
3. Push a change to `deployment.yaml` in `dev/` → Argo CD auto-syncs.
4. Show how "promotion" works: cherry-pick change to `prod/` → triggers deployment.

### ✅ Value:

* Real-world GitOps promotion flow
* Decouples code & infra per environment
* Encourages approval flow via Git PRs

---

## ⚙️ Use Case 2: **App-of-Apps Pattern**

### 🧠 Goal:

Use a single **"parent" app** to bootstrap a full platform stack (logging, monitoring, business apps).

### 📁 Git Structure:

```
apps/
  platform/
    app-of-apps.yaml
  monitoring/
    prometheus.yaml
  logging/
    loki.yaml
  app1/
    deployment.yaml
```

### 📝 Parent App YAML:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform
spec:
  source:
    path: apps/platform
  destination:
    namespace: argocd
  syncPolicy:
    automated: {}
```

Inside `platform/app-of-apps.yaml`:

```yaml
applications:
  - name: prometheus
    path: monitoring/
  - name: loki
    path: logging/
  - name: app1
    path: app1/
```

### 🧪 Demo Steps:

1. Apply the "platform" app.
2. Argo CD spawns 3 child apps.
3. Change version in `app1/` and push → only that app gets updated.

### ✅ Value:

* Modular, scalable deployments
* Great for managing microservices or platforms

---

## 🔐 Use Case 3: **Drift Detection + Self-Healing**

### 🧠 Goal:

Demonstrate how Argo CD notices and optionally corrects manual cluster changes.

### 🧪 Demo Steps:

1. Deploy an app via Argo CD.
2. Use `kubectl edit deployment guestbook` to change replica count (e.g., 3 → 1).
3. Watch Argo CD:

   * Flags app as **OutOfSync**
   * If `selfHeal: true`, it auto-fixes it back to **Git state**
4. Optionally disable auto-heal to show manual sync

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### ✅ Value:

* Shows Git truly is the "source of truth"
* Validates security/audit/compliance use cases

---

