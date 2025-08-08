# 📘 Kubernetes Commands Cheat Sheet

---

## ✅ 1. Basic Cluster Information

```bash
# Get all namespaces
kubectl get ns

# Get all nodes in the cluster
kubectl get nodes

# Get pods in default namespace
kubectl get pods

# Get pods in a specific namespace (e.g., nginx)
kubectl get pods -n nginx

# Watch pods in real-time (useful for scaling/deployment)
kubectl get pods -n nginx -w
```

---

## ✅ 2. Apply & Delete Resources

```bash
# Apply a YAML configuration file (create or update resource)
kubectl apply -f filename.yaml

# Delete resources defined in a YAML file
kubectl delete -f filename.yaml
```

---

## ✅ 3. Pod Management

```bash
# Get detailed information about a Pod
kubectl describe pod <pod-name> -n nginx

# Access Pod's shell
kubectl exec -it <pod-name> -n nginx -- bash

# View Pod logs
kubectl logs <pod-name> -n nginx
```

---

## ✅ 4. Deployment, ReplicaSet, and Namespace Info

```bash
# Get Deployments
kubectl get deploy -n nginx

# Get ReplicaSets
kubectl get rs -n nginx

# Get Pods with wide output (includes IP, Node)
kubectl get pods -n nginx -o wide
```

---

## ✅ 5. Additional Useful Commands

```bash
# Check Kubernetes API versions
kubectl api-resources

# Dry run resource creation (test YAML without applying)
kubectl apply -f file.yaml --dry-run=client
```

---

### ✅ To Update This File:

Keep adding new commands under the respective sections as you learn more.

