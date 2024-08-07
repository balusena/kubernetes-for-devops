# kubernetes Ingress
In Kubernetes, Ingress is an API object that acts as a front-end to route external traffic to services
running within the cluster. It serves as a gateway or entry point for external clients to access the services
running on the cluster nodes. In simple terms, Ingress exposes HTTP and HTTPS routes and manages the traffic
flow to different backend services based on a set of predefined rules.

Kubernetes Ingress is a resource to add rules to route traffic from external sources to the applications 
running in the kubernetes cluster.

# What is Kubernetes Ingress?

The literal meaning: Ingress refers to the act of entering.

It is the same in the Kubernetes world as well. Ingress means the traffic that enters the cluster and egress
is the traffic that exits the cluster.

![Kubernetes Ingress Egress](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/kubernetes_ingress_egress.png)

Ingress is a native Kubernetes resource like pods, deployments, etc. Using ingress, you can maintain the
DNS routing configurations. The ingress controller does the actual routing by reading the routing rules
from ingress objects stored in etcd.

understanding ingress with a high-level example.

Without Kubernetes ingress, to expose an application to the outside world, we have to add a service Type 
Loadbalancer to the deployments. Here is how it looks. (I have shown the nodePort just to show the 
traffic flow).

![Kubernetes NodePort](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/kubernetes_nodeport.png)

In the same implementation, with ingress,there is a reverse proxy layer (Ingress controller implementation)
between the load balancer and the kubernetes service endpoint.

Here is a very high-level view of ingress implementation.Later we will see a detailed architecture covering
all the key concepts.



