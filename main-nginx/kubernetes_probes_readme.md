# Kubernetes Probes Documentation

This document explains the **three main types of probes in Kubernetes** — Liveness, Readiness, and Startup probes — and how they can be used inside a Deployment.

---

## **1. Liveness Probe**
**Purpose:**  
Checks if the container is still running and healthy.  
If the liveness probe fails, Kubernetes restarts the container.

**Example:**
```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```
- **httpGet**: Sends an HTTP request to `/` on port 80.  
- **initialDelaySeconds**: Wait 5 seconds before the first check.  
- **periodSeconds**: Perform the check every 10 seconds.

**When to use:**  
Use when you want to ensure your application does not run in a broken state indefinitely.

---

## **2. Readiness Probe**
**Purpose:**  
Checks if the container is ready to accept traffic.  
If it fails, the Pod is removed from the Service's endpoint list.

**Example:**
```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```
- Similar parameters as liveness probe.  
- This ensures traffic is only sent to the Pod when it’s ready.

**When to use:**  
Use when your app needs time to load data, warm up cache, or connect to dependencies before handling requests.

---

## **3. Startup Probe**
**Purpose:**  
Checks if the application inside the container has started.  
Useful for applications with slow startup times.

**Example:**
```yaml
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```
- **failureThreshold**: Allows up to 30 failed checks before considering the container failed.  
- **periodSeconds**: Checks every 10 seconds.

**When to use:**  
Use when the application has a long initialization process to avoid premature restarts.

---

## **Full Deployment Example with All Probes**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
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
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          startupProbe:
            httpGet:
              path: /
              port: 80
            failureThreshold: 30
            periodSeconds: 10
```

---

## **Summary Table**

| Probe Type      | Purpose | When it Fails |
|----------------|---------|--------------|
| **Liveness**   | Checks if app is running | Pod is restarted |
| **Readiness**  | Checks if app is ready to serve requests | Pod removed from Service endpoints |
| **Startup**    | Checks if app has started | Pod restarted after startup delay |
