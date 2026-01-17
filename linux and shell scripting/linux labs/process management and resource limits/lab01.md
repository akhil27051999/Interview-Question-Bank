## Process Management & Resource Limits

**In this scenario, you will learn how to diagnose, manage, and mitigate runaway processes that consume excessive CPU or memory.**

**As a system administrator or Kubernetes operator, it's important to quickly identify and handle resource-hogging processes to keep the system stable.**

### You will practice:
  1. Inspecting running processes with ps , top , and htop
  2. Identifying high CPU and memory consumers
  3. Terminating runaway processes safely
  4. Applying resource limits with ulimit , nice , and renice

### Scenario Setup:
**Some simulated runaway processes have been started in the background:**
  - yes (infinite CPU usage)
  - dd (large memory usage)
  - An infinite while loop (CPU hog)
> Your task is to diagnose and fix the system by controlling or terminating these processes.
