
# Kubernetes Job Documentation

## ✅ What is a Kubernetes Job?

A **Job** in Kubernetes runs one or more Pods **to completion**. Unlike Deployments or ReplicaSets (which keep Pods running), Jobs are used for **batch or short-lived tasks**.

---

## ✅ Key Features:
- Runs Pods until a specified number of **successful completions**.
- Supports **parallel execution** of Pods.
- Deletes Pods after completion (depending on retention settings).

---

## ✅ YAML for the Job

```yaml
kind: Job                # This resource is a Job
apiVersion: batch/v1     # API group for Job resources
metadata:
  name: demo-job         # Name of the Job
  namespace: nginx       # Namespace where the Job runs
spec:
  completions: 2         # Total successful runs required
  parallelism: 2         # Number of Pods to run in parallel
  template:              # Pod template definition
    metadata:
      labels:
        app: demo-job    # Labels for Pods created by this Job
    spec:
      containers:
        - name: demo
          image: busybox:latest
          command: ["sh","-c","echo Hello World"]  # Command to execute
      restartPolicy: Never  # Pods won't restart after completion
```

---

## ✅ Explanation of Fields

### **kind**
Defines the resource type: `Job`.

### **apiVersion**
`batch/v1` → API group for Jobs.

### **metadata**
Contains **name** and **namespace** for Job identification.

### **spec**
Core settings for the Job:
- **completions:**  
  How many times the Job should successfully complete.  
  *Example:* `completions: 5` means 5 Pods must finish successfully.

- **parallelism:**  
  How many Pods can run at the same time.  
  *Example:* `parallelism: 2` means only 2 Pods run in parallel.

- **template:**  
  Defines the Pod to run. Includes containers, image, and commands.

- **restartPolicy:**  
  Must be `Never` or `OnFailure` for Jobs. Here we use `Never`.

---

## ✅ Completions vs Parallelism Explained

- **completions** = Total work to be done (how many successful Pods needed).  
- **parallelism** = How many Pods run simultaneously (like number of workers).

### Example 1:
```
completions: 4
parallelism: 2
```
- Total Pods needed = 4  
- Only 2 Pods run together → takes 2 rounds.

---

### Example 2:
```
completions: 1
parallelism: 4
```
- Only 1 successful Pod is needed.  
- 4 Pods start together → first to finish wins, others are stopped.

---

✅ **In short:**  
- `completions` = Total tasks.  
- `parallelism` = Speed (workers running in parallel).  

---

## ✅ How It Works
1. The Job controller creates Pods using the template.
2. Runs Pods until **completions** is met.
3. Parallel execution is limited by `parallelism`.
4. Job is marked **Complete** when all required Pods succeed.

---

## ✅ Commands to Use

```bash
# Apply the Job
kubectl apply -f job.yaml

# Check Job status
kubectl get jobs -n nginx

# Get Pods created by the Job
kubectl get pods -n nginx

# Check logs of a Pod
kubectl logs <pod-name> -n nginx

# Delete the Job and its Pods
kubectl delete job demo-job -n nginx
```

---

## ✅ Diagram: Job → Pods

```
Job (completions=4, parallelism=2)
   |
   ├── Pod 1 (Running)
   ├── Pod 2 (Running)
   ├── Pod 3 (Pending - waits until Pod 1 finishes)
   └── Pod 4 (Pending)
```
