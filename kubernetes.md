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

# Category 1: Fundamentals & Architecture

## Q1 — What is Kubernetes and what are its main components?
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

## Q2 — What is the difference between a Pod, a Service, and a Deployment?
**Answer**

### 1. Pod
**Definition:**  The smallest and simplest unit in Kubernetes. A Pod represents a single instance of a running process and can contain one or more containers.

**Use Case:**  Running applications or services in containers.

**YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

### 2. Deployment

**Definition:**  Provides declarative updates to Pods and manages ReplicaSets. Supports rolling updates, rollbacks, and scaling.

**Use Case:** Rolling updates, rollback, and horizontal scaling of applications.

**YAML:**

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```
### 3. Service
**Definition:**  A `Service` provides a stable internal endpoint for accessing a group of Pods. The `ClusterIP` type is the default, and is only accessible from within the cluster.

**Use Case:**  Internal communication between microservices or applications.

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

---

## Q3 — Explain the Pod lifecycle.
**Answer**

Pod phases (high-level):
- **Pending**: Pod accepted by the cluster, containers not yet created or started.  
- **Running**: Pod bound to a node and containers are created; at least one container is running or restarting.  
- **Succeeded**: All containers terminated successfully and will not be restarted.  
- **Failed**: All containers terminated and at least one terminated in failure.  
- **Unknown**: State cannot be obtained (e.g., node communication error).

---

## Q4 — What is etcd in Kubernetes and why is it important?
**Answer**

- **etcd** is a strongly consistent, distributed key-value store.
- It stores cluster state: node info, Pod specs, Secrets, ConfigMaps, Services, etc.
- It is the "source of truth"; losing etcd or its data can cause cluster state loss and outages.
- Protect etcd with backups, TLS, and encryption at rest in production.

---

# Category 2: Configuration & Security

## Q5 — What is the difference between a ConfigMap and a Secret?
**Answer**

### ConfigMap
**Definition:**  Stores non-sensitive config data in key-value format. Useful for app configuration.

**Use Case:**  Pass environment variables or files to containers.

**YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  APP_ENV: production
  DB_HOST: db-service

```
#### Mount ConfigMap in Pod
Inject configuration into Pods via environment variables or volume mounts.

**YAML:**

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-using-configmap
spec:
  containers:
    - name: myapp
      image: myapp:latest
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: my-configmap
              key: APP_ENV
      volumeMounts:
        - mountPath: "/etc/config"
          name: config-volume
  volumes:
    - name: config-volume
      configMap:
        name: my-configmap
```

### Secrets

**Definition:** Stores sensitive information like credentials, tokens, SSH keys in base64 encoded format.
In production enable encryption at rest for Secrets and restrict access via RBAC.

**Use Case:** Inject secrets into Pods securely.

**YAML:**

```yaml

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```
#### Mount Secret in Pod

**Use Case:**
Use secrets as environment variables or files in containers.

**YAML:**

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-using-secret
spec:
  containers:
    - name: myapp
      image: myapp:latest
      env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
```
---

## Q6 — How do you secure a Kubernetes cluster? (Real-time question)
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

## Q7 — What are the types of Kubernetes probes and what is their use?
**Answer**

### Kubernetes Probes

**Definition:** 
Probes are health check mechanisms in Kubernetes used to determine whether a container is running, ready to serve traffic, or has started successfully.

**Use Case:** 
Ensure application reliability by restarting unhealthy containers, controlling traffic flow, and handling slow-starting applications automatically.

**Types of Probes:** Kubernetes supports three types of probes
- Liveness Probe
- Readiness Probe
- Startup Probe

### 1. Startup Probe
**Definition:** Checks whether a container has started successfully.

**Use Case:**  
  - Used for slow-starting applications
  - Prevents liveness/readiness probes from running too early
**Action on Failure:** Pod is restarted

**YAML:**
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```
### 2. Liveness Probe

**Definition:** Checks whether the container is still running and responsive.

**Use Case:** 
  - Detects deadlocks or hung applications
  - Restarts the container if it becomes unresponsive

**Action on Failure:** Container is restarted

**YAML**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5
```

