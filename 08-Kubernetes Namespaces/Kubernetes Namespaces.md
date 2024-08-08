# kubernetes Namespaces

## What is Namespace?
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

## Need for Namespaces
Lets us see when do we need namespaces:

**1.Avoding Conflict:**
Let's say we define our service with the name todo-api-service, and it works fine. However, if someone else
uses the same name for their service, our service would get overridden, causing our application to break. 
This happens because we cannot use the same name for two different resources within a single namespace. 
With separate namespaces, we can use the same name for different resources in different namespaces. This 
way, teams can reuse the same resource names in different workspaces without any problems. Additionally, 
any modifications made to one resource will not affect other resources.

![Avoiding Conflict](https://github.com/balusena/kubernetes-for-devops/blob/main/08-Kubernetes%20Namespaces/avoding_conflict.png)

**2.Restricting Access:**
Let's say employees with different roles, such as developers and admins, are using our Kubernetes cluster.
We also have two different namespaces, dev and prod, within the same cluster. Everyone can create or modify
resources in the dev namespace, but we should not grant that freedom in the prod namespace, as changes 
could disrupt live applications. By limiting users and processes to specific namespaces, we can ensure that
only authorized users have access to resources in a given namespace.

![Restricting Access](https://github.com/balusena/kubernetes-for-devops/blob/main/08-Kubernetes%20Namespaces/restricting_access.png)

**3.Resource Limits:**
Namespaces also help in limiting resources for different applications. For example, let's say we have two 
teams deploying their applications in the same namespace. Initially, both applications run smoothly, but 
if app1 starts consuming excessive memory due to a memory leak, app2 may slow down because of the reduced 
memory available. This issue can be avoided by running the two apps in different namespaces and defining 
resource quotas for CPU and memory utilization. This ensures that each project or namespace has the 
resources it needs to operate efficiently.

![Resource Limits](https://github.com/balusena/kubernetes-for-devops/blob/main/08-Kubernetes%20Namespaces/resource_limits.png)

**4.Default Namespaces:**

When we create a Kubernetes cluster, we get four default namespaces:

- **Default**: Used for resources when a namespace isn't explicitly specified.
- **kube-node-lease**: Contains lease resources that send heartbeat signals from nodes.
- **kube-public**: Used for public resources and is open to all users with read-only access.
- **kube-system**: Reserved for objects created by the Kubernetes control plane.

### Default Namespace
By default, all resources created in the Kubernetes cluster are placed in the `default` namespace if no
namespace is explicitly mentioned. For example, resources like `nginx-deployment`, `nginx-service`, and
`nginx-ingress-rules` should be in the `default` namespace if we didn't specify a namespace during their
creation.

### kube-node-lease Namespace
If a node goes down in a cluster, the Pods on that node are recreated on a different healthy node. 
Kubernetes knows which node went down through lease objects in the `kube-node-lease` namespace. Every 
node has an associated lease object that sends heartbeats to the control plane, helping the cluster
monitor node availability and take action when failures are detected.

### kube-public Namespace
The `kube-public` namespace is used for public resources and is not recommended for regular use by users.
This namespace is open to all users with read-only access and is reserved for cluster-wide usage when 
resources need to be visible across the entire cluster.

### kube-system Namespace
The `kube-system` namespace is for objects created by the Kubernetes control plane.

### Custom Namespaces
In addition to these default namespaces, we can create custom namespaces, such as `todo`.

![Default Namespace](https://github.com/balusena/kubernetes-for-devops/blob/main/08-Kubernetes%20Namespaces/default_namespace.png)
