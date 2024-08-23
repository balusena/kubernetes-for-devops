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

### Scale Up and Scale Down in (HPA, VPA, CA) Autoscaling

#### 1. Scale Up
In autoscaling, "scale up" refers to increasing the number of pod replicas or resources allocated to existing pods to 
handle higher demand, thereby distributing the load more effectively.

- **Horizontal Pod Autoscaler (HPA)**
  - **Scale Up:** Increases the number of pod replicas based on metrics like CPU usage or memory, distributing the load 
    across more pods.

- **Vertical Pod Autoscaler (VPA)**
  - **Scale Up:** Increases resource requests (CPU and memory) for existing pods to handle higher loads, adjusting the 
    resources allocated to each pod.

- **Cluster Autoscaler (CA)**
  - **Scale Up:** Adds new nodes to the cluster when there are insufficient resources to schedule new pods, helping to 
    manage increased pod demands.
    
![Kubernetes Autoscaling ScaleUp](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/scaleup.png)  

#### 2. Scale Down
In autoscaling, "Scale down" involves reducing the number of pod replicas or resources when demand decreases, optimizing
resource usage and reducing costs.

- **Horizontal Pod Autoscaler (HPA)**
  - **Scale Down:** Decreases the number of pod replicas when metrics fall below the threshold, reducing resource consumption.

- **Vertical Pod Autoscaler (VPA)**
  - **Scale Down:** Decreases resource requests for existing pods when the load decreases, optimizing resource usage.

- **Cluster Autoscaler (CA)**
  - **Scale Down:** Removes underutilized nodes from the cluster to reduce costs when there are fewer pods requiring resources.
  
![Kubernetes Autoscaling ScaleDown](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/scaledown.png)

### Working examples of Autoscalers:

#### 1.Horizontal Pod Autoscaler (HPA):
Let's say our application is an e-commerce application and during a big billion sale, we get so many orders. When there 
is such unusual traffic, our application should be able to serve all the users without any downtime. For that, 

when the demand increases, the app should scale up, which means increasing the number of replicas to stay responsive.

![Kubernetes HPA ScaleUp](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_scaleup.png)

When the demand decreases, the app should scale down, which means decreasing the number of replicas to avoid wasting any
resources.

![Kubernetes HPA ScaleDown](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_scaledown.png)

But how does Kubernetes know when to scale up and when to scale down a deployment? For that, Kubernetes offers a resource
named Horizontal Pod Autoscaler (HPA). With HPA, we can instruct Kubernetes when and how to scale our deployment based on
specific metrics.

#### Horizontal Pod Autoscaler (HPA) Workflow:
Let's see how HPA works in Kubernetes. In the Kubernetes architecture, we have learned that on every worker node, kubelet
runs, and there is an agent in kubelet called cAdvisor (Container Advisor) by Google. When our pods are running on the 
worker node, cAdvisor can scrape the pod's CPU and memory usage every 10 seconds. Every minute, the metrics server aggregates
these metrics and exposes them to the Kubernetes API server. The HPA controller queries the API server every 15 minutes 
for these metrics. Once the controller gets the desired pod metrics, it checks them against our definitions and decides 
to scale up or scale down our replicas. The HPA controller updates the replica count in the target deployment. Spinning 
up new pods or deleting existing pods is managed by the replication controller, and cAdvisor starts collecting metrics 
from the new pods as well.

![Kubernetes HPA Worflow](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_workflow.png)