### 3. Readiness Probe
**Definition:** Checks whether the container is ready to accept incoming traffic.

**Use Case:**  
  - Prevents traffic until the app is fully ready
  - Useful when the app depends on databases or external services
**Action on Failure:** Pod is removed from Service endpoints (no restart)

**YAML:**
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```

---

## Q8 — What is the difference between requests and limits in a Pod's resources?
**Answer**

### Resource Requests and Limits

**Definition:** Resource requests define the minimum CPU and memory required by a container, while limits define the maximum resources a container is allowed to consume.

**Use Case:** 
  - Helps the Kubernetes scheduler place Pods on nodes with sufficient resources
  - Prevents a single container from over-consuming CPU or memory
  - Ensures fair resource sharing in a cluster

### How Kubernetes Uses Requests and Limits

**Scheduler Behavior:**
  - Uses requests to decide which node can run the Pod
  - Ignores limits during scheduling

**Runtime Behavior:**
  - CPU limits → Container is throttled
  - Memory limits → Container is OOMKilled if exceeded

**Resource Configuration YAML**
```yaml
spec:
  containers:
    - name: app
      resources:
        requests:
          cpu: "500m"
          memory: "256Mi"
        limits:
          cpu: "1"
          memory: "512Mi"
```
**Key Points:**

1. CPU
  - 500m = 0.5 CPU core
  - Exceeding CPU limit → Throttling

2. Memory
  - Memory is not compressible
  - Exceeding memory limit → OOMKilled

# Category 3: Networking & Services

## Q9 — How does Kubernetes networking work?
**Answer**

### Kubernetes Networking

**Definition:** Kubernetes networking is the system that enables communication between Pods, Services, and external clients, ensuring seamless connectivity across the cluster.

**Core Networking Requirements (Kubernetes Model):**
  - Kubernetes follows a flat network model, which means:
  - Every Pod gets a unique IP address
  - Pods can communicate with any other Pod without NAT
  - Nodes can communicate with all Pods
  - Services provide stable access to Pods

### 1. Pod-to-Pod Networking

**Definition:** Allows Pods to communicate directly using their Pod IPs, even if they are on different nodes.

**How it Works:**
  - Implemented using a CNI (Container Network Interface) plugin
  - CNI assigns IP addresses and sets up routing

**Examples of CNI Plugins:**
  - Calico
  - Flannel
  - Weave
  - Cilium

**Use Case:**
  - Microservices communication
  - East-west traffic inside the cluster

### 2. Pod-to-Service Networking

**Definition:** Services provide a stable IP and DNS name to access a group of Pods.

**How it Works:**
  - Service creates a virtual IP (ClusterIP)
  - kube-proxy routes traffic to backend Pods
  - Load balancing is done at the Service level

**Traffic Flow:**
  ```txt
  Client → Service IP → kube-proxy → Pod
  ```

### 3. Pod-to-External Communication (Egress)

**Definition:** Allows Pods to communicate with external services outside the cluster.

**How it Works:**
  - Traffic is routed through the node’s network
  - NAT is applied using the node’s IP
  - Controlled using Network Policies (optional)

**Use Case:**
  - Accessing external APIs
  - Database hosted outside the cluster

### 4. External-to-Pod Communication (Ingress)

**Definition:** Allows external users to access applications running inside the cluster.

**How it Works:**
  - External traffic enters via:
    - NodePort
	- LoadBalancer
	- Ingress Controller
  - Routed to Services, then Pods

**Traffic Flow:**
  ```txt
  Client → LoadBalancer / Ingress → Service → Pod
  ```

### 5. DNS in Kubernetes

**Definition:** Kubernetes provides built-in DNS-based service discovery.

**How it Works:**
  - CoreDNS runs as a Pod
  - Services get DNS names like:
  
    ```sh
	my-service.my-namespace.svc.cluster.local
	```

**Use Case:** Pods communicate using service names instead of IPs

### 6. kube-proxy

**Definition:** A networking component that maintains network rules for Services.

**How it Works:**
  - Uses iptables or IPVS
  - Performs load balancing and traffic routing

**Use Case:** Routes traffic from Service IP to Pod IPs

### 7. Network Policies
**Definition:** Network Policies control which Pods can communicate with each other.

**Use Case:**
  - Enforce security boundaries
  - Allow/deny traffic based on labels, namespaces, ports
  - Note:
    - Requires a CNI plugin that supports Network Policies (e.g., Calico, Cilium)


## Q10 — What is a Kubernetes Service and what are its types?
**Answer**

### Kubernetes Service

**Definition:** A Service is an abstraction that exposes a set of Pods as a network service, providing a stable IP and DNS name.

**Use Case:**
  - Provides stable networking despite Pod restarts
  - Load balances traffic across multiple Pods
  - Decouples applications from Pod lifecycle changes

**Common Commands:**
```sh
kubectl get services
kubectl describe service <service-name>
```

### 1. ClusterIP Service

**Definition:** Default Service type that exposes the application internally within the cluster.

**Use Case:** 
  - Communication between internal services
  - Backend or internal microservices
  - Not accessible from outside the cluster

**Action:** Traffic is accessible only within the cluster

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 2. NodePort Service

**Definition:** Exposes the Service on a static port on each Node’s IP.

**Use Case:**
  - External access via <NodeIP>:<NodePort>
  - Useful for testing and quick access
  - Not recommended for production

**Action:** Traffic enters through the node’s IP and port

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

### 3. LoadBalancer Service

**Definition:** Exposes the Service externally using a cloud provider’s load balancer.

**Use Case:** 
  - Production-grade external access
  - Automatically provisions a cloud load balancer (AWS, GCP, Azure)

**Action:** Traffic is routed via an external load balancer

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

### 4. ExternalName Service

**Definition:** Maps a Service to an external DNS name instead of a ClusterIP.

**Use Case:**
  - Access external services (e.g., external DB, SaaS APIs)
  - No proxying—only DNS resolution

**Action:** Service resolves directly to an external DNS name

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: external.example.com
```

