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

### 13.Creating Resources in Their Respective Namespaces

To create a resource in a specific namespace instead of the `default` namespace, you need to specify the 
namespace in the `metadata` section of the resource definition.

### 14.Create the `nginx-deployment.yaml` File
```
ubuntu@balasenapathi:~$ nano nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
  annotations:
    kubernetes.io/change-cause: "Updating to alipine version"
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:alpine
          ports:
            - containerPort: 80
```
### 15.Apply the Changes to the Cluster
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
**Note:**
The nginx-deployment was created successfully in the nginx namespace. Even though there was an existing 
nginx-deployment in the default namespace, it did not affect the new deployment because Kubernetes 
resources are isolated by namespaces. A resource created or modified in one namespace does not affect 
resources in other namespaces. This is why the deployment was created again rather than updated—the two 
deployments are in different namespaces.

### 16.Now get all the resources in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS       AGE
pod/nginx-deployment-7c4c499fd5-4n5jm   1/1     Running   1 (116m ago)   4d9h
pod/nginx-deployment-7c4c499fd5-7skqs   1/1     Running   1 (116m ago)   4d9h
pod/nginx-deployment-7c4c499fd5-nflzm   1/1     Running   1 (116m ago)   4d9h
pod/nginx-deployment-7c4c499fd5-rw4hx   1/1     Running   1 (116m ago)   4d9h
pod/todo-api-6665cbbf8d-r7pv2           1/1     Running   1 (116m ago)   4d7h
pod/todo-api-6665cbbf8d-vdlk8           1/1     Running   1 (116m ago)   4d7h
pod/todo-ui-64ccd9d555-24lv9            1/1     Running   1 (116m ago)   4d7h
pod/todo-ui-64ccd9d555-8w7rl            1/1     Running   1 (116m ago)   4d7h

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP    4d9h
service/nginx-service      ClusterIP   10.107.16.240   <none>        8082/TCP   4d9h
service/todo-api-service   ClusterIP   10.104.42.224   <none>        8080/TCP   4d7h
service/todo-ui-service    ClusterIP   10.100.26.224   <none>        3001/TCP   4d7h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           4d9h
deployment.apps/todo-api           2/2     2            2           4d7h
deployment.apps/todo-ui            2/2     2            2           4d7h

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       4d9h
replicaset.apps/todo-api-6665cbbf8d           2         2         2       4d7h
replicaset.apps/todo-ui-64ccd9d555            2         2         2       4d7h
```
**Note:** We should have two deployments: one in the default namespace and another in the nginx namespace.
The reason they are not both showing up here is that when we execute kubectl get all, it retrieves 
resources only from the default namespace. To view resources from a different namespace, you need to 
specify the namespace using the -n flag followed by the namespace name, e.g., nginx.

### 17.To get all the resources from nginx namespace:
```
ubuntu@balasenapathi:~$ kubectl get all -n nginx            
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c4c499fd5-29r9z   1/1     Running   0          72m
pod/nginx-deployment-7c4c499fd5-n5lzh   1/1     Running   0          72m
pod/nginx-deployment-7c4c499fd5-pgwrg   1/1     Running   0          72m
pod/nginx-deployment-7c4c499fd5-qq78x   1/1     Running   0          72m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           72m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       72m
```

### 18.If we want to get all the reources from all namespaces
```
ubuntu@balasenapathi:~$ kubectl get all --all-namespaces
NAMESPACE       NAME                                            READY   STATUS      RESTARTS       AGE
default         pod/nginx-deployment-7c4c499fd5-4n5jm           1/1     Running     1 (128m ago)   4d9h
default         pod/nginx-deployment-7c4c499fd5-7skqs           1/1     Running     1 (128m ago)   4d9h
default         pod/nginx-deployment-7c4c499fd5-nflzm           1/1     Running     1 (128m ago)   4d9h
default         pod/nginx-deployment-7c4c499fd5-rw4hx           1/1     Running     1 (128m ago)   4d9h
default         pod/todo-api-6665cbbf8d-r7pv2                   1/1     Running     1 (128m ago)   4d7h
default         pod/todo-api-6665cbbf8d-vdlk8                   1/1     Running     1 (128m ago)   4d7h
default         pod/todo-ui-64ccd9d555-24lv9                    1/1     Running     1 (128m ago)   4d7h
default         pod/todo-ui-64ccd9d555-8w7rl                    1/1     Running     1 (128m ago)   4d7h
ingress-nginx   pod/ingress-nginx-admission-create-bjgkn        0/1     Completed   0              5d1h
ingress-nginx   pod/ingress-nginx-admission-patch-fgwsx         0/1     Completed   0              5d1h
ingress-nginx   pod/ingress-nginx-controller-7799c6795f-g85mf   1/1     Running     2 (128m ago)   5d1h
kube-system     pod/coredns-5d78c9869d-9rhzb                    1/1     Running     2 (128m ago)   5d2h
kube-system     pod/etcd-ingress-cluster                        1/1     Running     2 (128m ago)   5d2h
kube-system     pod/kube-apiserver-ingress-cluster              1/1     Running     2 (128m ago)   5d2h
kube-system     pod/kube-controller-manager-ingress-cluster     1/1     Running     2 (128m ago)   5d2h
kube-system     pod/kube-proxy-dgdpn                            1/1     Running     2 (128m ago)   5d2h
kube-system     pod/kube-scheduler-ingress-cluster              1/1     Running     2 (128m ago)   5d2h
kube-system     pod/storage-provisioner                         1/1     Running     7 (126m ago)   5d2h
nginx           pod/nginx-deployment-7c4c499fd5-29r9z           1/1     Running     0              75m
nginx           pod/nginx-deployment-7c4c499fd5-n5lzh           1/1     Running     0              75m
nginx           pod/nginx-deployment-7c4c499fd5-pgwrg           1/1     Running     0              75m
nginx           pod/nginx-deployment-7c4c499fd5-qq78x           1/1     Running     0              75m

