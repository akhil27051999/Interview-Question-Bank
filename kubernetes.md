# DevOps Interview Preparation Guide — Kubernetes (Reformatted)

This file contains the Kubernetes section of the DevOps Interview Preparation Guide, reformatted for readability and interview use.  
Each question is shown as a heading; the answer is clearly separated under a highlighted "Answer" subheading. Commands and examples are in fenced code blocks. Tables are used where helpful.

---

## Contents
- [Category 1: Fundamentals & Architecture](#category-1-fundamentals--architecture)
- [Category 2: Configuration & Security](#category-2-configuration--security)
- [Category 3: Networking & Services](#category-3-networking--services)
- [Category 4: Storage](#category-4-storage)
- [Category 5: Deployment & Scheduling](#category-5-deployment--scheduling)
- [Category 6: Troubleshooting & Commands (Real-time Scenarios)](#category-6-troubleshooting--commands-real-time-scenarios)
- [Category 7: Advanced Concepts](#category-7-advanced-concepts)

---

## Category 1: Fundamentals & Architecture

### Q1 — What is Kubernetes and what are its main components?
**Answer**

Kubernetes is an open-source container orchestration platform for automating the deployment, scaling, and management of containerized applications.

Main components:

| Control Plane (Master) | Description |
|---|---|
| kube-apiserver | The front-end for the control plane; the only component you talk to directly (via kubectl). |
| etcd | Strongly consistent distributed key-value store used as Kubernetes' backing store for cluster data. |
| kube-scheduler | Watches for newly created Pods with no assigned node and selects a node for them to run on. |
| kube-controller-manager | Runs controller processes (e.g., Node Controller, Replication Controller). |
| cloud-controller-manager | Embeds cloud-specific control logic (if running on cloud). |

| Worker Node Components | Description |
|---|---|
| kubelet | Agent that runs on each node, ensuring containers are running in a Pod. |
| kube-proxy | Maintains network rules on the node to allow network communication to Pods. |
| Container Runtime | Software that runs containers (e.g., Docker, containerd, CRI-O). |

---

### Q2 — What is the difference between a Pod, a Service, and a Deployment?
**Answer**

- **Pod**: The smallest and simplest Kubernetes object. Represents one or more containers that share storage/network and a spec for how to run them.  
- **Deployment**: Higher-level abstraction that manages Pods and ReplicaSets. Provides declarative updates, rolling updates, and rollbacks.  
- **Service**: Stable network endpoint that exposes a logical set of Pods and a policy to access them (provides stable IP and DNS name).

---

### Q3 — Explain the Pod lifecycle.
**Answer**

Pod phases (high-level):
- **Pending**: Pod accepted by the cluster, containers not yet created or started.  
- **Running**: Pod bound to a node and containers are created; at least one container is running or restarting.  
- **Succeeded**: All containers terminated successfully and will not be restarted.  
- **Failed**: All containers terminated and at least one terminated in failure.  
- **Unknown**: State cannot be obtained (e.g., node communication error).

---

### Q4 — What is etcd in Kubernetes and why is it important?
**Answer**

- **etcd** is a strongly consistent, distributed key-value store.
- It stores cluster state: node info, Pod specs, Secrets, ConfigMaps, Services, etc.
- It is the "source of truth"; losing etcd or its data can cause cluster state loss and outages.
- Protect etcd with backups, TLS, and encryption at rest in production.

---

## Category 2: Configuration & Security

### Q5 — What is the difference between a ConfigMap and a Secret?
**Answer**

- **ConfigMap**: Stores non-confidential configuration data as key-value pairs (plain text). Use for app configs, flags, env vars.  
- **Secret**: Stores sensitive data (passwords, tokens, keys). Data is base64-encoded in the API (not encrypted by default). In production enable encryption at rest for Secrets and restrict access via RBAC.

---

### Q6 — How do you secure a Kubernetes cluster? (Real-time question)
**Answer**

Key practices:
- Implement **RBAC**: grant least privilege to users and service accounts.  
- Use **Network Policies** to restrict pod-to-pod traffic.  
- **Encrypt etcd** at rest and require TLS for etcd client/peer connections.  
- Patch and update Kubernetes, container runtime, and OS regularly.  
- Enforce Pod Security Standards / Pod Security Admission to disallow privileged containers.  
- Use trusted base images and scan images for vulnerabilities.  
- Use TLS for all API and inter-component communication.  
- Audit logs, monitor control plane and kube-system components, and rotate credentials.

---

### Q7 — What are the types of Kubernetes probes and what is their use?
**Answer**

- **Liveness Probe**: Detects if container is alive. If it fails, kubelet kills the container (it will be restarted per the Pod restart policy). Use for detecting deadlocks.  
- **Readiness Probe**: Detects if container is ready to accept traffic. If it fails, the Pod is removed from Service endpoints. Use to prevent routing traffic to unready Pods.  
- **Startup Probe**: Used for slow-starting containers; disables liveness/readiness until startup succeeds. Useful for legacy apps with long startup times.

---

### Q8 — What is the difference between requests and limits in a Pod's resources?
**Answer**

- **requests**: Minimum resources guaranteed for scheduling. The scheduler uses requests to place Pods on nodes.  
- **limits**: Maximum resources the container may use.
  - CPU: container is throttled when exceeding limit.
  - Memory: container exceeding limit will be OOMKilled.

---

## Category 3: Networking & Services

### Q9 — How does Kubernetes networking work?
**Answer**

Kubernetes networking model requires:
- Every Pod can communicate with every other Pod without NAT.  
- Nodes can communicate with all Pods.  
- A Pod's IP is the same internally and externally (no NAT for Pod IPs).

A CNI plugin (Calico, Flannel, Cilium, etc.) implements the network (overlay or native) to satisfy these requirements.

---

### Q10 — What is a Kubernetes Service and what are its types?
**Answer**

A Service exposes a logical set of Pods and provides stable discovery.

Types:
- **ClusterIP** (default): Exposes service inside the cluster.  
- **NodePort**: Exposes service on a static port on every node.  
- **LoadBalancer**: Provisions an external cloud load balancer and exposes service externally.  
- **ExternalName**: Maps a service to an external DNS name (CNAME).

---

### Q11 — What is the difference between ClusterIP and LoadBalancer?
**Answer**

- **ClusterIP**: Internal-facing service accessible only within the cluster.  
- **LoadBalancer**: External-facing. A cloud provider allocates a public LB, routes traffic to NodePort/ClusterIP underneath.

---

### Q12 — What is an Ingress and why is it used?
**Answer**

- **Ingress**: Kubernetes API object to manage external HTTP/HTTPS access to services, providing host/path-based routing and TLS termination.  
- Requires an **Ingress Controller** (NGINX, Traefik, HAProxy, etc.) to implement rules.

---

### Q13 — What are Network Policies?
**Answer**

- **NetworkPolicy**: Kubernetes resource that controls ingress/egress traffic to/from Pods.  
- By default, Pods are non-isolated (allow all traffic). When a NetworkPolicy selects a Pod, it becomes isolated and only the specified traffic is permitted.

---

## Category 4: Storage

### Q14 — How is storage managed in Kubernetes?
**Answer**

- **Volume**: Directory accessible to containers in a Pod (lifecycle = Pod).  
- **PersistentVolume (PV)**: Cluster resource provisioned manually or dynamically; lifecycle independent of Pods.  
- **PersistentVolumeClaim (PVC)**: User's request for storage (claims a PV).  
- **StorageClass**: Defines different storage types and parameters for dynamic provisioning (e.g., fast SSD vs slow HDD).

---

### Q15 — What is the difference between emptyDir and hostPath?
**Answer**

- **emptyDir**: Ephemeral storage created when Pod is scheduled on a node; deleted when Pod dies. Good for scratch or sharing between containers in a Pod.  
- **hostPath**: Mounts a file or directory from the node's filesystem into the Pod. Non-portable and potentially insecure (used for node-level access).

---

## Category 5: Deployment & Scheduling

### Q16 — What is the difference between a Deployment and a StatefulSet?
**Answer**

- **Deployment**: For stateless apps. Pods are interchangeable, and scaling/updates are unordered.  
- **StatefulSet**: For stateful apps needing stable network identity and persistent storage. Pods have stable, unique IDs and ordered creation/termination. Useful for databases and clustered systems.

---

### Q17 — How do you perform a rolling update and rollback with a Deployment?
**Answer**

- Trigger rolling update by updating the Pod template (e.g., image). Example:
```bash
kubectl set image deployment/my-app my-container=my-image:v2
```
- Rollback:
```bash
kubectl rollout undo deployment/my-app
```
You can target a specific revision with `--to-revision`.

---

### Q18 — What are Taints and Tolerations?
**Answer**

- **Taint (node)**: Marks node with key/value and effect (NoSchedule, PreferNoSchedule, NoExecute) to repel Pods.  
- **Toleration (pod)**: Lets a Pod tolerate a node's taint and be scheduled there.

Use case: dedicate nodes (e.g., GPU nodes) or isolate workloads.

---

### Q19 — What are Node Affinity and Pod Affinity/Anti-Affinity?
**Answer**

- **Node Affinity**: Constrains Pods onto nodes matching node labels (e.g., `disktype=ssd`).  
- **Pod Affinity**: Co-locate Pods together (run on same node or zone).  
- **Pod Anti-Affinity**: Spread Pods across nodes (prevent colocating Pods for HA).

---

## Category 6: Troubleshooting & Commands (Real-time Scenarios)

### Q20 — A Pod is stuck in Pending state. How do you debug it?
**Answer**

Steps:
1. Describe the Pod:
```bash
kubectl describe pod <pod-name>
```
2. Check events for scheduling errors (insufficient resources, selector/affinity mismatch, no matching nodes).  
3. Inspect node resource usage:
```bash
kubectl top nodes
kubectl get nodes
kubectl describe node <node-name>
```
4. Verify taints/tolerations and node selectors/affinities.

---

### Q21 — A Pod is in CrashLoopBackOff state. What are the steps to debug?
**Answer**

1. Check logs:
```bash
kubectl logs <pod-name> [-c <container-name>]
kubectl logs <pod-name> --previous
```
2. Describe the Pod for events:
```bash
kubectl describe pod <pod-name>
```
3. Exec into the Pod (if possible):
```bash
kubectl exec -it <pod-name> -- /bin/sh
```
4. Check resource limits (OOMKilled), ConfigMaps/Secrets, entrypoint/cmd, and start-up dependencies.

---

### Q22 — How do you check the health of your cluster?
**Answer**

Useful commands:
```bash
kubectl get componentstatuses      # check control plane components (deprecated in some versions)
kubectl get nodes                  # check node Ready status
kubectl get pods --all-namespaces  # check system and application pods
kubectl top nodes                  # requires Metrics Server
kubectl top pods
```
Also check critical `kube-system` Pods and control plane logs.

---

### Q23 — How do you curl a Service from within the cluster for testing?
**Answer**

Run a temporary pod with curl:
```bash
kubectl run -it --rm --image=curlimages/curl debug-pod -- sh
# inside pod
curl http://<service-name>.<namespace>.svc.cluster.local
```

---

### Q24 — What is the command to see the CPU and Memory usage of Pods?
**Answer**

```bash
kubectl top pods
```
Requires Metrics Server installed in the cluster.

---

## Category 7: Advanced Concepts

### Q25 — What are Custom Resource Definitions (CRDs) and Operators?
**Answer**

- **CRD**: Extend the Kubernetes API by defining new resource types (e.g., `CronTab`).  
- **Operator**: A custom controller (often built around CRDs) that encodes operational knowledge for managing complex applications (backups, upgrades, scaling) in a Kubernetes-native way.

---

### Q26 — What are Init Containers?
**Answer**

Init containers run before app containers in a Pod and must complete successfully before the main containers start. Used for initialization tasks such as migrations, config bootstrapping, or waiting for external services.

---

### Q27 — What is the Horizontal Pod Autoscaler (HPA) and how does it work?
**Answer**

HPA scales the number of Pod replicas based on observed metrics (CPU or custom metrics). It queries the Metrics Server (or custom metrics API), compares current usage to target, and adjusts replicas accordingly.

---

### Q28 — What is a DaemonSet?
**Answer**

DaemonSet ensures a copy of a Pod runs on all (or selected) nodes. Useful for node-level services: log collection, node monitoring, storage daemons.

---

### Q29 — What is the role of the Kubelet?
**Answer**

Kubelet runs on every node and:
- Registers the node with the API server
- Watches the API for Pods scheduled to the node
- Instructs the container runtime to run/stop containers
- Runs liveness/readiness probes and reports status

---

### Q30 — What is the difference between a ReplicaSet and a Deployment?
**Answer**

- **ReplicaSet**: Ensures a specified number of replicas of a Pod are running. Lower-level controller.  
- **Deployment**: Manages ReplicaSets and provides declarative updates, rolling updates, and rollback. Typically you manage Deployments (not ReplicaSets) directly.

---

If you'd like, I can:
- Integrate this reformatted Kubernetes section into your main README (replacing the original).  
- Reformat the remaining technology sections (Docker, Terraform, AWS, CI/CD, Monitoring, Linux & Shell, Networking) in the same style.  
- Produce separate Markdown files per technology or create printable cheat sheets / flashcards.

Which would you like next?
