# Kubernetes Namespace Management

## ✅ Overview

Namespaces in Kubernetes allow you to logically separate and organize resources. This is useful for multi-team environments or isolating different stages (dev, staging, prod) within the same cluster.

---

## ✅ Cluster Status Check

```bash
kubectl get nodes
kubectl get ns
```

**Example Output:**

```
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   133d   v1.32.0
```

```
NAME              STATUS   AGE
default           Active   133d
kube-node-lease   Active   133d
kube-public       Active   133d
kube-system       Active   133d
```

---

## ✅ 1. Create a Namespace

### Method 1: Using CLI

```bash
kubectl create ns nginx
```

Verify:

```bash
kubectl get ns
kubectl get ns nginx
```

---

### Method 2: Using YAML

Create a file: **`namespaces.yaml`**

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: nginx
```

Apply:

```bash
kubectl apply -f namespaces.yaml
```

Check:

```bash
kubectl get ns
```

**Explanation of YAML:**

* `kind: Namespace` → Defines resource type.
* `apiVersion: v1` → Uses core API group.
* `metadata.name` → Sets namespace name.

---

## ✅ 2. Deploy Pod in Namespace

```bash
kubectl run nginx --image=nginx -n nginx
```

Check pods:

```bash
kubectl get pods -n nginx
```

**Wide Output:**

```bash
kubectl get pods -o wide -n nginx
```

---

## ✅ 3. Delete Pod and Namespace

Delete Pod:

```bash
kubectl delete pod nginx -n nginx
```

Delete Namespace:

```bash
kubectl delete namespace nginx
```

---

## ✅ 4. Context Switching Between Namespaces

Instead of typing `-n nginx` every time, set a default namespace for the current context:

```bash
kubectl config set-context --current --namespace=nginx
```

Now `kubectl get pods` will default to `nginx`.

Check current context:

```bash
kubectl config view --minify | grep namespace:
```

Reset to default:

```bash
kubectl config set-context --current --namespace=default
```

---

## ✅ 5. Key Commands Table

| Command                            | Description                        |
| ---------------------------------- | ---------------------------------- |
| `kubectl get ns`                   | List all namespaces                |
| `kubectl create ns <name>`         | Create namespace via CLI           |
| `kubectl apply -f namespaces.yaml` | Create namespace via YAML          |
| `kubectl get pods -n <namespace>`  | List pods in a namespace           |
| `kubectl delete ns <name>`         | Delete namespace and its resources |

---

