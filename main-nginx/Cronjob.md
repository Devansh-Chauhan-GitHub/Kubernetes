
# 📘 Kubernetes CronJob Documentation

---

## ✅ What is a CronJob?

A **CronJob** in Kubernetes is used to **run Jobs on a schedule**, similar to Linux `cron`.  
- It **creates Jobs automatically** based on a given schedule.
- Each Job spawns Pods to execute the defined task.
- Useful for **periodic tasks** like:
  - Backups
  - Report generation
  - Cleanup scripts

---

## ✅ Key Features
- **Runs periodically** using cron syntax.
- Each scheduled run creates a **Job**.

---

## ✅ Difference Between Job and CronJob

| Feature           | **Job**                               | **CronJob**                                |
|-------------------|--------------------------------------|-------------------------------------------|
| **Purpose**       | Run task once or a fixed number of times | Run task repeatedly based on a schedule  |
| **Trigger**       | Manual (`kubectl apply`)            | Automatic, based on `schedule`           |
| **Extra Field**   | `completions`, `parallelism` only   | Adds `schedule` and `jobTemplate`        |
| **Use Case**      | Data processing, batch jobs         | Backups, recurring reports               |

---

## ✅ CronJob YAML with Inline Comments

```yaml
kind: CronJob                      # Key difference: CronJob instead of Job
apiVersion: batch/v1
metadata:
  name: demo-cron-job              # Name of the CronJob resource
  namespace: nginx
spec:
  schedule: "*/2 * * * *"          # CRON syntax → runs every 2 minutes
                                   # Use https://crontab.guru to generate custom schedules
  jobTemplate:                     # Template for the Job that will run on schedule
    spec:
      completions: 1               # How many times each Job must complete
      parallelism: 1               # How many Pods can run in parallel for the Job
      template:                    # Pod template inside the Job
        metadata:
          name: cron-job-pod       # Name for the Pod
          labels:
            app: CronJob
        spec:
          containers:
            - name: printer-container
              image: busybox:latest
              command: ["sh","-c","echo Hello World"]  # Task to run
          restartPolicy: Never      # Required: Pods in Jobs/CronJobs should not restart
```

---

## ✅ Important Fields Explained:

- **`schedule`**:
  - Defines when the Job runs using **cron syntax**:
    - Format: `minute hour day-of-month month day-of-week`
    - Example:
      - `*/5 * * * *` → Every 5 minutes
      - `0 0 * * *` → At midnight every day
  - Use **[crontab.guru](https://crontab.guru)** to easily calculate cron expressions.

- **`jobTemplate`**:
  - Same structure as **Job**, but wrapped inside CronJob.

- **`restartPolicy: Never`**:
  - Ensures Pods don’t restart after success or failure.

---

## ✅ How It Works
**CronJob Controller** checks schedule → Creates a **Job** → Job creates Pods → Pods run commands.

---

## ✅ Diagram: CronJob → Job → Pod

```
+-----------+       +-----------+       +-----------+
|  CronJob  | --->  |   Job     | --->  |   Pod     |
+-----------+       +-----------+       +-----------+
(Scheduled)         (Manages Pods)      (Executes Task)
```

---

## ✅ Common Cron Schedules:
| Schedule         | Meaning                        |
|------------------|--------------------------------|
| `*/2 * * * *`    | Every 2 minutes              |
| `0 0 * * *`      | Every day at midnight        |
| `0 12 * * 1-5`   | At 12:00 PM, Mon–Fri        |

---

## ✅ Key Differences with Job YAML
- **CronJob adds:**
  - `schedule`
  - `jobTemplate` wrapper
- **Job runs once**, **CronJob runs periodically**.
