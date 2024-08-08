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

## Organizing Resources
We have resources related to `nginx` and `todo` applications in the same namespace, and we also have 
`nginx-ingress-controller` resources that we created in the ingress session.

![Organizing Resources](https://github.com/balusena/kubernetes-for-devops/blob/main/08-Kubernetes%20Namespaces/organizing_resources.png)

**Note:**
Now, let's create two namespaces: `nginx` and `todo`. We will move all resources related to `nginx` to
the `nginx` namespace and all resources belonging to `todo-ui` and `todo-api` into the `todo` namespace.

## Namespaces Can Be Created in Two Ways:

- 1. Using `kubectl`
- 2. Using a configuration file

### 1.Creating a Namespace Using `kubectl`
```
ubuntu@balasenapathi:~$ kubectl create namespace nginx
namespace/nginx created
```

### 2.Listing All Namespaces
```
ubuntu@balasenapathi:~$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   5d1h
ingress-nginx     Active   4d23h
kube-node-lease   Active   5d1h
kube-public       Active   5d1h
kube-system       Active   5d1h
nginx             Active   70s
```
**Note:**
- You can see the four default namespaces: **default**, **kube-node-lease**, **kube-public**, and **kube-system**.
- The nginx namespace that we created is listed.
- You can also see the ingress-nginx namespace, which was created when we installed the nginx-ingress-controller.

It is always recommended to use configuration files instead of kubectl commands to create resources for 
better tracking and maintenance.

### 3.Creating Namespaces Using a Configuration File

Just like any other Kubernetes resource, a namespace is also a Kubernetes resource and requires 
`apiVersion`, `kind`, and `metadata` to be created. The `spec` field is optional for namespaces.

### 4.Getting the `apiVersion` of Namespaces in Kubernetes
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep namespace
namespaces            ns           v1            false        Namespace
```
### 5.Creating a Namespace Using a Configuration File
Create a configuration file named nginx-namespace.yaml.
```
ubuntu@balasenapathi:~$ nano nginx-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx
```
### 6.Applying the Configuration to the Cluster
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-namespace.yaml
Warning: resource namespaces/nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by 
kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl 
apply. The missing annotation will be patched automatically.
namespace/nginx configured
```
**Note:**
You might receive a warning because the namespace nginx already exists. In such cases, you need to delete 
the existing namespace first before applying the changes again.

### 7.Deleting the Existing Namespace
```
ubuntu@balasenapathi:~$ kubectl delete namespace nginx
namespace "nginx" deleted
```
### 8.Reapplying the Configuration
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-namespace.yaml
namespace/nginx created
```
### 9.Creating the `todo` Namespace
Create a Configuration File named `todo-namespace.yaml`.
```
ubuntu@balasenapathi:~$ nano todo-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: todo
```
### 10.Apply the configuration file to create the todo namespace.
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-namespace.yaml
namespace/todo created
```
### 11.Verify the Creation of the todo Namespace
```
ubuntu@balasenapathi:~$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   5d1h
ingress-nginx     Active   5d
kube-node-lease   Active   5d1h
kube-public       Active   5d1h
kube-system       Active   5d1h
nginx             Active   5m6s
todo              Active   8s
```
**Note:**
All resources created so far were placed in the default namespace because no specific namespace was 
mentioned during their creation. This is why they all appear in the default namespace by default.

### 12.Current Resources in the Cluster
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS      AGE
pod/nginx-deployment-7c4c499fd5-4n5jm   1/1     Running   1 (39m ago)   4d8h
pod/nginx-deployment-7c4c499fd5-7skqs   1/1     Running   1 (39m ago)   4d8h
pod/nginx-deployment-7c4c499fd5-nflzm   1/1     Running   1 (39m ago)   4d8h
pod/nginx-deployment-7c4c499fd5-rw4hx   1/1     Running   1 (39m ago)   4d8h
pod/todo-api-6665cbbf8d-r7pv2           1/1     Running   1 (39m ago)   4d5h
pod/todo-api-6665cbbf8d-vdlk8           1/1     Running   1 (39m ago)   4d5h
pod/todo-ui-64ccd9d555-24lv9            1/1     Running   1 (39m ago)   4d5h
pod/todo-ui-64ccd9d555-8w7rl            1/1     Running   1 (39m ago)   4d5h

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP    4d8h
service/nginx-service      ClusterIP   10.107.16.240   <none>        8082/TCP   4d8h
service/todo-api-service   ClusterIP   10.104.42.224   <none>        8080/TCP   4d5h
service/todo-ui-service    ClusterIP   10.100.26.224   <none>        3001/TCP   4d5h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           4d8h
deployment.apps/todo-api           2/2     2            2           4d5h
deployment.apps/todo-ui            2/2     2            2           4d5h

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       4d8h
replicaset.apps/todo-api-6665cbbf8d           2         2         2       4d5h
replicaset.apps/todo-ui-64ccd9d555            2         2         2       4d5h

```
**Note:** The above output consists of 

```
ubuntu@balasenapathi:~$ kubectl get all

# 1.Pods

NAME                                    READY   STATUS    RESTARTS      AGE
pod/nginx-deployment-7c4c499fd5-4n5jm   1/1     Running   1 (39m ago)   4d8h
pod/nginx-deployment-7c4c499fd5-7skqs   1/1     Running   1 (39m ago)   4d8h
pod/nginx-deployment-7c4c499fd5-nflzm   1/1     Running   1 (39m ago)   4d8h
pod/nginx-deployment-7c4c499fd5-rw4hx   1/1     Running   1 (39m ago)   4d8h
pod/todo-api-6665cbbf8d-r7pv2           1/1     Running   1 (39m ago)   4d5h
pod/todo-api-6665cbbf8d-vdlk8           1/1     Running   1 (39m ago)   4d5h
pod/todo-ui-64ccd9d555-24lv9            1/1     Running   1 (39m ago)   4d5h
pod/todo-ui-64ccd9d555-8w7rl            1/1     Running   1 (39m ago)   4d5h

# 2.Services

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP    4d8h
service/nginx-service      ClusterIP   10.107.16.240   <none>        8082/TCP   4d8h
service/todo-api-service   ClusterIP   10.104.42.224   <none>        8080/TCP   4d5h
service/todo-ui-service    ClusterIP   10.100.26.224   <none>        3001/TCP   4d5h

# 3.Deployments

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           4d8h
deployment.apps/todo-api           2/2     2            2           4d5h
deployment.apps/todo-ui            2/2     2            2           4d5h

# 4.ReplicaSets

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       4d8h
replicaset.apps/todo-api-6665cbbf8d           2         2         2       4d5h
replicaset.apps/todo-ui-64ccd9d555            2         2         2       4d5h
```
**Note:**
All resources currently exist in the default namespace because we did not specify a namespace during 
their creation. To organize these resources into the appropriate namespaces, such as nginx and todo, 
you'll need to update the resource definitions and apply them to the correct namespaces.





























