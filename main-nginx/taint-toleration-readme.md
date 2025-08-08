
# 🧪 Kubernetes Taints and Tolerations — Simple Example

This document explains how to use **taints** and **tolerations** in Kubernetes, with a real example.

---

## ✅ Step-by-Step Example

### 🔹 1. Add a Taint to a Node

Let’s say we want to **taint** a node called `worker-node-1` so that only specific pods can be scheduled on it.

```bash
kubectl taint nodes worker-node-1 env=prod:NoSchedule
```

📌 This means:  
> "Don’t schedule any pods on `worker-node-1` **unless** they tolerate the taint `env=prod:NoSchedule`."

---

### 🔹 2. Create a Deployment with Matching Toleration

Now we create a deployment where pods **tolerate** this taint.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-toleration
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
      tolerations:
        - key: "env"
          operator: "Equal"
          value: "prod"
          effect: "NoSchedule"
```

📌 This tells Kubernetes:  
> "This pod can run on any node tainted with `env=prod:NoSchedule`."

---

### 🔹 3. What Happens Without Toleration?

If a pod **doesn't have a toleration** for `env=prod:NoSchedule`, it **won’t be scheduled** on `worker-node-1`.

---

### 🔹 4. Remove the Taint (Optional)

If you want to **remove the taint**, use:

```bash
kubectl taint nodes worker-node-1 env=prod:NoSchedule-
```

Note the `-` at the end — it removes the taint.
