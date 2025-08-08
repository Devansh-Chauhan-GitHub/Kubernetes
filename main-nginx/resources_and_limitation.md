
# Nginx Deployment with Resource Requests and Limits

This README explains the `nginx-deployment` configuration, focusing mainly on the **resource requests** and **limits** in Kubernetes.

## Deployment YAML

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 5
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
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi
```

---

## **Understanding Resource Requests and Limits**

In Kubernetes, **resources** define how much compute (CPU) and memory a container can use. The `resources` field inside a container specification allows you to define:

### **1. Requests**
- **Definition**: The minimum amount of CPU and memory the container is guaranteed.
- Kubernetes **schedules** the pod on a node only if the node has enough capacity for all requests.
- **In our case**:
  ```yaml
  requests:
    cpu: 100m
    memory: 128Mi
  ```
  - **`cpu: 100m`** → The container is guaranteed 0.1 CPU core.
  - **`memory: 128Mi`** → The container is guaranteed 128 MiB of memory.

---

### **2. Limits**
- **Definition**: The maximum amount of CPU and memory the container can use.
- If the container tries to exceed the memory limit, **it will be terminated** (OOMKilled).
- If it tries to exceed the CPU limit, **it will be throttled** (slowed down).
- **In our case**:
  ```yaml
  limits:
    cpu: 200m
    memory: 256Mi
  ```
  - **`cpu: 200m`** → The container can use at most 0.2 CPU core.
  - **`memory: 256Mi`** → The container can use at most 256 MiB of memory.

---

### **How Requests and Limits Work Together**
- **Requests** → Scheduling decision (guarantee).
- **Limits** → Runtime enforcement (cap).
- Example: If a node has 2 CPU cores and 4 GiB memory, Kubernetes can schedule multiple pods until the total requested resources fit.

---

## **Why Set Resource Requests and Limits?**
1. **Stability** → Prevents one pod from consuming all node resources.
2. **Fairness** → Ensures multiple applications can share resources properly.
3. **Predictability** → Avoids unpredictable slowdowns or crashes.
4. **Efficient Scheduling** → Kubernetes can better utilize cluster resources.

---

## **Summary Table**
| Resource Type | Purpose | Example in this Deployment |
|---------------|---------|----------------------------|
| **CPU Request** | Minimum guaranteed CPU for container | `100m` |
| **CPU Limit** | Maximum CPU allowed | `200m` |
| **Memory Request** | Minimum guaranteed memory | `128Mi` |
| **Memory Limit** | Maximum memory allowed before kill | `256Mi` |

---

## **Final Notes**
- Requests should be set to what your app **needs to run**.
- Limits should be set to what your app can use **without harming others**.
- Setting them too low → Pod may crash or be throttled.
- Setting them too high → Wastes cluster capacity.

