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