NAMESPACE       NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
default         service/kubernetes                           ClusterIP   10.96.0.1       <none>        443/TCP                      4d9h
default         service/nginx-service                        ClusterIP   10.107.16.240   <none>        8082/TCP                     4d9h
default         service/todo-api-service                     ClusterIP   10.104.42.224   <none>        8080/TCP                     4d7h
default         service/todo-ui-service                      ClusterIP   10.100.26.224   <none>        3001/TCP                     4d7h
ingress-nginx   service/ingress-nginx-controller             NodePort    10.108.13.27    <none>        80:31226/TCP,443:32390/TCP   5d1h
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   10.101.239.58   <none>        443/TCP                      5d1h
kube-system     service/kube-dns                             ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP       5d2h

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   5d2h

NAMESPACE       NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
default         deployment.apps/nginx-deployment           4/4     4            4           4d9h
default         deployment.apps/todo-api                   2/2     2            2           4d7h
default         deployment.apps/todo-ui                    2/2     2            2           4d7h
ingress-nginx   deployment.apps/ingress-nginx-controller   1/1     1            1           5d1h
kube-system     deployment.apps/coredns                    1/1     1            1           5d2h
nginx           deployment.apps/nginx-deployment           4/4     4            4           75m

NAMESPACE       NAME                                                  DESIRED   CURRENT   READY   AGE
default         replicaset.apps/nginx-deployment-7c4c499fd5           4         4         4       4d9h
default         replicaset.apps/todo-api-6665cbbf8d                   2         2         2       4d7h
default         replicaset.apps/todo-ui-64ccd9d555                    2         2         2       4d7h
ingress-nginx   replicaset.apps/ingress-nginx-controller-7799c6795f   1         1         1       5d1h
kube-system     replicaset.apps/coredns-5d78c9869d                    1         1         1       5d2h
nginx           replicaset.apps/nginx-deployment-7c4c499fd5           4         4         4       75m

NAMESPACE       NAME                                       COMPLETIONS   DURATION   AGE
ingress-nginx   job.batch/ingress-nginx-admission-create   1/1           58s        5d1h
ingress-nginx   job.batch/ingress-nginx-admission-patch    1/1           59s        5d1h
```
**Note:** We have two nginx deployments one is in default namespace and another is in nginx namespace

- Note: We can also use.
```
ubuntu@balasenapathi:~$ kubectl get all -A
```
## Moving `nginx-service` to the `nginx` Namespace

### 19.Create a file named `nginx-service.yaml`.
```
ubuntu@balasenapathi:~$ nano nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 8082
      targetPort: 80
```

### 20Apply the configuration file to create the nginx-service in the nginx namespace.
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-service.yaml
service/nginx-service created
```

### 21.To verify that the nginx-service is created in the nginx namespace.
```
ubuntu@balasenapathi:~$ kubectl get all -n nginx
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c4c499fd5-29r9z   1/1     Running   0          83m
pod/nginx-deployment-7c4c499fd5-n5lzh   1/1     Running   0          83m
pod/nginx-deployment-7c4c499fd5-pgwrg   1/1     Running   0          83m
pod/nginx-deployment-7c4c499fd5-qq78x   1/1     Running   0          83m

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/nginx-service   ClusterIP   10.101.212.181   <none>        8082/TCP   26s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           84m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       83m
```
**Note:**We can see that the nginx-service is created in the nginx namespace.

If you are actively working on this nginx application, it can be cumbersome to specify the -n nginx flag
for every command. By default, kubectl commands operate within the default namespace. To simplify this, 
you can change the current active namespace using tools like kubens, or directly with kubectl.

This command sets the current namespace to nginx, so you no longer need to specify -n nginx for subsequent kubectl commands
```
ubuntu@balasenapathi:~$ kubectl config set-context --current --namespace=nginx
Context "ingress-cluster" modified.
```

