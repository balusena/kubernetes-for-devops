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


