# kubernetes Resource Management:
Kubernetes resource management involves controlling the allocation of compute resources like CPU and memory to containers
within a cluster. It uses requests to reserve resources and limits to cap usage, ensuring efficient utilization. Resource
quotas can restrict overall usage across namespaces, while limits ranges control resource limits per container. Kubernetes
also supports auto-scaling to dynamically adjust resource allocation based on demand, maintaining optimal performance and
cost-efficiency in cloud-native environments.

![Kubernetes Resource Management](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/resource_management.png)

Once an application is scheduled onto a node, it starts consuming the node's resources. Since multiple applications can 
be deployed to a Kubernetes cluster, it's important to be cautious when deploying your application. This ensures that it
uses computing resources like CPU and memory wisely, performs well, and doesn't impact other applications.

## Key Things We Need to Understand:
- **1.Requesting CPU and Memory:** Guarantees resources for the application using requests.
- **2.Limiting CPU and Memory:** Restricts resource usage for the application using limits.
- **3.Quality of Service (QoS):** Ensures performance levels for applications based on resource requests and limits.
- **4.Setting Minimum, Maximum, and Default Resources for Pods in a Namespace:** Achieved through LimitRange.
- **5.Limiting Resources in a Namespace:** Managed using ResourceQuota.

## Computing Resources:
Every node has resources such as CPU, memory, and disk space. CPU and memory are referred to as compute resources. We 
can request these compute resources from a node for later use.

![Kubernetes Computing Resources CPU Memory](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/computing_resources_cpu_memory.png)

These computing resources, such as CPU and memory, are crucial for a node, and we should be particularly cautious when 
using them.

### 1.Requests:
In Kubernetes, requests define the minimum amount of CPU and memory that a container requires. These requests ensure 
that the container receives the guaranteed resources it needs to function properly. By specifying requests, you help  
the scheduler allocate resources effectively and avoid contention among containers for limited resources.

Let's say we have an application that requires a minimum of one CPU and 200 MB of memory to start. If this application 
is deployed on a node where sufficient resources are not available, it may not start.

![Kubernetes Requests 1](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/request_1.png)

What if there is a way to instruct Kubernetes to deploy this application or pod onto a node that has the minimum required
capacity? For every container, we can define the resources and request Kubernetes to deploy the application onto a node 
where the minimum CPU and memory are available by using resource requests. When we define the request, Kubernetes will 
guarantee to schedule the pod on a node where the minimum capacity is available. Once this pod is scheduled, it immediately
uses the resources of Node2, reducing the allocatable resources on Node2 to 400 MB of memory and 200m of CPU.

![Kubernetes Requests 2](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/request_2.png)

If there are no nodes with the minimum capacity defined in the pod, the pod will remain in a pending state until the 
required resources become available.

![Kubernetes Requests 3](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/request_3.png)

Once the resources become available on any node, the Kubernetes scheduler will schedule the pod onto that node. In short,
Kubernetes compares the resource requests defined in the pod with the available resources on each node in the cluster and
automatically assigns a suitable node to the pod. If we don't specify CPU or memory requests, we are implicitly indicating
that we don't have specific requirements for how much CPU time the process running in the container is allocated. In the 
worst-case scenario, the process may not receive any CPU time at all.

![Kubernetes Requests 4](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/request_4.png)

Let's say there is suddenly a memory leak in the first application, causing Pod1 to use a lot of memory. What happens to
the second application on that node, which is Pod3? Pod3 might get terminated due to insufficient memory if it doesn't 
have enough to function properly. However, if you had instructed Kubernetes to limit the amount of memory Pod1 could use,
this situation could have been avoided. Fortunately, Kubernetes provides a safeguard against this scenario by allowing 
you to set resource limits, which prevent a pod from using more memory than specified.

![Kubernetes Requests 5](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/request_5.png)





