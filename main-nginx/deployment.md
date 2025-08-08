# Kubernetes Deployment Documentation

## ✅ File Name: `nginx-deployment.yaml`

---

## ✅ What is a Deployment?

A **Deployment** in Kubernetes is a higher-level abstraction that manages **ReplicaSets**, which in turn manage **Pods**. It provides:

* **Declarative Updates** → You define the desired state, and Kubernetes ensures it.
* **Scalability** → Easily scale up/down Pods.
* **Self-healing** → If a Pod crashes, a new one is created automatically.

---

## ✅ Resource Hierarchy

```
Deployment → ReplicaSet → Pods
```

* **Deployment**: Manages **ReplicaSets** and rolling updates.
* **ReplicaSet**: Ensures a specific number of Pod replicas are running.
* **Pods**: Smallest deployable unit, running your container(s).

---

## ✅ YAML File: `nginx-deployment.yaml`

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 5              # Number of Pods
  selector:
    matchLabels:
      app: nginx           # Must match template labels
  template:                # Pod template (defines Pods)
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

### ✅ **Explanation**

#### **kind & apiVersion**

* `kind: Deployment` → Object type is a Deployment.
* `apiVersion: apps/v1` → Current stable API for Deployments.

#### **metadata**

* `name: nginx-deployment` → Deployment name.
* `namespace: nginx` → This Deployment runs in `nginx` namespace.

#### **spec.replicas**

* `replicas: 5` → Desired Pod count.

#### **spec.selector**

* Ensures Deployment knows which Pods to manage.
* Must match `metadata.labels` inside `template`.

#### **template**

* This is the **default Pod blueprint** used by the ReplicaSet.
* Structure inside template is the same as a **Pod manifest**.

#### **containers**

* `image: nginx:latest` → Docker image for Pods.
* `ports.containerPort: 80` → Nginx runs on port 80.

---

## ✅ Commands Used

```bash
# Create namespace
kubectl apply -f namespaces.yaml

# Apply Deployment
kubectl apply -f nginx-deployment.yaml

# Check Pods in nginx namespace
kubectl get pods -n nginx

# Check ReplicaSets
kubectl get rs -n nginx

# Check Deployment
kubectl get deploy -n nginx
```

---

## ✅ Output Summary

* **Pods created**: 5
* **ReplicaSet created**: 1
* **Deployment created**: 1

```bash
kubectl get pods -n nginx
NAME                   READY   STATUS    RESTARTS   AGE
nginx-96b9d695-2sfpx   1/1     Running   0          17s
nginx-96b9d695-jg947   1/1     Running   0          17s
nginx-96b9d695-l8xhm   1/1     Running   0          17s
nginx-96b9d695-rrsfw   1/1     Running   0          17s
nginx-96b9d695-z9nbm   1/1     Running   0          17s
```

---

## ✅ Deployment Workflow Diagram

```
+-------------------------+
|   Deployment            |
| (nginx-deployment)      |
|  replicas: 5            |
+-------------------------+
          |
          v
+-------------------+
|   ReplicaSet      |
|  (nginx-96b9d695) |
+-------------------+
          |
          v
+-------------------+
|     5 Pods        |
| (nginx containers)|
+-------------------+
```

---

## ✅ Why Deployment?

* Provides **scaling**:

  ```bash
  kubectl scale deployment nginx-deployment --replicas=10 -n nginx
  ```
* Handles **rolling updates**:

  ```bash
  kubectl set image deployment/nginx-deployment nginx=nginx:1.21 -n nginx
  ```
* Ensures **self-healing** if Pods crash.

---
