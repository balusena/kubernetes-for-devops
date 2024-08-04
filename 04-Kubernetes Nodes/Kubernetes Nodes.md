# Kubernetes Nodes
In Kubernetes, a **Node** is a fundamental component of the cluster architecture. It represents a single 
machine (physical or virtual) that runs containerized applications. Nodes are the worker machines in the 
Kubernetes cluster and are managed by the Kubernetes control plane.

A Pod always runs on a Node. A Node is a worker machine in Kubernetes and may be either a virtual or a 
physical machine, depending on the cluster. Each Node is managed by the control plane. A Node can have 
multiple pods, and the Kubernetes control plane automatically handles scheduling the pods across the 
Nodes in the cluster. The control plane's automatic scheduling takes into account the available resources
on each Node.

Every Kubernetes Node runs at least:

- Kubelet, a process responsible for communication between the Kubernetes control plane and the Node; 
  it manages the Pods and the containers running on a machine.

- A container runtime (like Docker) responsible for pulling the container image from a registry, unpacking
  the container, and running the application.

### Types of Nodes

1. **Master Node:**
    - **Purpose:** Manages the Kubernetes cluster and handles the orchestration and control of all worker nodes.
    - **Components:**
        - **API Server:** Exposes the Kubernetes API and serves as the entry point for all administrative tasks.
        - **Controller Manager:** Ensures the desired state of the cluster by managing controllers.
        - **Scheduler:** Schedules Pods onto available nodes based on resource requirements and constraints.
        - **etcd:** A distributed key-value store that holds the cluster’s state and configuration data.

2. **Worker Node:**
    - **Purpose:** Runs the containerized applications and provides the runtime environment for Pods.
    - **Components:**
        - **Kubelet:** An agent that ensures containers are running in a Pod. It communicates with the Kubernetes API server to manage and report the node’s status.
        - **Kube-Proxy:** Maintains network rules and manages load balancing for network traffic within the cluster.
        - **Container Runtime:** Software responsible for running containers, such as Docker or containerd.

### Node Lifecycle

Nodes can have different states during their lifecycle:

- **Ready:** The node is healthy and ready to accept Pods.
- **NotReady:** The node is not accepting Pods due to a problem or maintenance.
- **Unknown:** The node status cannot be determined, typically due to a communication failure with the master node.

### Node Management

- **Scaling:** The number of nodes can be adjusted based on the workload demands. This can be done manually or using autoscalers.
- **Health Monitoring:** Kubernetes continuously monitors node health and will attempt to replace or reschedule Pods if a node fails.
- **Maintenance:** Nodes can be drained to safely remove Pods before performing maintenance tasks, ensuring minimal disruption to running applications.

### Node Overview

![Kubernetes Nodes](https://github.com/balusena/kubernetes-for-devops/blob/main/04-Kubernetes%20Nodes/kubernetes_nodes.png)

### Example Commands

To view nodes in your cluster:
```sh
kubectl get nodes
