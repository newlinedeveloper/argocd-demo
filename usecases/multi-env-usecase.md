# Multi-Environment Deployment Using ArgoCD and Minikube

I'd be happy to walk you through setting up a multi-environment deployment pipeline (Dev → QA → Prod) using ArgoCD on Minikube. This is a common pattern for GitOps-based continuous delivery.

## Overview of the Demo

We'll create:
1. A simple application with environment-specific configurations
2. A Git repository structure that supports multi-environment deployments
3. ArgoCD setup in Minikube
4. Application deployments across Dev, QA, and Prod environments

Let's get started with a step-by-step guide.

## Step 1: Set Up Minikube and ArgoCD

First, we need to start Minikube and install ArgoCD:

```bash
# Start Minikube with enough resources
minikube start --cpus 4 --memory 8192 --driver=docker

# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

# Port forward ArgoCD server (run in a separate terminal)
# kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get the ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Step 2: Create Environment Namespaces

Let's create separate namespaces for our environments:

```bash
kubectl create namespace dev
kubectl create namespace qa
kubectl create namespace prod
```

## Step 3: Create a Git Repository Structure

We'll set up a Git repository with the following structure:

```
├── app/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── overlays/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── config.yaml
│       ├── qa/
│       │   ├── kustomization.yaml
│       │   └── config.yaml
│       └── prod/
│           ├── kustomization.yaml
│           └── config.yaml
└── argocd/
    ├── dev-app.yaml
    ├── qa-app.yaml
    └── prod-app.yaml
```

Let's create these files:

Now let's create the environment-specific overlays:

## Step 4: Create ArgoCD Application Definitions

Now let's create the ArgoCD Application manifests for each environment:

## Step 5: Implementation Steps

Here's a script to help you set up and test everything:

## Step 6: Monitoring and Verifying Deployments

After running the setup script, you can verify that ArgoCD is syncing your applications by checking:

```bash
# Check ArgoCD applications status
kubectl get applications -n argocd

# Check the deployments in each environment
kubectl get deployments -n dev
kubectl get deployments -n qa
kubectl get deployments -n prod
```

You can access the ArgoCD UI to visualize your deployments:

```bash
# Port forward the ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access in your browser: https://localhost:8080
# Username: admin
# Password: (retrieved from the script output)
```

## Step 7: Making Changes to Demonstrate GitOps Flow

To demonstrate the GitOps workflow, you can make changes to your configuration in Git:

1. Modify the number of replicas in one of the environment config files
2. Commit and push the changes
3. ArgoCD will automatically detect and apply the changes to the corresponding environment

For example, to update the number of replicas in the dev environment:

```bash
# Update the dev config
cd $TEMP_DIR  # The directory created by the script
sed -i 's/replicas: 1/replicas: 2/' app/overlays/dev/config.yaml

# Commit the change
git add .
git commit -m "Increase dev replicas to 2"

# Wait for ArgoCD to sync
kubectl get deployments -n dev -w
```

## Summary of the Multi-Environment Demo

This setup demonstrates:

1. **Environment Separation**: Each environment (dev, qa, prod) has its own namespace and configuration
2. **GitOps Workflow**: All changes are made through Git, not directly to the cluster
3. **Configuration Management**: Using Kustomize to manage environment-specific configurations
4. **Automated Sync**: ArgoCD automatically deploys changes when they're pushed to Git
5. **Progressive Delivery**: Changes can be tested in dev and qa before being applied to prod

The key differentiators between environments are:
- **Dev**: 1 replica, minimal resources
- **QA**: 2 replicas, moderate resources
- **Prod**: 3 replicas, higher resources

This approach allows you to maintain a single code base with environment-specific configurations, making it easy to promote changes through your environments while ensuring consistency.

Would you like me to expand on any specific part of this demo?
