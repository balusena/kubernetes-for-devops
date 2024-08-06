# kubernetes Services
Kubernetes services provide a way to expose a set of Pods as a network service. Services abstract the
underlying Pods, ensuring communication even as Pods are dynamically created or deleted.

## 1.Types of Services

### 1.ClusterIP

ClusterIP is the default type of service in Kubernetes, exposing the service on an internal IP address 
within the cluster. This type of service is only reachable from within the cluster, making it ideal for 
internal communication between different components of an application. For example, a front-end service 
can communicate with a back-end service via ClusterIP without exposing the back-end to external traffic. 
This helps maintain security and simplifies network configurations by keeping internal traffic isolated.

### 2.Multiport

Multiport services in Kubernetes allow a single service to expose multiple ports, each potentially 
configured with different protocols. This is particularly useful for complex applications that require 
multiple ports to function correctly, such as a web server running on port 80 for HTTP traffic and port 443 
for HTTPS traffic. By defining multiple ports within the same service, Kubernetes simplifies the management 
and access of these ports, ensuring that all required ports are consistently available and correctly routed 
to the appropriate Pods.

### 3.NodePort

NodePort services expose the service on each Node's IP address at a static port, known as the NodePort. 
This makes the service accessible from outside the cluster by requesting `<NodeIP>:<NodePort>`. Kubernetes 
automatically creates a corresponding ClusterIP service to which the NodePort service routes traffic. NodePort is useful for development, testing, or environments where external access is needed but without the complexity of setting up a cloud load balancer. However, it requires manual handling of traffic routing and security considerations.

### 4.LoadBalancer

LoadBalancer services are designed for use with cloud providers that support external load balancers, such 
as AWS, GCP, or Azure. This type of service exposes the Kubernetes service externally, using the cloud 
provider's load balancer to distribute incoming traffic across multiple backend Pods. The LoadBalancer 
service automatically creates both NodePort and ClusterIP services, routing traffic through the NodePort. 
This setup is beneficial for production environments where reliable, high-availability access to services 
is required, as it efficiently manages and balances incoming traffic, ensuring scalability and resilience.

## 2.Benefits

Services in Kubernetes enable reliable network access and service discovery by maintaining stable endpoints 
while scaling and updating applications within a Kubernetes cluster. They abstract the underlying Pods, 
providing a stable interface for communication and ensuring consistent access even as the application 
dynamically scales. This abstraction simplifies the network configuration and enhances the security and 
manageability of applications, ensuring that internal and external traffic is appropriately routed and 
managed according to the service type.

## 3.How Kubernetes Services Works.

So far, we have seen deploying Nginx to Kubernetes using deployments, and we have also seen how to access 
the deployed Nginx with port forwarding in the same cluster (local-cluster).
```
ubuntu@balasenapathi:~$ kubectl port-forward deployment.apps/nginx-deployment 8081:80
Forwarding from 127.0.0.1:8081 -> 80
Forwarding from [::1]:8081 -> 80
```
**Note:** This is used for debugging purposes. But how do we access it from the production environment as 
end users?

**Use case-1:**
We have our Nginx up and running with two replicas, meaning two pods. When a pod is created, a new cluster 
private IP address is assigned to that pod. This is managed by the Kube Proxy Agent running on the node, as
discussed in Kubernetes architecture. These pods are created in a separate network, and we cannot reach these
pods from outside the cluster; we can only access them from within the cluster.

Let's say we want to connect to a service running in a pod from another pod. We can connect to it using the 
IP address of the pod, but we cannot access the same service from outside the cluster using the same 
IP address. Even within the cluster, we cannot rely on the IP address of the pod. Let us see why?

As we use deployments to create these pods, these pods are created and destroyed dynamically. If we decrease
the replica set count, extra pods will be deleted; if we increase the replica set count, extra pods will be 
created. Also, if we change the image version, all the pods get deleted and new pods get created. Whenever a
pod is recreated, a new IP address is assigned to that pod.

Pods are non-permanent resources, and this IP address keeps changing. Obviously, when we try to access with 
the old IP address, it fails to connect. Therefore, we cannot rely on their IP addresses to communicate. If 
we want to access the services running in a pod, it's a common use case to need access to the application on
the internet or from other services, especially in a microservices world. Port forwarding is just for 
debugging purposes to access on our local machine; it's not publicly available.

**Kubernetes Services Workflow:**

![Kubernetes Services Workflow](https://github.com/balusena/kubernetes-for-devops/blob/main/06-Kubernetes%20Services/services_workflow.png)




