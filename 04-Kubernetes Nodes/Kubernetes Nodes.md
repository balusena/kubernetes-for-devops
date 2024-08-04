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

## Types of Nodes

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

## Node Lifecycle

Nodes can have different states during their lifecycle:

- **Ready:** The node is healthy and ready to accept Pods.
- **NotReady:** The node is not accepting Pods due to a problem or maintenance.
- **Unknown:** The node status cannot be determined, typically due to a communication failure with the master node.

## Node Management

- **Scaling:** The number of nodes can be adjusted based on the workload demands. This can be done manually or using autoscalers.
- **Health Monitoring:** Kubernetes continuously monitors node health and will attempt to replace or reschedule Pods if a node fails.
- **Maintenance:** Nodes can be drained to safely remove Pods before performing maintenance tasks, ensuring minimal disruption to running applications.

## Node Overview

![Kubernetes Nodes](https://github.com/balusena/kubernetes-for-devops/blob/main/04-Kubernetes%20Nodes/kubernetes_nodes.png)

## Comprehensive list of commands

### 1.To view nodes in your cluster.
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
local-cluster       Ready    control-plane   98m   v1.27.3
local-cluster-m02   Ready    <none>          93m   v1.27.3
```
### 2.To get detailed information about a specific node.
```
$ kubectl describe node local-cluster
Name:               local-cluster
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=local-cluster
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  2023-08-04T10:00:00Z
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  local-cluster
  AcquireTime:     <unset>
  RenewTime:       2023-08-04T10:45:00Z
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                     Message
  ----             ------  -----------------                 ------------------                ------                     -------
  MemoryPressure   False   2023-08-04T10:44:00Z              2023-08-04T10:00:00Z              KubeletHasSufficientMemory kubelet has sufficient memory available
  DiskPressure     False   2023-08-04T10:44:00Z              2023-08-04T10:00:00Z              KubeletHasNoDiskPressure   kubelet has no disk pressure
  PIDPressure      False   2023-08-04T10:44:00Z              2023-08-04T10:00:00Z              KubeletHasSufficientPID    kubelet has sufficient PID available
  Ready            True    2023-08-04T10:44:00Z              2023-08-04T10:00:00Z              KubeletReady               kubelet is posting ready status
Addresses:
  InternalIP:  192.168.49.2
  Hostname:    local-cluster
Capacity:
  cpu:                2
  ephemeral-storage:  97610672Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             2045972Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  97610672Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             2045972Ki
  pods:               110
System Info:
  Machine ID:                 8b82bfdd5e5a46e8965ab3c917b8c087
  System UUID:                8b82bfdd-5e5a-46e8-965a-b3c917b8c087
  Boot ID:                    dfbf6a6d-1d10-4d5d-9057-0e90e90a8462
  Kernel Version:             5.4.0-80-generic
  OS Image:                   Ubuntu 20.04.2 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.7
  Kubelet Version:            v1.27.3
  Kube-Proxy Version:         v1.27.3
Non-terminated Pods:          (6 in total)
  Namespace                   Name                                      CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                      ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-74ff55c5b-fd5rd                   100m (5%)     100m (5%)   70Mi (4%)        170Mi (8%)     98m
  kube-system                 etcd-local-cluster                        100m (5%)     0 (0%)      100Mi (5%)       0 (0%)         98m
  kube-system                 kube-apiserver-local-cluster              250m (12%)    0 (0%)      0 (0%)           0 (0%)         98m
  kube-system                 kube-controller-manager-local-cluster     200m (10%)    0 (0%)      0 (0%)           0 (0%)         98m
  kube-system                 kube-proxy-8x9cv                          0 (0%)        0 (0%)      0 (0%)           0 (0%)         98m
  kube-system                 kube-scheduler-local-cluster              100m (5%)     0 (0%)      0 (0%)           0 (0%)         98m
Events:                       <none>
```
### 3.Cordon a Node - To mark the node as unschedulable.
```
ubuntu@balasenapathi:~$ kubectl cordon local-cluster
node/local-cluster cordoned
```
### 4.Uncordon a Node - To mark the node as schedulable.
```
ubuntu@balasenapathi:~$ kubectl uncordon local-cluster
node/local-cluster uncordoned
```
### 5.Drain a Node - To safely evict all pods from the node before marking it unschedulable.
```
ubuntu@balasenapathi:~$ kubectl drain local-cluster --ignore-daemonsets
node/local-cluster cordoned
evicting pod kube-system/kube-proxy-8mpxd
evicting pod kube-system/coredns-74ff55c5b-7sm4v
pod/kube-proxy-8mpxd evicted
pod/coredns-74ff55c5b-7sm4v evicted
```
### 6.Label a Node - To add a label to the node.
```
ubuntu@balasenapathi:~$ kubectl label nodes local-cluster environment=production
node/local-cluster labeled
```
### 7.Remove a Node Label - To remove a label from the node.
```
ubuntu@balasenapathi:~$ kubectl label nodes local-cluster environment-
node/local-cluster labeled
```
### 8.Taint a Node - To add a taint to the node and prevent pods from being scheduled unless they tolerate the taint.
```
ubuntu@balasenapathi:~$ kubectl taint nodes local-cluster key=value:NoSchedule
node/local-cluster tainted
```
### 9.Remove a Node Taint - To remove a taint from the node.
```
ubuntu@balasenapathi:~$ kubectl taint nodes local-cluster key=value:NoSchedule-
node/local-cluster untainted
```