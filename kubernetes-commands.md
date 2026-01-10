# 📘 Kubernetes Commands Cheat Sheet with Explanations

## 🔍 Basic Cluster Info & Node Management

| Command | Description |
|--------|-------------|
| `kubectl version` | Display the version of the Kubernetes client and server. |
| `kubectl cluster-info` | Show cluster master and service endpoint URLs. |
| `kubectl get nodes` | List all nodes in the cluster. |
| `kubectl describe node <node-name>` | Show detailed information about a specific node. |


## 📦 Working with Pods

| Command | Description |
|--------|-------------|
| `kubectl get pods` | List all pods in the current namespace. |
| `kubectl get pods -A` | List pods across all namespaces. |
| `kubectl describe pod <pod-name>` | Get detailed info about a specific pod (events, containers, status). |
| `kubectl logs <pod-name>` | View logs from a container in a pod. |
| `kubectl exec -it <pod-name> -- /bin/bash` | Access a running container in the pod using bash. |
| `kubectl delete pod <pod-name>` | Delete a pod; new one will be created if managed by a controller. |



## 🚀 Deployments, ReplicaSets & StatefulSets

| Command | Description |
|--------|-------------|
| `kubectl get deployments` | List all deployments in the namespace. |
| `kubectl describe deployment <name>` | Show detailed info about a deployment. |
| `kubectl rollout status deployment <name>` | Monitor rollout progress. |
| `kubectl rollout undo deployment <name>` | Roll back to the previous deployment. |
| `kubectl get rs` | List ReplicaSets. |
| `kubectl get statefulsets` | List StatefulSets for stateful workloads. |



## 🌐 Services, Networking & Ingress

| Command | Description |
|--------|-------------|
| `kubectl get svc` | List all services (ClusterIP, NodePort, LoadBalancer). |
| `kubectl describe svc <name>` | Detailed info about a specific service. |
| `kubectl port-forward svc/<name> <local-port>:<target-port>` | Forward service port to your local machine. |
| `kubectl get endpoints` | Show internal Pod IPs backing a service. |



## 🔐 ConfigMaps & Secrets

| Command | Description |
|--------|-------------|
| `kubectl get configmaps` | List all ConfigMaps. |
| `kubectl describe configmap <name>` | View key-value pairs in a ConfigMap. |
| `kubectl get secrets` | List all Secrets. |
| `kubectl describe secret <name>` | Decode and display secret contents. |
| `kubectl get secret <name> -o yaml` | View raw YAML output of a Secret. |



## 🗂 Namespaces & Resource Management

| Command | Description |
|--------|-------------|
| `kubectl get ns` | List all namespaces. |
| `kubectl create ns <namespace>` | Create a new namespace. |
| `kubectl config set-context --current --namespace=<ns>` | Set default namespace for current context. |
| `kubectl get resourcequotas` | List ResourceQuotas applied to a namespace. |



## 📄 YAML & Resource Apply

| Command | Description |
|--------|-------------|
| `kubectl apply -f <file>.yaml` | Apply or update resources from a YAML file. |
| `kubectl delete -f <file>.yaml` | Delete resources defined in a YAML file. |
| `kubectl create -f <file>.yaml` | Create resources from YAML (fails if exists). |
| `kubectl edit <resource> <name>` | Edit live resource definition in your default editor. |
| `kubectl get all` | Get all resources (Pods, Services, etc.) in the current namespace. |



## 📊 Monitoring & Debugging

| Command | Description |
|--------|-------------|
| `kubectl top pods` | View CPU/memory usage of each pod (requires Metrics Server). |
| `kubectl top nodes` | View CPU/memory usage of cluster nodes. |
| `kubectl get events` | View recent cluster events (failures, restarts). |
| `kubectl describe <resource> <name>` | Drill into any resource to troubleshoot. |



## ⚙️ Jobs, CronJobs & Autoscaling

| Command | Description |
|--------|-------------|
| `kubectl get jobs` | List all jobs running in the cluster. |
| `kubectl get cronjobs` | View all cron jobs. |
| `kubectl get hpa` | Check HorizontalPodAutoscalers status. |



## 🔒 RBAC: Access & Security

| Command | Description |
|--------|-------------|
| `kubectl get serviceaccounts` | List service accounts in the namespace. |
| `kubectl get roles,rolebindings` | View role-based access in the current namespace. |
| `kubectl get clusterroles,clusterrolebindings` | View access roles cluster-wide. |



## 💡 Helpful Tricks & Tips

- ✅ Add `-n <namespace>` to specify namespace explicitly.
- ✅ Use `-o yaml` or `-o json` for machine-readable output.
- ✅ Use `kubectl explain <resource>` for field-level documentation.



## 📚 Practice Resources

- **K9s**: Terminal UI for Kubernetes.
- **Lens**: GUI Kubernetes dashboard.
- **Helm**: Use `helm install`, `helm upgrade` to manage charts.
- **Minikube / Kind**: Set up a local cluster for testing.

📌 **Pro Tip**: Always start with `kubectl get <resource>` → then `describe` → then `logs` or `exec` to debug!
