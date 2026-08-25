# What is Kubernetes?

Kubernetes is an orchestration platform that automates the deployment, management, and scaling of containerized applications across clusters of machines.

## Kubernetes Architecture

Kubernetes is installed on servers and organized into **clusters**. Each cluster consists of a **Master Node** (Control Plane) and one or more **Worker Nodes**.

### Master Node (Control Plane)
The Master Node manages all components within the cluster and makes global decisions about the cluster.

**Key Components:**
- **API Server** - The central hub of Kubernetes. All cluster interactions flow through the API Server, making it the entry point for all administrative tasks.
- **etcd** - A distributed key-value store that holds all cluster configuration data and state information. It serves as the cluster's "source of truth."
- **Scheduler** - Watches for newly created pods and assigns them to worker nodes based on resource requirements and constraints.
- **Controller Manager** - Runs controller processes that regulate the state of the cluster (e.g., maintaining desired replica counts, handling node failures).
- **Kubernetes Control Components** - Ensure the cluster runs according to the specified configuration.

### Worker Node
Worker Nodes are the machines where containerized applications actually run. They communicate status, metrics, and other information back to the Master Node.

**Key Components:**
- **kubelet** - An agent that runs on each worker node and communicates with the Master Node to ensure containers are running as expected.
- **Container Runtime** - Software that runs containers (e.g., Docker, containerd). Manages container lifecycle (create, start, stop, delete).
- **kube-proxy** - Maintains network rules and enables communication between pods and services within the cluster.

## Communication Flow

The **kubectl** command-line tool is used to interact with the Master Node's API Server. It allows administrators and applications to:
- Deploy and manage applications
- Query cluster status and pod information
- Ensure nodes are running according to the desired configuration
- View logs, execute commands, and debug applications

## Few Kubectl Commands

### `kubectl run hello-world`
**Explanation:** Creates and runs a pod named "hello-world" using a default image. This is the quickest way to deploy a simple containerized application to your Kubernetes cluster without writing [...]

---

### `kubectl cluster-info`
**Explanation:** Displays information about the Kubernetes cluster, including the API server endpoint, DNS service location, and other control plane details. Useful for verifying cluster connectiv[...]

---

### `kubectl get nodes`
**Explanation:** Lists all worker nodes in the cluster and their status (Ready, NotReady, etc.). This command gives you an overview of available compute resources and helps monitor node health.
