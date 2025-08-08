
# Kubernetes Controllers: Deployment vs ReplicaSet vs DaemonSet (with YAML Differences)

---

## ✅ Overview
Kubernetes offers multiple controllers for managing Pods. The most common are:

- **Deployment** → Manages ReplicaSets and Pods; supports rolling updates and rollbacks.
- **ReplicaSet** → Ensures a specific number of identical Pods are running.
- **DaemonSet** → Ensures **one Pod per node** (or per selected nodes).

---

## ✅ Key Differences

| Feature         | Deployment                   | ReplicaSet                       | DaemonSet                               |
| --------------- | ---------------------------- | -------------------------------- | --------------------------------------- |
| **Purpose**     | Declarative updates, scaling | Maintain replica count           | One Pod per node                        |
| **Updates**     | Rolling updates supported    | No rolling updates               | No rolling updates (manual or recreate) |
| **Manages**     | ReplicaSets (and Pods)       | Pods only                        | Pods only                               |
| **Scaling**     | Yes                          | Yes                              | No (1 per node automatically)           |
| **Use Case**    | Web apps, scalable services  | Rare direct use                  | Node-level services like logging agents |

---

## ✅ Base Reference: Deployment YAML

```yaml
kind: Deployment  # Base controller, supports rolling updates and rollback
apiVersion: apps/v1
metadata:
  name: nginx-deployment   # Deployment resource name
  namespace: nginx
spec:
  replicas: 5              # Defines number of Pods
  selector:
    matchLabels:
      app: nginx
  template:                # Pod template definition
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

---

## ✅ ReplicaSet YAML (Differences Commented)

```yaml
kind: ReplicaSet   # DIFFERENCE: ReplicaSet does NOT manage rolling updates like Deployment
apiVersion: apps/v1
metadata:
  name: nginx-replicaset    # DIFFERENCE: Resource name changed to nginx-replicaset
  namespace: nginx
spec:
  replicas: 5  # SAME: Replica count, but no rolling updates feature
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
          ports:
            - containerPort: 80
```

---

## ✅ DaemonSet YAML (Differences Commented)

```yaml
kind: DaemonSet    # DIFFERENCE: Ensures one Pod per Node (no replicas field)
apiVersion: apps/v1
metadata:
  name: nginx-daemonset    # DIFFERENCE: Resource name changed to nginx-daemonset
  namespace: nginx
spec:
  # DIFFERENCE: No replicas field in DaemonSet → It automatically creates one Pod per node
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
          ports:
            - containerPort: 80
```

---

## ✅ Quick Summary

- **Deployment** → For scalable applications, supports rolling updates.
- **ReplicaSet** → Maintains Pods but lacks update strategy.
- **DaemonSet** → Runs a Pod on every node for node-level tasks.
