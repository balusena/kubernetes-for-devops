# kubernetes Namespaces
Kubernetes namespaces are a way to logically partition a Kubernetes cluster into multiple virtual clusters. 
Each namespace provides a scope for names, ensuring that resource names within different namespaces do not 
conflict. This allows for better organization, resource allocation, and access control within the cluster. 
Namespaces are useful for isolating environments (e.g., dev, test, prod), managing complex deployments, and 
setting different policies for groups of users or teams. By default, Kubernetes provides "default," 
"kube-system," and "kube-public" namespaces, but custom namespaces can be created as needed for specific 
workloads or projects.

![Kubernetes Namespaces](https://github.com/balusena/kubernetes-for-devops/blob/main/08-Kubernetes%20Namespaces/kubernetes_namespaces.png)

When managing numerous Kubernetes resources across multiple applications, it can be challenging to maintain
them all effectively. To address this issue, there should be a way to organize these resources, enabling
different project teams or customers to share a single Kubernetes cluster. This is where Kubernetes 
namespaces come into play.

So far, we have created Pods, ReplicaSets, Deployments, Services, and Ingress. We will also create other 
resources like ConfigMaps, Secrets, etc., and multiple applications within the same cluster. When managing
multiple resources across various applications, there needs to be a way to organize them for various reasons.
This organization can be achieved by using namespaces.

Namespaces are a way to organize a cluster into virtual sub-clusters. We will create our resources in 
these namespaces instead of placing everything in a single namespace. A cluster can support any number 
of namespaces, and each namespace is logically separated from the others, but they can still communicate 
with each other.