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

![Kubernetes Autoscaling Intro](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/autoscaling.png)
  
Kubernetes can handle scaling not only for pods but also for the underlying infrastructure. If we're running on a cloud 
environment, Kubernetes can automatically spin up additional nodes when the existing nodes cannot accommodate more pods.
This ensures that our applications remain scalable and responsive, even during periods of high demand.

## Types of Autoscalers:
Kubernetes offers 3 types of Autoscalers.
- **1.HPA(Horizontal Pod Autoscaler)
- **2.VPA(Vertical Pod Autoscaler)
- **3.CA(Cluster Autoscaler)

### 1.HPA(Horizontal Pod Autoscaler):
The Horizontal Pod Autoscaler (HPA) increases the number of replicas whenever there is a spike in CPU, Memory, QPS, or 
other metrics, distributing the load among the pods. This process is known as scaling up. However, increasing the number
of replicas with HPA is not always the best solution. For example, to make a database handle more connections, we might 
need to increase the memory and CPU of existing pods instead of creating new ones.This can be achieved using the Vertical
Pod Autoscaler (VPA).

![Kubernetes Horizontal Pod Autoscaler](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa.png)

### 2.VPA(Vertical Pod Autoscaler):
The Vertical Pod Autoscaler (VPA) analyzes the resource usage of a deployment and adjusts them accordingly to handle the
load. With VPA, instead of creating new pods, we increase the resources (like CPU and memory) of existing pods.This allows
the application to handle higher demand without needing to scale out horizontally.

![Kubernetes Vertical Pod Autoscaler](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/vpa.png)

### 3.CA(Cluster Autoscaler):
Finally if there is no capacity in the cluster it doesnt make any sense to increase the number of pods with horizontal
pods autoscaler or increase the resources with VPA(Vertical Pod Autoscaler) as needed we should be able to add nodes
to the cluster automatically to accommodate the more number the more number of pods.CA(Cluster Autoscaler) can do that
job for us it adds the nodes to the cluster if there are any pods struck in the pending state because of lack of resources
in the cluster.

![Kubernetes Cluster Autoscaler](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/ca.png)
 
