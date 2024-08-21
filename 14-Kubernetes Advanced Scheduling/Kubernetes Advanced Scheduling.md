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