### 22.Now we can listdown the resources without specifying the namespace:
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c4c499fd5-29r9z   1/1     Running   0          106m
pod/nginx-deployment-7c4c499fd5-n5lzh   1/1     Running   0          106m
pod/nginx-deployment-7c4c499fd5-pgwrg   1/1     Running   0          106m
pod/nginx-deployment-7c4c499fd5-qq78x   1/1     Running   0          106m

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/nginx-service   ClusterIP   10.101.212.181   <none>        8082/TCP   22m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           106m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       106m
```
**Note:** We got the resources from nginx namespace instead of Default namespace.Because our current 
default namespace is set to nginx.

## Moving `todo` Resources to the `todo` Namespace

### 23.Create a file named `todo-ui-api.yaml`.
```
ubuntu@balasenapathi:~$ nano todo-ui-api.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-api
  namespace: todo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-api
  template:
    metadata:
      name: todo-api-pod
      labels:
        app: todo-api
    spec:
      containers:
        - name: todo-api
          image: balasenapathi/todo-api:1.0.2
          ports:
            - containerPort: 8082
          env:
            - name: "spring.data.mongodb.uri"
              value: "mongodb+srv://root:321654@cluster0.p9jq2.mongodb.net/todo?retryWrites=true&w=majority"
---
apiVersion: v1
kind: Service
metadata:
  name: todo-api-service
  namespace: todo
spec:
  selector:
    app: todo-api
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8082
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-ui
  namespace: todo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-ui
  template:
    metadata:
      name: todo-ui-pod
      labels:
        app: todo-ui
    spec:
      containers:
        - name: todo-ui
          image: balasenapathi/todo-ui:1.0.2
          ports:
            - containerPort: 80
          env:
            - name: "REACT_APP_BACKEND_SERVER_URL"
              value: "http://todo.com/api"
---
apiVersion: v1
kind: Service
metadata:
  name: todo-ui-service
  namespace: todo
spec:
  selector:
    app: todo-ui
  ports:
    - name: http
      port: 3001
      targetPort: 80
```

### 24.Apply the configuration file to create the todo resources in the todo namespace:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-api.yaml
deployment.apps/todo-api created
service/todo-api-service created
deployment.apps/todo-ui created
service/todo-ui-service created
```
**Note:** All our resources are created in todo namespace now.

## Accessing Services Across Namespaces

### 25.Viewing Resources

First, list the resources to confirm the setup:

```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c4c499fd5-29r9z   1/1     Running   0          123m
pod/nginx-deployment-7c4c499fd5-n5lzh   1/1     Running   0          123m
pod/nginx-deployment-7c4c499fd5-pgwrg   1/1     Running   0          123m
pod/nginx-deployment-7c4c499fd5-qq78x   1/1     Running   0          123m

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/nginx-service   ClusterIP   10.101.212.181   <none>        8082/TCP   39m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           123m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       123m
```
### 26.Accessing todo Service from nginx Pod
Enter the nginx Pod:
```
ubuntu@balasenapathi:~$ kubectl exec -it pod/nginx-deployment-7c4c499fd5-29r9z -- sh
/ # 
```
### 27.Attempt to Access the todo Service:
```
/ # curl todo-api-service:8080/api/todos
curl: (6) Could not resolve host: todo-api-service
```
**Note:** The request fails because the nginx pod is in a different namespace from the todo service.

### 28.Accessing the Service Using Fully Qualified Domain Name (FQDN):

Services in Kubernetes can be accessed using a DNS pattern that includes the service name, namespace, and
the .svc.cluster.local suffix.
```
/ # curl todo-api-service.todo.svc.cluster.local:8080/api/todos
[{"id":"64fb404160cc1b77a0bba513","title":"SBMK","completed":false,"createdAt":"2023-09-08T15:39:45.944+00:00"}]
```

**Note:** As we can see, we received the todo items. We don’t need to include ".svc.cluster.local" in the 
DNS name because Kubernetes DNS will automatically resolve the address. For example, you can use curl 
todo-api-service.todo:8080/api/todos. If your resources are in the same namespace, you don’t even need
to specify the namespace: simply use curl todo-api-service:8080/api/todos.

**The DNS name can be simplified by omitting the .svc.cluster.local suffix:**
```
/ # curl todo-api-service.todo:8080/api/todos
[{"id":"64fb404160cc1b77a0bba513","title":"SBMK","completed":false,"createdAt":"2023-09-08T15:39:45.944+00:00"}]
```
**Notes on Namespace Usage:**

**1.Accessing Services:** To access services across different namespaces, use the format service-name.namespace.svc.cluster.local:port.

**2.Default Namespace:** For fewer resources, using the default namespace might be sufficient.

**3.Multiple Teams/Environments:** For multiple teams or environments (e.g., dev and prod), separate namespaces are recommended.

**4.Role-Based Access Control (RBAC) and Resource Quotas:** For larger organizations, implement RBAC and resource quotas within namespaces.

**5.Enterprise Setup:** Large enterprises may use multiple clusters for better isolation and management.

**6.Deleting a Namespace:** Deleting a namespace will remove all resources within that namespace.


















































