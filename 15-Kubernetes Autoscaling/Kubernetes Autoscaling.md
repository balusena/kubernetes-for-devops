# Kubernetes Autoscaling:
Kubernetes autoscaling dynamically adjusts the number of pods or nodes in a cluster based on resource usage or custom metrics.

- **Horizontal Pod Autoscaler (HPA):** Scales the number of pod replicas.
- **Vertical Pod Autoscaler (VPA):** Adjusts resource requests for individual pods.
- **Cluster Autoscaler (CA):** Adds or removes nodes to meet the cluster's resource demands.

This ensures efficient resource utilization,cost management,and high availability of applications based on workload needs.

In deployments, we have seen how to scale our deployments manually. This allows us to adjust the number of replicas 
whenever there is unusual traffic. However, monitoring traffic continuously and manually scaling our applications to 
handle such traffic spikes can be a tedious process. What if there were a way to monitor our pods and automatically 
scale them whenever there is an increase in CPU usage, memory, or other metrics like QPS (Queries Per Second)? This 
is called **Autoscaling**—scaling automatically based on specific metrics.

![Kubernetes Autoscaling Intro]()
  