---

## Q11 — What is an Ingress and Egress and why is it used?
**Answer**

### Ingress

**Definition:** Ingress controls incoming network traffic from external clients to Services inside the Kubernetes cluster, typically for HTTP/HTTPS traffic.

**Use Case:**
  - Expose applications to the outside world
  - Route traffic based on hostnames and URL paths
  - Centralize SSL/TLS termination
  - Reduce the need for multiple LoadBalancer Services

**Action:**
  ```txt
  External traffic is routed → Ingress Controller → Service → Pod
  ```

**YAML:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

### Egress

**Definition:** Egress controls outgoing network traffic from Pods to external services or other Pods.

**Use Case:**
  - Restrict Pods from accessing unauthorized external endpoints
  - Allow access only to specific APIs, databases, or services
  - Improve security and compliance

**Action:** Pod traffic is allowed or blocked when leaving the Pod based on Egress rules.

**YAML:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-db
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 3306
```
#### Interview One-Liner

Ingress manages incoming traffic to the cluster, while Egress controls outgoing traffic from Pods for security and access control.


## Q12 — What are Network Policies?
**Answer**

### Network Policies

**Definition:** Network Policies are Kubernetes resources that control network traffic between Pods and/or namespaces, defining which connections are allowed or denied.

**Use Case:**
  - Restrict communication between Pods for security
  - Allow only specific Pods or namespaces to talk to each other
  - Enforce zero-trust networking inside the cluster

**Action:** Traffic is allowed only if explicitly permitted by a NetworkPolicy; all other traffic is denied (when policies are applied).

**YAML:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 80
```
#### Interview One-Liner

Network Policies act as firewalls for Pods, allowing fine-grained control over Pod-to-Pod communication.

# Category 4: Storage

## Q14 — How is storage managed in Kubernetes?
**Answer**

**Definition:** Kubernetes manages storage using abstractions like Volumes, PersistentVolumes (PV), and PersistentVolumeClaims (PVC) to provide persistent and dynamic storage for Pods, independent of Pod lifecycle.

**Use Case:** 
  - Store data that outlives Pod restarts
  - Mount shared storage between multiple Pods
  - Support dynamic provisioning via StorageClasses
  - Integrate with cloud storage providers (AWS EBS, GCP Persistent Disk, Azure Disk)

