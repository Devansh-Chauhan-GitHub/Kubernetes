# 📘 Kubernetes Multi-App Setup with Ingress (NGINX + Notes App)

---

## ✅ Overview

We will deploy **two applications** inside the same namespace (`nginx`) and expose them using **Ingress Controller** for path-based routing:

* **NGINX App** → Accessible at `/nginx`
* **Notes App** → Accessible at `/`

### Components Used:

* **Namespace**
* **Deployment** (NGINX & Notes App)
* **Service** (ClusterIP for internal routing)
* **Ingress** (External entry point with path-based routing)
* **NGINX Ingress Controller** (Must be installed)

---

## ✅ Why Use Ingress?

* Without Ingress, you need **NodePort or LoadBalancer** for each Service → Complex.
* Ingress provides:

  * **Single external endpoint**
  * **Path/Host-based routing**
  * SSL/TLS termination

---

## ✅ Architecture Flow

```
Client → Ingress Controller → Ingress Rules → Service → Pods
```

---

## ✅ Full YAML Setup

### 1️⃣ Namespace

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: nginx
```

---

### 2️⃣ NGINX Deployment + Service

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
---
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

### 3️⃣ Notes App Deployment + Service

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: notes-app-deployment
  namespace: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app-pod
  template:
    metadata:
      labels:
        app: notes-app-pod
    spec:
      containers:
        - name: notes-app-pod
          image: trainwithshubham/notes-app-k8s
          ports:
            - containerPort: 8000
---
kind: Service
apiVersion: v1
metadata:
  name: notes-app-service
  namespace: nginx
spec:
  selector:
    app: notes-app-pod
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
  type: ClusterIP
```

---

### 4️⃣ Ingress Resource

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-notes-nginx
  namespace: nginx
  annotations: 
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /nginx
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
          - pathType: Prefix
            path: /
            backend:
              service:
                name: notes-app-service
                port:
                  number: 8000
```

---

## ✅ Detailed Explanation of Ingress

* **ingressClassName: nginx**

  * Tells Kubernetes which Ingress Controller should manage this rule.
  * If you have multiple controllers (NGINX, Traefik), this decides which one handles the routing.
* **nginx.ingress.kubernetes.io/rewrite-target: "/"**

  * Rewrites `/nginx` → `/` before sending to backend service.
* **rules → paths**

  * `/nginx` → `nginx-service` (port 80)
  * `/` → `notes-app-service` (port 8000)

---

## ✅ Why `ingressClassName` is Needed?

* Ingress resources only define rules.
* The actual routing is done by an **Ingress Controller** (NGINX, Traefik, etc.).
* If you have multiple controllers, Kubernetes uses **ingressClassName** to decide which controller processes your Ingress.

Check IngressClass:

```bash
kubectl get ingressclass
```

Example output:

```
NAME    CONTROLLER
nginx   k8s.io/ingress-nginx
```

---

## ✅ Testing Locally with Port-Forward

Since Services are `ClusterIP`, use **port-forward** to access Ingress locally:

```bash
kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 8080:80
```

Now open:

```
http://localhost:8080/nginx     → NGINX page
http://localhost:8080/          → Notes App
```

---

### ✅ How It Works Internally:

1. Request → `http://localhost:8080/nginx`
2. Port-forward forwards to **Ingress Controller** inside cluster (port 80).
3. Ingress Controller checks rules:

   * Path `/nginx` → Backend = `nginx-service`
4. Annotation rewrites path `/nginx` → `/`.
5. Controller sends traffic to Pod serving NGINX default page.

---

## ✅ Apply Commands:

```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml

# Verify
kubectl get all -n nginx
kubectl get ingress -n nginx
```

---

## ✅ Why You See NGINX Default Page?

* Because `GET /` maps to `/usr/share/nginx/html/index.html` inside the container.
* Annotation dropped the `/nginx` prefix → sent `/` to backend.

---

## ✅ Key Points:

✔ Single external endpoint for multiple apps
✔ Path-based routing with Ingress
✔ Must have Ingress Controller installed
✔ Use `kubectl port-forward` for local testing

---

