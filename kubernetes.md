# DevOps Interview Preparation Guide

A comprehensive collection of real-time and technical interview questions for DevOps roles.

Technologies Covered
- Kubernetes
- Docker
- Terraform
- AWS
- CI/CD
- Monitoring
- Linux & Shell Scripting
- Networking

---

## Table of Contents
- [Kubernetes](#kubernetes)
  - [Category 1: Fundamentals & Architecture](#category-1-fundamentals--architecture)
  - [Category 2: Configuration & Security](#category-2-configuration--security)
  - [Category 3: Networking & Services](#category-3-networking--services)
  - [Category 4: Storage](#category-4-storage)
  - [Category 5: Deployment & Scheduling](#category-5-deployment--scheduling)
  - [Category 6: Troubleshooting & Commands (Real-time Scenarios)](#category-6-troubleshooting--commands-real-time-scenarios)
  - [Category 7: Advanced Concepts](#category-7-advanced-concepts)

---

## Kubernetes

### Category 1: Fundamentals & Architecture

1. **What is Kubernetes and what are its main components?**  
Answer:  
Kubernetes is an open-source container orchestration platform for automating the deployment, scaling, and management of containerized applications.

Main Components:

- Control Plane (Master):
  - **kube-apiserver**: The front-end for the control plane; the only component you talk to directly (via kubectl)
  - **etcd**: A consistent and highly-available key-value store used as Kubernetes' backing store for all cluster data
  - **kube-scheduler**: Watches for newly created Pods with no assigned node and selects a node for them to run on
  - **kube-controller-manager**: Runs controller processes (e.g., Node Controller, Replication Controller)
  - **cloud-controller-manager**: Embeds cloud-specific control logic
- Worker Nodes:
  - **kubelet**: An agent that runs on each node, ensuring containers are running in a Pod
  - **kube-proxy**: Maintains network rules on the node, allowing network communication to your Pods
  - **Container Runtime**: The software that runs containers (e.g., Docker, containerd, CRI-O)

2. **What is the difference between a Pod, a Service, and a Deployment?**  
Answer:
- **Pod**: The smallest and simplest Kubernetes object. It represents a single instance of a running process in your cluster and can contain one or more containers.
- **Deployment**: A higher-level abstraction that manages the deployment and scaling of a set of Pods. It provides declarative updates, rolling updates, and rollbacks for Pods and ReplicaSets.
- **Service**: An abstraction that defines a logical set of Pods and a policy by which to access them. It provides a stable IP address and DNS name to decouple the front-end from the back-end Pods, which are ephemeral.

3. **Explain the Pod lifecycle.**  
Answer:  
A Pod's status field is a PodStatus object, which has a phase field. The main phases are:
- **Pending**: The Pod has been accepted by the cluster, but one or more containers have not been set up and made ready to run.
- **Running**: The Pod has been bound to a node, and all containers have been created. At least one container is still running, is in the process of starting, or is restarting.
- **Succeeded**: All containers in the Pod have terminated in success and will not be restarted.
- **Failed**: All containers in the Pod have terminated, and at least one container has terminated in failure.
- **Unknown**: The state of the Pod could not be obtained, typically due to an error in communicating with the node.

4. **What is etcd in Kubernetes and why is it important?**  
Answer:  
etcd is a strongly consistent, distributed key-value store. It is the "source of truth" for a Kubernetes cluster. It stores all cluster data, including:
- Cluster node information
- Pod definitions and status
- Secrets and ConfigMaps
- Service endpoints
- Namespaces

Its importance lies in its consistency and reliability. If etcd goes down or loses data, the cluster state is lost, leading to potential service outages.

---

### Category 2: Configuration & Security

5. **What is the difference between a ConfigMap and a Secret?**  
Answer:
- **ConfigMap**: Used to store non-confidential data in key-value pairs (e.g., configuration files, environment variables, command-line arguments). Data is stored as plain text.
- **Secret**: Used to store sensitive information, such as passwords, OAuth tokens, and SSH keys. Data is stored as base64-encoded strings (which is not encryption, but obfuscation). It's crucial to enable Encryption at Rest for Secrets in production.

6. **How do you secure a Kubernetes cluster? (Real-time question)**  
Answer:
- Use **RBAC**: Implement Role-Based Access Control to grant minimal necessary permissions to users and service accounts.
- **Network Policies**: Restrict pod-to-pod traffic within the cluster.
- **Secure etcd**: Encrypt etcd data at rest and use client certificate authentication (etcd peer-to-peer and client-to-etcd).
- **Update Regularly**: Keep Kubernetes, the container runtime, and the underlying OS patched.
- **Pod Security Policies/Standards**: Use Pod Security Standards (PSS) to restrict privileged pod creation.
- **Image Security**: Use trusted base images, scan for vulnerabilities, and avoid running as root.
- **TLS Everywhere**: Use TLS for all API communication.

7. **What are the types of Kubernetes probes and what is their use?**  
Answer:
- **Liveness Probe**: Determines if the container is running. If it fails, the kubelet kills the container, and it is subject to its restart policy. Use Case: To restart a container that is running but is in a broken state (e.g., a deadlock).
- **Readiness Probe**: Determines if the container is ready to accept requests. If it fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services. Use Case: To prevent sending traffic to a pod that is still starting up or is overloaded.
- **Startup Probe**: Used for slow-starting containers. It disables liveness and readiness checks until it succeeds. Use Case: For legacy applications that might require an additional startup time on their first initialization.

8. **What is the difference between requests and limits in a Pod's resources?**  
Answer:
- **requests**: The minimum amount of CPU/memory guaranteed to the container. The scheduler uses this to decide which node to place the Pod on.
- **limits**: The maximum amount of CPU/memory the container is allowed to use.

Behavior:
- **CPU**: A container cannot use more CPU than its limit. It will be throttled.
- **Memory**: A container using more memory than its limit will be terminated (OOMKilled).

---

### Category 3: Networking & Services

9. **How does Kubernetes networking work?**  
Answer:  
Kubernetes imposes these fundamental networking rules:
- Pods can all communicate with all other Pods without NAT.
- Nodes can communicate with all Pods without NAT.
- The IP that a Pod sees itself as is the same IP that others see it as.

This is typically achieved through a CNI (Container Network Interface) plugin (e.g., Calico, Flannel, Cilium) that sets up the overlay or underlying network to meet these requirements.

10. **What is a Kubernetes Service and what are its types?**  
Answer:  
A Service provides a stable network endpoint for a set of Pods.

- **ClusterIP**: Default type. Exposes the Service on a cluster-internal IP. Only reachable from within the cluster.
- **NodePort**: Exposes the Service on each Node's IP at a static port. Accessible from outside the cluster via <NodeIP>:<NodePort>.
- **LoadBalancer**: Exposes the Service externally using a cloud provider's load balancer. Creates a NodePort and ClusterIP automatically.
- **ExternalName**: Maps the Service to the contents of the externalName field (e.g., my-service.prod.svc.cluster.local -> api.example.com), by returning a CNAME record.

11. **What is the difference between ClusterIP and LoadBalancer?**  
Answer:  
- **ClusterIP** is for internal cluster communication.
- **LoadBalancer** is for exposing a service to the external internet, typically by provisioning an external cloud load balancer that routes traffic to the NodePort and ClusterIP services underneath it.

12. **What is an Ingress and why is it used?**  
Answer:  
An Ingress is an API object that manages external access to the services in a cluster, typically HTTP/HTTPS. It provides:
- Host-based routing: Route traffic to different services based on the host header (e.g., blog.example.com vs. app.example.com).
- Path-based routing: Route traffic based on the URL path (e.g., /api vs. /web).
- TLS termination: Secure the traffic by offloading SSL/TLS.

An Ingress requires an Ingress Controller (e.g., Nginx, Traefik, HAProxy) to function.

13. **What are Network Policies?**  
Answer:  
Network Policies are Kubernetes resources that control the flow of traffic between Pods. They act as a firewall. By default, all Pods are non-isolated (can receive traffic from any source). Once a Network Policy selects a Pod, that Pod becomes isolated, and only traffic allowed by the ingress/egress rules of the policy can reach it.

---

### Category 4: Storage

14. **How is storage managed in Kubernetes?**  
Answer:  
Kubernetes uses a volume abstraction. Key concepts:
- **Volume**: A directory accessible to containers in a Pod. Its lifetime is tied to the Pod.
- **PersistentVolume (PV)**: A cluster-wide storage resource provisioned by an administrator or dynamically using a StorageClass. It has a lifecycle independent of any individual Pod.
- **PersistentVolumeClaim (PVC)**: A user's request for storage. It is a claim on a PV's resources. Pods use PVCs to access persistent storage.
- **StorageClass**: Allows administrators to define "classes" of storage (e.g., "fast SSD," "slow HDD"). Dynamic provisioning uses StorageClasses to automatically create PVs when a PVC is created.

15. **What is the difference between emptyDir and hostPath?**  
Answer:
- **emptyDir**: A temporary directory that is created when a Pod is assigned to a node. It exists as long as the Pod is running on that node. Data is lost if the Pod is removed from the node. Used for scratch space or sharing files between containers in the same Pod.
- **hostPath**: Mounts a file or directory from the host node's filesystem into the Pod. This can be used for accessing node-level data (e.g., Docker internals) but is not portable and poses a security risk.

---

### Category 5: Deployment & Scheduling

16. **What is the difference between a Deployment and a StatefulSet?**  
Answer:
- **Deployment**: Used for stateless applications. Pods are treated as interchangeable, ephemeral units. Scaling and updates are done in any order.
- **StatefulSet**: Used for stateful applications that require stable, unique identities and stable, persistent storage.
  - Pods are created in a sequential, ordered fashion (ordinal index: web-0, web-1).
  - Stable network identity: web-0.nginx.default.svc.cluster.local.
  - Stable persistent storage: PVC for web-0 is always mounted to web-0, even if it gets rescheduled.

17. **How do you perform a rolling update and rollback with a Deployment?**  
Answer:
- **Rolling Update**: The default strategy for Deployments. It incrementally updates Pod instances with new ones. Triggered by updating the Pod template spec (e.g., `kubectl set image deployment/my-app nginx=nginx:1.16`).
- **Rollback**: Use `kubectl rollout undo deployment/my-app`. This will roll back to the previous revision. You can also specify a specific revision with `--to-revision`.

18. **What are Taints and Tolerations?**  
Answer:  
They are a mechanism to repel Pods from inappropriate nodes.
- **Taint**: Applied to a node. It has a key, value, and effect (e.g., NoSchedule, PreferNoSchedule, NoExecute). It marks the node to not accept any Pods that don't tolerate the taint.
- **Toleration**: Applied to a Pod. It "tolerates" a node's taint, allowing (but not guaranteeing) the scheduler to place the Pod on that node.

Use Case: Dedicate nodes for specific workloads (e.g., GPU nodes only for AI workloads).

19. **What are Node Affinity and Pod Affinity/Anti-Affinity?**  
Answer:
- **Node Affinity**: Constrains which nodes a Pod can be scheduled on based on node labels (e.g., "run this Pod only on nodes with the disktype=ssd label").
- **Pod Affinity**: Attracts Pods to other Pods (e.g., "run this Pod on a node that is already running a Pod with the label app=cache").
- **Pod Anti-Affinity**: Repels Pods from other Pods (e.g., "do not run two Pods with the label app=web on the same node" for high availability).

---

### Category 6: Troubleshooting & Commands (Real-time Scenarios)

20. **A Pod is stuck in Pending state. How do you debug it?**  
Answer:
- Check Pod details: `kubectl describe pod <pod-name>`
- Look at the Events section. Common reasons:
  - **Insufficient Resources**: "failed scheduling" due to insufficient CPU/Memory on any node.
  - **No Nodes Match Selector**: Pod's nodeSelector or affinity rules don't match any node.
  - **Taints**: The Pod doesn't have a toleration for a node's taint.
- Check Node Resources: `kubectl top nodes`
- Check Node Conditions: `kubectl get nodes` and `kubectl describe node <node-name>` to see if the node is Ready.

21. **A Pod is in CrashLoopBackOff state. What are the steps to debug?**  
Answer:  
CrashLoopBackOff means a container is repeatedly crashing after starting.

- Check Pod Logs: `kubectl logs <pod-name> [-c <container-name>]`
- Check Previous Logs: `kubectl logs <pod-name> --previous`
- Describe the Pod: `kubectl describe pod <pod-name>` for events and state details
- Debug by Executing: `kubectl exec -it <pod-name> -- /bin/sh` (if the container stays up long enough)
- Check Resource Limits: The container might be getting OOMKilled. Check `kubectl describe pod`.
- Verify Configuration: Check your ConfigMaps, Secrets, and command/args for errors.

22. **How do you check the health of your cluster?**  
Answer:
- `kubectl get componentstatuses` (kubectl get cs) - Check control plane components
- `kubectl get nodes` - Check if all nodes are Ready
- `kubectl get pods --all-namespaces` - Check the status of critical system Pods in kube-system namespace
- `kubectl top nodes` / `kubectl top pods` - Check resource utilization

23. **How do you curl a Service from within the cluster for testing?**  
Answer:  
Run a temporary debug Pod and use curl from inside it.

Example:
```bash
kubectl run -it --rm --image=curlimages/curl debug-pod -- sh
# Then inside the pod:
curl http://<service-name>.<namespace>.svc.cluster.local
```

24. **What is the command to see the CPU and Memory usage of Pods?**  
Answer:  
`kubectl top pods`. This requires the Metrics Server to be installed in the cluster.

---

### Category 7: Advanced Concepts

25. **What are Custom Resource Definitions (CRDs) and Operators?**  
Answer:
- **CRD**: Allows you to define your own custom resources (objects) in Kubernetes, extending the Kubernetes API. For example, you could create a CronTab resource.
- **Operator**: A pattern that uses CRDs. It is a custom controller that manages the entire lifecycle of a complex application (e.g., a database) in a Kubernetes-native way. It encodes human operational knowledge (backups, updates, scaling) into software.

26. **What are Init Containers?**  
Answer:  
Init containers are specialized containers that run before the app containers in a Pod. They must run to completion successfully before the main app containers are started. They are used for setup scripts, waiting for dependencies, or registering the Pod with a service discovery system.

27. **What is the Horizontal Pod Autoscaler (HPA) and how does it work?**  
Answer:  
HPA automatically scales the number of Pods in a deployment, replica set, or stateful set based on observed CPU utilization or other custom metrics.
- It queries the Metrics Server for resource usage.
- If the average CPU usage across all Pods exceeds the target (e.g., 80%), the HPA increases the number of replicas.
- It can also scale based on custom metrics provided by a custom metrics API.

28. **What is a DaemonSet?**  
Answer:  
A DaemonSet ensures that a copy of a Pod is running on all (or some) nodes in the cluster. As nodes are added/removed, the DaemonSet adds/removes Pods. Typical use cases:
- Cluster storage daemons (e.g., glusterd, ceph)
- Logs collection daemons (e.g., fluentd, filebeat)
- Node monitoring daemons (e.g., prometheus-node-exporter)

29. **What is the role of the Kubelet?**  
Answer:  
The Kubelet is an agent that runs on each worker node. Its primary responsibilities are:
- Registering the node with the API server
- Watching the API server for Pods that are scheduled to its node
- Instructing the container runtime to start/stop containers
- Running liveness and readiness probes
- Reporting the node and Pod status back to the API server

30. **What is the difference between a ReplicaSet and a Deployment?**  
Answer:
- **ReplicaSet**: A low-level controller that ensures a specified number of Pod replicas are running at any given time. It is used by Deployments.
- **Deployment**: A higher-level abstraction that manages ReplicaSets and provides declarative updates, rolling updates, and rollback functionality. You typically manage Deployments, not ReplicaSets, directly.

---

If you'd like, I can:
- Format the other technology sections (Docker, Terraform, AWS, CI/CD, Monitoring, Linux & Shell, Networking) into the same README structure.
- Add printable flashcards, PDF export, or a condensed cheat-sheet.
- Convert this into separate Markdown files per technology.

Let me know which next step you prefer.