**Action:** Pods request storage via a PersistentVolumeClaim, which binds to a PersistentVolume. Kubernetes mounts the volume into the Pod at a specified path.

**YAML:**
```yaml
# PersistentVolume (PV)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data

---
# PersistentVolumeClaim (PVC)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
# Pod using PVC
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-pvc
spec:
  containers:
    - name: myapp
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: pvc-example
```
#### Interview One-Liner

Kubernetes abstracts storage via Volumes, PVs, and PVCs, allowing Pods to use persistent storage that survives restarts and can be dynamically provisioned.

## Q15 — What is the difference between emptyDir and hostPath?
**Answer**

### emptyDir

**Definition:**
A temporary directory created inside the Pod’s node for the lifetime of the Pod. It is empty when the Pod starts and deleted when the Pod is removed.

**Use Case:**
  - Share data between containers in the same Pod
  - Temporary scratch space or caching
  - Fast ephemeral storage for logs or intermediate files

**Action:** Data is stored on the node’s filesystem (or tmpfs if medium: Memory is used) and removed when the Pod terminates.

**YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-example
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```

### hostPath

**Definition:** Mounts a directory or file from the host node into the Pod. Data persists as long as the host exists.

**Use Case:**
  - Access host-level resources (logs, configuration, sockets)
  - Debugging or monitoring applications
  - Sharing data between Pods and host system

**Action:** Data is stored directly on the node’s filesystem and is not deleted when the Pod terminates.

**YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: /data
          name: host-storage
  volumes:
    - name: host-storage
      hostPath:
        path: /mnt/data
        type: Directory
```

#### Interview One-Liner

emptyDir is ephemeral storage tied to Pod lifetime, while hostPath maps a host node directory into a Pod, persisting beyond Pod termination.

# Category 5: Deployment & Scheduling

## Q16 — What is the difference between a Deployment and a StatefulSet?
**Answer**

### Deployment

**Definition:** A Deployment manages stateless applications, ensuring the desired number of Pod replicas are running, with automated rolling updates and rollbacks.

**Use Case:**
  - Stateless web applications or APIs
  - Scaling Pods up or down easily
  - Continuous delivery with rolling updates

**Action:**
Pods are interchangeable, identified by labels, and do not maintain persistent identity or storage.

**YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

### StatefulSet

**Definition:** A StatefulSet manages stateful applications, ensuring Pods have stable, unique identities and persistent storage across restarts.

**Use Case:**
  - Databases (MySQL, PostgreSQL, MongoDB)
  - Stateful services that require ordered deployment and stable network IDs
  - Applications needing persistent storage tied to each Pod

**Action:**
Pods are uniquely named and maintain stable network IDs and storage. Rolling updates are done in order.

**YAML:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db-statefulset
spec:
  serviceName: "db"
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: mysql
          image: mysql:8
          volumeMounts:
            - name: db-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: db-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 5Gi
```
#### Interview One-Liner

Deployments manage stateless Pods with interchangeable replicas, while StatefulSets manage stateful Pods with stable identities, ordering, and persistent storage.


## Q17 — How do you perform a rolling update and rollback with a Deployment?
**Answer**

### Rolling Update and Rollback with Deployment

**Definition:** 
  - A rolling update gradually replaces old Pods of a Deployment with new Pods without downtime.
  - A rollback restores the Deployment to a previous version if the update fails.

**Use Case:**
  - Deploy new application versions safely
  - Avoid downtime for stateless applications
  - Recover quickly if a new release causes issues

**Action:** 
  - Rolling Update: Pods are replaced incrementally according to maxUnavailable and maxSurge settings.
  - Rollback: Deployment reverts to a previous revision automatically or manually.

**YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
```

#### Rolling Update Commands:
  ```sh
  # Update image
  kubectl set image deployment/web-deployment nginx=nginx:1.22

  # Check rollout status
  kubectl rollout status deployment/web-deployment
  ```

#### Rollback Commands:

  ```sh
  # Rollback to previous revision
  kubectl rollout undo deployment/web-deployment

  # Rollback to a specific revision
  kubectl rollout undo deployment/web-deployment --to-revision=2
  ```

#### Interview One-Liner

