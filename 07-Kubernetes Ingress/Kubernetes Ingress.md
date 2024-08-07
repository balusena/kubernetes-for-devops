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

![Kubernetes LoadBalancer](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/kubernetes_loadbalancer.png)

**Note:** The AWS, Azure, GCP cloud ingress controller implementation differs a little.

**AWS:** The AWS Load Balancer Controller integrates with the AWS Elastic Load Balancer (ELB) to provide 
Ingress functionality. The ELB itself acts as the Ingress Controller.

**Azure:** The Azure Application Gateway Ingress Controller integrates with the Azure Application Gateway,
which handles the Ingress functions.

**GCP:** The GCP Ingress Controller integrates with Google Cloud Load Balancer, managing the Ingress 
functionality.

These cloud-specific implementations often provide additional features and tighter integration with their
respective cloud services.

# How Does Kubernetes Ingress work?

If we are a beginner and trying to understand ingress, there is possible confusion on how it works.

For example, We might think, hey, we created the ingress rules, but we are not sure how to map it to a 
domain name or route the external traffic to internal deployments.

We need to be very clear about two key concepts to understand that.

**1.Kubernetes Ingress Resource:** Kubernetes ingress resource is responsible for storing DNS routing rules
in the cluster.

**2.Kubernetes Ingress Controller:** Kubernetes ingress controllers (Nginx/HAProxy etc.) are responsible 
for routing by accessing the DNS rules applied through ingress resources.

![Kubernetes Ingress Resource Controller](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/kubernetes_ingress_resource_controller.png)

# Ingress & Ingress Controller Architecture

Here is the architecture diagram that explains the ingress & ingress controller setup on a kubernetes cluster.

It shows ingress rules routing traffic to two payment & auth applications

Now if we look at the architecture, it will make more sense and we will probably be able to understand how
each ingress workflow works.

![Kubernetes Ingress Ingress-Controller Architecture](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/kubernetes_ingress_ingress_controller_architecture.png)






