# MySQL Kubernetes Deployment

This guide explains the deployment of a MySQL database in Kubernetes using **Namespace**, **ConfigMap**, **Secret**, **Service**, and **StatefulSet** along with YAML file explanations.

---

## 1. Namespace

**File:** `namespaces.yaml`
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: mysql
```
**Explanation:**
- Creates a new namespace called `mysql`.
- Namespaces help in logically grouping Kubernetes resources.

---

## 2. ConfigMap

**File:** `configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysqlconfigmap
  namespace: mysql
data:
  MYSQL_DATABASE: mysql
```
**Explanation:**
- Stores non-sensitive configuration data in key-value pairs.
- Here, we define the database name `MYSQL_DATABASE`.
- This can be injected into containers via environment variables.

---

## 3. Secret

**File:** `secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysqlsecret
  namespace: mysql
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdA==
```
**Explanation:**
- Stores sensitive information like passwords.
- `cm9vdA==` is the base64 encoding of `root`.
- You can generate it using:
```bash
echo "root" | base64
```

---

## 4. Service

**File:** `service.yaml`
```yaml
apiVersion: v1
kind: Service
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
```
**Explanation:**
- Exposes the MySQL StatefulSet on port `3306`.
- Uses the label selector `app: mysql` to route traffic.

---

## 5. StatefulSet

**File:** `statefulset.yaml`
```yaml
apiVersion: apps/v1
kind: StatefulSet
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
              valueFrom:
                secretKeyRef:
                  name: mysqlsecret
                  key: MYSQL_ROOT_PASSWORD
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: mysqlconfigmap
                  key: MYSQL_DATABASE
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
**Explanation:**
- Deploys MySQL with **3 replicas** (pods).
- Uses `mysql-service` for network identity.
- **Environment Variables:**
  - `MYSQL_ROOT_PASSWORD` comes from a **Secret** (`mysqlsecret`) where the key name matches the secret's key.
  - `MYSQL_DATABASE` comes from a **ConfigMap** (`mysqlconfigmap`) where the key matches the ConfigMap key.
- `volumeClaimTemplates` ensures each pod has its own persistent volume.

---

## 6. Deployment Order
Apply in the following order:
```bash
kubectl apply -f namespaces.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f service.yaml
kubectl apply -f statefulset.yaml
```

---

## 7. Verify Deployment
```bash
kubectl get pods -n mysql
kubectl get svc -n mysql
kubectl get pvc -n mysql
```

---

## Notes
- Ensure the **name** in `secretKeyRef` and `configMapKeyRef` exactly matches the name of the Secret and ConfigMap.
- `valueFrom` is used to inject environment variables from Secrets/ConfigMaps into containers.