Rolling updates replace Pods incrementally without downtime, and rollbacks restore the Deployment to a previous working revision.


## Q18 — What are Taints and Tolerations?
**Answer**

### Taints and Tolerations

**Definition:** 
  - Taints: Applied to nodes to repel certain Pods, preventing them from being scheduled unless the Pod tolerates the taint.
  - Tolerations: Applied to Pods to allow them to be scheduled on nodes with matching taints.

**Use Case:**
  - Reserve nodes for specific workloads (e.g., GPU workloads, critical services)
  - Prevent general Pods from being scheduled on special-purpose nodes
  - Ensure isolation and quality-of-service for workloads

**Action:** 
  - Nodes reject Pods that do not have a matching toleration
  - Pods with a matching toleration can be scheduled on the tainted node

**Taint Example:**

```sh
# Taint a node so only GPU workloads can run
kubectl taint nodes node1 gpu=true:NoSchedule
```
- Effect: NoSchedule prevents Pods without the gpu=true toleration from being scheduled on node1

**Toleration Example (Pod YAML):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: gpu-app
      image: nvidia/cuda:latest
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```
#### Types of Taint Effects:

| **Effect**         | **Meaning**                                                                           |
| ------------------ | ------------------------------------------------------------------------------------- |
| `NoSchedule`       | Pod is **not scheduled** unless it tolerates the taint                                |
| `PreferNoSchedule` | Kubernetes **avoids scheduling** Pods on this node if possible                        |
| `NoExecute`        | Pod is **evicted** if it doesn’t tolerate the taint, or **prevented from scheduling** |

#### Interview One-Liner

Taints repel Pods from nodes, and Tolerations allow Pods to tolerate those taints and get scheduled on the node.

---

## Q19 — What are Node Affinity and Pod Affinity/Anti-Affinity?

**Answer**

### Node Affinity and Pod Affinity/Anti-Affinity

**Definition:** 
- `Node Affinity`: Rules that constrain which nodes a Pod can be scheduled on, based on node labels.
- `Pod Affinity`: Rules that prefer or require Pods to be scheduled on the same node or zone as other Pods.
- `Pod Anti-Affinity`: Rules that prefer or require Pods to avoid being scheduled on the same node or zone as other Pods.

**Use Case:**

  - `Node Affinity`:
    - Schedule Pods to nodes with specific hardware (GPU, SSD, region)
	- Ensure critical workloads run on certain nodes

  - `Pod Affinity`:
    - Co-locate related services for low-latency communication
	- Group microservices that often interact

  - `Pod Anti-Affinity`:
    - Spread Pods across nodes for high availability
	- Avoid single points of failure

**Action:**
  - Scheduler considers these rules during Pod placement
  - Hard requirement → Pod won’t schedule unless condition is met 
    ```txt
	(requiredDuringSchedulingIgnoredDuringExecution)
	```

  - Soft preference → Scheduler tries to follow but may schedule elsewhere 
    ```txt
	(preferredDuringSchedulingIgnoredDuringExecution)
	```

**Node Affinity Example (Pod YAML):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: gpu-app
      image: nvidia/cuda:latest
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: gpu
                operator: In
                values:
                  - "true"
```

**Pod Affinity Example:**
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: frontend
        topologyKey: "kubernetes.io/hostname"
```
  - This schedules the Pod on the same node as other Pods with label app=frontend.

**Pod Anti-Affinity Example:**
```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: backend
          topologyKey: "kubernetes.io/hostname"
