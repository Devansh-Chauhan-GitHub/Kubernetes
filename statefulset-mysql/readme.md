# 📚 Kubernetes StatefulSet for MySQL with Persistent Storage

---

## ✅ Why StatefulSet?

A **StatefulSet** in Kubernetes is designed to manage **stateful applications**, where each pod:

* **Requires persistent storage** that survives restarts.
* Needs a **stable network identity**.
* Must follow **ordered deployment and termination**.

In contrast, Deployments are best suited for **stateless applications** where pod identity does not matter.

### **Key Features of StatefulSet**

* **Stable Pod names**: Pods get names like `mysql-statefulset-0`, `mysql-statefulset-1`, etc.
* **Ordered Rolling Updates**: Pods are started and terminated in sequence.
* **Persistent Volume Claims (PVCs)** per Pod for **data durability**.
* Works with **Headless Services** for DNS resolution.

---

## ✅ Why Persistent Storage for MySQL?

MySQL is a **database**, meaning it needs to **store data permanently**.

* If Pods restart without persistent storage, **all data is lost**.
* StatefulSet uses **volumeClaimTemplates** to dynamically create **PVCs** for each Pod.

---

## ✅ Architecture

```
Client Application → Headless Service (mysql-service) → StatefulSet (mysql pods) → PVC → PV
```

* **Headless Service**: Required for StatefulSet to manage DNS records of Pods.
* **StatefulSet**: Deploys MySQL pods with stable identity and persistent storage.
* **PVC & PV**: Ensure data persistence even after Pod crashes or reschedules.

---

## ✅ Kubernetes Resources Used

* **Namespace**: Logical grouping of MySQL resources.
* **Service**: Headless service for Pod discovery.
* **StatefulSet**: Runs MySQL pods with persistent volumes.

---

## ✅ Complete YAMLs

---

### 1. **Namespace**

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: mysql
```

✅ **Purpose**

* Isolates all MySQL resources inside the `mysql` namespace.

---

### 2. **Headless Service**

```yaml
kind: Service
apiVersion: v1
metadata:
  name: mysql-service
  namespace: mysql
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  clusterIP: None
```

✅ **Explanation**

* **clusterIP: None** → Makes it a **Headless Service**.
* Used by StatefulSet for **DNS-based Pod discovery**.
* Creates DNS like:

  ```
  mysql-statefulset-0.mysql-service.mysql.svc.cluster.local
  ```

---

### 3. **StatefulSet**

```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: mysql-statefulset
  namespace: mysql
spec:
  serviceName: mysql-service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:latest
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: root
            - name: MYSQL_DATABASE
              value: mysql
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

---

## ✅ Detailed Explanation of StatefulSet YAML

---

### **spec.serviceName**

* Tells StatefulSet to use **mysql-service** for stable DNS.
* Each Pod will be resolvable as:

  ```
  mysql-statefulset-0.mysql-service
  mysql-statefulset-1.mysql-service
  mysql-statefulset-2.mysql-service
  ```

---

### **replicas: 3**

* Deploy **3 MySQL Pods**, each with **its own PVC**.

---

### **selector & template.labels**

* Selector matches Pods with `app: mysql`.
* Template ensures Pods created have this label.

---

### **container section**

* **image: mysql**\*\*:latest\*\*

  * Official MySQL image from Docker Hub.
* **ports.containerPort: 3306**

  * MySQL default port.
* **env**

  * `MYSQL_ROOT_PASSWORD`: Sets root password for MySQL.
  * `MYSQL_DATABASE`: Creates a default database.
* **volumeMounts**

  * Mounts the volume at `/var/lib/mysql` (default MySQL data dir).

---

### **volumeClaimTemplates**

* Creates **one PVC per Pod** automatically.
* **accessModes: ReadWriteOnce**

  * Only one node can mount this volume.
* **resources.requests.storage: 1Gi**

  * Each PVC requests 1Gi of storage.

---

## ✅ StatefulSet Pod Identity & Storage Mapping

* Pods are named:

  ```
  mysql-statefulset-0
  mysql-statefulset-1
  mysql-statefulset-2
  ```
* Each Pod gets its **own PVC**:

  ```
  mysql-data-mysql-statefulset-0
  mysql-data-mysql-statefulset-1
  mysql-data-mysql-statefulset-2
  ```

---

## ✅ Deployment Steps

```bash
# Apply Namespace
kubectl apply -f namespaces.yaml

# Apply Headless Service
kubectl apply -f service.yaml

# Apply StatefulSet
kubectl apply -f statefulset.yaml
```

---

## ✅ Verify Resources

```bash
kubectl get ns
kubectl get pods -n mysql
kubectl get svc -n mysql
kubectl get pvc -n mysql
kubectl get pv
```

---

## ✅ Test MySQL Connection

```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- mysql -u root -p
# Enter password: root
```

---

## ✅ Why Headless Service in StatefulSet?

* Ensures **stable DNS entries** for each Pod.
* Required for **Pod-to-Pod communication** in stateful apps.

---

## ✅ Key Benefits of StatefulSet for MySQL

* **Data Persistence**: Pods retain their data.
* **Stable Network Identity**: Each Pod has its own DNS name.
* **Ordered Scaling**: Pods start and stop in sequence.
* **Supports Clustering**: Useful for MySQL clusters.

---

## ✅ Common Issues and Fixes

* **PVC Pending** → Add a StorageClass for dynamic provisioning.
* **Pods CrashLoopBackOff** → Check MySQL env variables.
* **Cannot connect** → Ensure service name matches StatefulSet `serviceName`.

---

## ✅ Best Practices

* Use **Secrets** for sensitive data like `MYSQL_ROOT_PASSWORD`.
* Configure **StorageClass** for dynamic PV provisioning.
* Add **liveness and readiness probes**.
* Enable **resource limits** for better scheduling.

---

## ✅ Summary

StatefulSet is the best choice for deploying **databases like MySQL** in Kubernetes because:

* It ensures **persistent storage per Pod**.
* Provides **stable DNS and identity**.
* Supports **scaling with persistence**.

---

