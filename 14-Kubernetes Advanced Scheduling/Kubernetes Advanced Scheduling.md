# Kubernetes Advanced Scheduling
Kubernetes advanced scheduling optimizes pod placement and resource management. Node Name and Node Selector assign pods 
to specific nodes or those with certain labels. Node Affinity provides flexible rules for node selection based on labels.
Pod Affinity/Anti-Affinity dictates how pods should be scheduled in relation to other pods. Taints and Tolerations control
pod placement on nodes with specific taints. Together, these features enhance resource efficiency, balance workloads, and
meet precise deployment needs.

![Kubernetes Advanced Scheduling](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/advanced_scheduling.png)

We know that the Kubernetes scheduler is responsible for scheduling pods onto nodes. But how does the Kubernetes scheduler
pick the most suitable nodes for a pod, and is it possible to customize this behavior? In this discussion, we will explore
how the Kubernetes scheduler works and how we can customize this behavior using:

- **NodeName**
- **NodeSelector**
- **Affinity**
  - **Node Affinity**
  - **Pod Affinity**
- **Taints and Tolerations**

## How Kubernetes Scheduler Works
In a Kubernetes cluster with many nodes, when a pod is created, the Kubernetes scheduler first filters the nodes to 
identify which ones are capable of running the pod. It then evaluates these feasible nodes by running a set of scoring 
functions, rating each node from 1 to 10. Finally, the scheduler selects the node with the highest score and assigns 
it to the pod.

![Kubernetes Scheduler Workflow 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_1.png)

The Kubernetes scheduler takes into account the pod's resource requirements and any customizations defined by us when 
scheduling the pod. Most of the time, we don't need to write custom scheduling logic, as the default scheduler effectively
handles this automatically.

![Kubernetes Scheduler Workflow 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_2.png)

For example, running our pods across multiple nodes ensures that if one node goes down, the pods on other nodes can 
still handle requests. 

![Kubernetes Scheduler Workflow 3](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_3.png)

![Kubernetes Scheduler Workflow 4](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_4.png)

Kubernetes handles rescheduling of deleted pods onto different nodes when using Deployments or StatefulSets. 

![Kubernetes Scheduler Workflow 5](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_5.png) 

However,there may be cases where we want to deploy our pods onto specific nodes,such as nodes with SSDs attached or nodes
within the same availability zone for frequently communicating pods.

To achieve this,we need to instruct the Kubernetes scheduler.There are several methods to guide Kubernetes in scheduling
pods onto specific nodes.

### 1.Create a minikube cluster with 4 nodes:
```
ubuntu@balasenapathi:~$ minikube start --nodes 4
😄  minikube v1.31.1 on Ubuntu 20.04
✨  Automatically selected the docker driver. Other choices: virtualbox, ssh
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node minikube-m02 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
🔎  Verifying Kubernetes components...

👍  Starting worker node minikube-m03 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2,192.168.49.3
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3
🔎  Verifying Kubernetes components...

👍  Starting worker node minikube-m04 in cluster minikube
🚜  Pulling base image ...
⌛  Another minikube instance is downloading dependencies... 
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2,192.168.49.3,192.168.49.4
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3,192.168.49.4
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
### 2.To verify the nodes in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
minikube       Ready    control-plane   17m     v1.27.3
minikube-m02   Ready    <none>          13m     v1.27.3
minikube-m03   Ready    <none>          9m33s   v1.27.3
minikube-m04   Ready    <none>          5m10s   v1.27.3
```

### Different methods to guide Kubernetes in scheduling pods onto specific nodes.
We have four nodes ready in our cluster, named `1.minikube`, `2.minikube-m02`, `3.minikube-m03`, and `4.minikube-m04`. 
Each node in the cluster has a unique name, and the simplest way to instruct Kubernetes to schedule a pod onto a specific
node is by using the node name.

![Kubernetes Node Name 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodename_1.png) 

![Kubernetes Node Name 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodename_2.png) 