```
  - This prefers to schedule Pods away from other backend Pods, spreading them across nodes.

#### Interview One-Liner

Node Affinity controls which nodes a Pod can run on, Pod Affinity co-locates Pods, and Pod Anti-Affinity spreads Pods for high availability.

# Category 6: Troubleshooting & Commands (Real-time Scenarios)

## Q20 — A Pod is stuck in Pending state. How do you debug it?
**Answer**

### Debugging a Pod in Pending State

**Definition:** A Pod is in Pending when it has been accepted by the Kubernetes cluster but cannot be scheduled onto a node yet.

**Use Case:**
  - Ensure Pods are successfully scheduled and running
  - Identify resource, affinity, or configuration issues preventing scheduling

**Action:** 

  1. Check Pod Status and Events
  
    ```sh
	kubectl describe pod <pod-name>
	```
  - Look for Events section
  - Common messages:
    - 0/3 nodes are available: Insufficient cpu/memory
	- node(s) didn't match node selector
	- taint/toleration conflicts

  2. Check Node Resources

    ```sh
	kubectl get nodes -o wide
	kubectl describe node <node-name>
	```
    - Ensure nodes have enough CPU, memory
    - Verify ready status

  3. Check Resource Requests and Limits

    - Pods with high requests may not fit on any node
	- Adjust requests or scale nodes

  4. Check Node Affinity / Taints / Tolerations
	- Node labels must match nodeSelector or nodeAffinity
	- Pod must tolerate taints on nodes
  
  5. Check Namespace Quotas or LimitRanges

    ```sh
	kubectl get resourcequota -n <namespace>
	kubectl describe limitrange -n <namespace>
	```
	- Quotas may prevent new Pods from scheduling

  6. Check Scheduler Logs (Optional for deeper debugging)
    ```sh
	kubectl -n kube-system logs <kube-scheduler-pod-name>
	```

**Common Causes:**
| Cause                                 | Description                                               |
| ------------------------------------- | --------------------------------------------------------- |
| **Insufficient resources**            | CPU or memory requested by Pod exceeds available on nodes |
| **Taints / Tolerations mismatch**     | Pod does not tolerate node taints                         |
| **Node affinity / selector mismatch** | Pod cannot match node labels                              |
| **Resource quotas exceeded**          | Namespace or cluster limits prevent scheduling            |
| **All nodes NotReady**                | Nodes are unschedulable or cordoned                       |

#### Interview One-Liner
A Pod in Pending state usually indicates scheduling issues; check events, node resources, affinity, taints, and quotas to debug.

---

## Q21 — A Pod is in CrashLoopBackOff state. What are the steps to debug?
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

## Q22 — How do you check the health of your cluster?
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

## Q23 — How do you curl a Service from within the cluster for testing?
**Answer**

Run a temporary pod with curl:
```bash
kubectl run -it --rm --image=curlimages/curl debug-pod -- sh
# inside pod
curl http://<service-name>.<namespace>.svc.cluster.local
```

---

## Q24 — What is the command to see the CPU and Memory usage of Pods?
**Answer**

```bash
kubectl top pods
```
Requires Metrics Server installed in the cluster.

---

# Category 7: Advanced Concepts

## Q25 — What are Custom Resource Definitions (CRDs) and Operators?
**Answer**

- **CRD**: Extend the Kubernetes API by defining new resource types (e.g., `CronTab`).  
- **Operator**: A custom controller (often built around CRDs) that encodes operational knowledge for managing complex applications (backups, upgrades, scaling) in a Kubernetes-native way.

---

## Q26 — What are Init Containers?
**Answer**

Init containers run before app containers in a Pod and must complete successfully before the main containers start. Used for initialization tasks such as migrations, config bootstrapping, or waiting for external services.

---

## Q27 — What is the Horizontal Pod Autoscaler (HPA) and how does it work?
**Answer**

HPA scales the number of Pod replicas based on observed metrics (CPU or custom metrics). It queries the Metrics Server (or custom metrics API), compares current usage to target, and adjusts replicas accordingly.

---

## Q28 — What is a DaemonSet?
**Answer**

DaemonSet ensures a copy of a Pod runs on all (or selected) nodes. Useful for node-level services: log collection, node monitoring, storage daemons.

---

## Q29 — What is the role of the Kubelet?
**Answer**

Kubelet runs on every node and:
- Registers the node with the API server
- Watches the API for Pods scheduled to the node
- Instructs the container runtime to run/stop containers
- Runs liveness/readiness probes and reports status

---

## Q30 — What is the difference between a ReplicaSet and a Deployment?
**Answer**

- **ReplicaSet**: Ensures a specified number of replicas of a Pod are running. Lower-level controller.  
- **Deployment**: Manages ReplicaSets and provides declarative updates, rolling updates, and rollback. Typically you manage Deployments (not ReplicaSets) directly.

