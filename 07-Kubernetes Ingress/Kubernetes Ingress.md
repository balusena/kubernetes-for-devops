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

To make use of the Ingress resource, Kubernetes requires an Ingress controller, which acts as the 
implementation of the Ingress rules. There are several Ingress controllers available, each with its own 
set of features and configuration options. Popular examples include NGINX Ingress Controller, Traefik, 
HAProxy Ingress, and Istio Gateway.

The Ingress controller runs as a pod in the Kubernetes cluster and listens for changes in the Ingress 
resources. When a new Ingress object is created or modified, the controller dynamically updates its 
configuration to reflect the desired routing rules. This dynamic nature allows for easy scaling and 
reconfiguration of the Ingress controller based on the cluster’s needs.

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

# Benefits of Using Kubernetes Ingress

**1.Routing flexibility:** Kubernetes Ingress offers advanced routing capabilities, allowing traffic to be 
directed to different services based on various criteria such as path, host, HTTP headers, and more. This
flexibility enables developers to build complex application architectures, route traffic to different 
microservices, and implement blue-green or canary deployments seamlessly.

*2.Load balancing:** Ingress controllers, which are components responsible for managing the Ingress resource,
provide built-in load balancing capabilities. They distribute incoming traffic across multiple instances of
a service, ensuring high availability and efficient resource utilization. This load balancing feature 
enhances application performance and scalability.

**4.TLS termination:** Ingress supports SSL/TLS termination, enabling secure communication between clients
and backend services. It allows the use of SSL certificates to encrypt the traffic, ensuring data 
confidentiality and integrity. This feature eliminates the need for each individual service to manage 
its own SSL certificate and simplifies the management of encryption at the cluster level.

**5.Namespace isolation:** Ingress resources are namespaced objects in Kubernetes, meaning they can be 
scoped to specific namespaces. This isolation enables different teams or applications within a cluster to
have their own dedicated Ingress configurations, preventing interference and maintaining security 
boundaries.

### 1.Create a new cluster in minikube with name ingress-cluster.
```
ubuntu@balasenapathi:~$  minikube start -p ingress-cluster --driver=docker
😄  [ingress-cluster] minikube v1.31.1 on Ubuntu 20.04
✨  Using the docker driver based on user configuration
📌  Using Docker driver with root privileges
👍  Starting control plane node ingress-cluster in cluster ingress-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🎉  minikube 1.31.2 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.31.2
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
▪ Generating certificates and keys ...
▪ Booting up control plane ...
▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
❗  Executing "docker container inspect ingress-cluster --format={{.State.Status}}" took an unusually long time: 18.000221506s
💡  Restarting the docker service may improve performance.
▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "ingress-cluster" cluster and "default" namespace by default
```

### 2.We are using this nginx-deployment file in new cluster for our deployment.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    kubernetes.io/change-cause: "Updating to alpine version"
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
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7c4c499fd5-4n5jm   1/1     Running   0          11s
nginx-deployment-7c4c499fd5-7skqs   1/1     Running   0          11s
nginx-deployment-7c4c499fd5-nflzm   1/1     Running   0          11s
nginx-deployment-7c4c499fd5-rw4hx   1/1     Running   0          11s
```

### 3.We are using below nginx-service file to create service in the cluster.
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 8082
      targetPort: 80
```
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-service.yaml
service/nginx-service created
```
### 4.To get the information about all services running in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP    4m
nginx-service   ClusterIP   10.107.16.240   <none>        8082/TCP   12s
```

### 5.To get the information about all pods,services,deploments,replicasets.
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c4c499fd5-4n5jm   1/1     Running   0          2m29s
pod/nginx-deployment-7c4c499fd5-7skqs   1/1     Running   0          2m29s
pod/nginx-deployment-7c4c499fd5-nflzm   1/1     Running   0          2m29s
pod/nginx-deployment-7c4c499fd5-rw4hx   1/1     Running   0          2m29s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP    4m59s
service/nginx-service   ClusterIP   10.107.16.240   <none>        8082/TCP   71s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           2m29s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c4c499fd5   4         4         4       2m29s
```

**Note:** Now let us accesses nginx with ingress as we discussed we need an ingress-controller to process
the ingress rules and now deploy an ingress-controller in our kubernetes cluster.

# Setting Up Kubernetes Ingress and Ingress controller

### 1.Create a new cluster in minikube with name ingress-cluster.
```
ubuntu@balasenapathi:~$ minikube start -p ingress-cluster --driver=docker
😄  [ingress-cluster] minikube v1.31.1 on Ubuntu 20.04
✨  Using the docker driver based on user configuration
📌  Using Docker driver with root privileges
👍  Starting control plane node ingress-cluster in cluster ingress-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🎉  minikube 1.31.2 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.31.2
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
▪ Generating certificates and keys ...
▪ Booting up control plane ...
▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
❗  Executing "docker container inspect ingress-cluster --format={{.State.Status}}" took an unusually long time: 18.000221506s
💡  Restarting the docker service may improve performance.
▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "ingress-cluster" cluster and "default" namespace by default
```

**2.Setup the Ingress Controller:**

There are many third party implementations ingress-controllers like:

- 1.HA PROXY
- 2.trafik
- 3.Istio  etc.,

But we have one implementation of ingress-controller which is maintained by kubernetes itself which is 
nginx-ingress-controller

### 2.To deploy nginx-ingress-controller in minikube.
```
ubuntu@balasenapathi:~$ minikube addons enable ingress -p ingress-cluster
❗  Executing "docker container inspect ingress-cluster --format={{.State.Status}}" took an unusually long time: 4.571829003s
💡  Restarting the docker service may improve performance.
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
▪ Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

### 3.To verify whether nginx-ingress-controller is enabled or not.
```
ubuntu@balasenapathi:~$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-bjgkn        0/1     Completed   0          4m12s
ingress-nginx-admission-patch-fgwsx         0/1     Completed   0          4m12s
ingress-nginx-controller-7799c6795f-g85mf   1/1     Running     0          4m10s
```
**Note:** Here -n denotes the namespace, as we can see that ingress-nginx-controller pod is running just
like other applications.

### 4.ingress-controller is a pod and it gets exposed through a service.
```
ubuntu@balasenapathi:~$ kubectl get service -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.108.13.27    <none>        80:31226/TCP,443:32390/TCP   9m57s
ingress-nginx-controller-admission   ClusterIP   10.101.239.58   <none>        443/TCP                      9m57s
```
**Note:** As we can see that a NodePort Service is created for this nginx-ingress-controller,as we discussed loadbalancer
is not supported on minikube thats the reason NodePort Service is created,if we are in cloud this should create a loadbalancer
service instead of NodePort Service and this loadbalacer will act like a entrypoint to our cluster.

**Ingress Rules:**

- Note: Now we have our nginx-ingress-controller ready to process our ingress rules.

To write ingress rule to access the nginx service and extend it to simple todo-ui application:

### 5.To get the apiVersion of ingress in kubernetes.
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep Ingress
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
ingresses                         ing          networking.k8s.io/v1                   true         Ingress
```
### 6.Create the Ingress Resource YAML File.

- Ingress-Rule:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
    - host: nginx-demo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 8082
```                  
                  
**Path Type:**

**1.ImplementationSpecific:** The path type is decided by the ingress-controller

**2.Exact:** The URL should match the exact path with case sensitivity

**3.Prefix:** The URL should match the path split by the / 

### 7.Apply the changes to the ingress-cluster to implement the ingress-rule.
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-ingress.yaml
ingress.networking.k8s.io/nginx-ingress created
```

### 8.We can verify that, it was created with apply nginx-ingress.yaml file.
```
ubuntu@balasenapathi:~$ kubectl get ingress
NAME            CLASS   HOSTS            ADDRESS        PORTS   AGE
nginx-ingress   nginx   nginx-demo.com   192.168.67.2   80      94s
```
**Note:** In general the ADDRESS will be the address of LoadBalancer

### 9.Now go to the browser and check whether we can able to access the nginx-demo.com
```
[nginx-demo.com]

This site can’t be reached Check if there is a typo in nginx-demo.com.
DNS_PROBE_FINISHED_NXDOMAIN
```
**Note:** This is because nginx-demo.com is not a valid DNS(Domain_Name_Service),when we type in any DNS in the browser it should 
resolve to some IP Address,nginx-demo.com should resolve to the minikube ip address.

### 10.To we can get the minikube ip by using this.
```
ubuntu@balasenapathi:~$ minikube ip -p ingress-cluster
192.168.67.2
```
### 11.Now we should map the minikube ip address to our nginx-demo.com in our machine,this can be done by editing below:
```
ubuntu@balasenapathi:~$ sudo nano /etc/hosts
192.168.67.2 nginx-demo.com
```
**Note:** Here nginx-demo.com should resolve to 192.168.67.2 which is minikube IP address

**Note:** But in cloud, we will map the CNAME to the LoadBalancer

### 12.Now go to the browser and check whether we can able to access the nginx-demo.com
```
[nginx-demo.com]

Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```
**Note:** Now we are able to access the nginx application on custom domain i.e, nginx-demo.com from web browser. 

### 13.Now extend this ingress to access our to-do ui and api applications:

This is the configuration file which holds the deployment and service configuration for the ui and api applications:
```
ubuntu@balasenapathi:~$ nano todo-ui-api.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-api
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
          image: pavanelthepu/todo-api:1.0.2
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
          image: pavanelthepu/todo-ui:1.0.2
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
spec:
  selector:
    app: todo-ui
  ports:
    - name: http
      port: 3001
      targetPort: 80
```
**Note:**

As we can see, we are pulling the todo-api image from Docker Hub and creating a deployment that creates 2 
replicas of the API application. Additionally, we are creating the todo-api-service to access the API 
application.

Similarly, we are creating a deployment for the todo-ui application and a todo-ui-service to access the UI
application.

### 14.Apply the changes in the ingress-cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-api.yaml
deployment.apps/todo-api created
service/todo-api-service created
deployment.apps/todo-ui created
service/todo-ui-service created
```
**Note:** The todo-ui deploment,todo-ui-service and todo-api deployment,todo-api-service are created.

## Now we will access these two applications using ingress:

- We will be accessing this application in two ways of routing:

**1.Path-Based Routing/Mapping:**
**2.Host-Based Routing/Mapping:**

**Path-Based Routing in Kubernetes Ingress:** Path-based routing directs traffic based on the URL path of incoming requests, allowing
different paths like "/(.*)" ,"/ui" and "/api" to be sent to distinct backend services. This approach is useful for managing various
application endpoints within a single domain.

**Host-Based Routing in Kubernetes Ingress:** Host-based routing routes traffic based on the hostname in the request, enabling multiple
websites or applications to share the same IP and port but serve content based on distinct hostnames (e.g., "todo.ui.com" and
"todo-api.com"). It's a powerful method for hosting multiple services on a single cluster while maintaining separation based on  
the requested domain.

# 1. Path-Based Routing/Mapping:
Path-Based Routing in Kubernetes Ingress: Path-based routing directs traffic based on the URL path of 
incoming requests, allowing different paths like /, /ui, and /api to be sent to distinct backend services.
This approach is useful for managing various application endpoints within a single domain.

### 1.Create the todo-ingress-path-based.yaml file 
```
ubuntu@balasenapathi:~$ nano todo-ingress-path-based.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todo-ingress-path-based
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: todo.com
      http:
        paths:
          - path: /(.*)
            pathType: Prefix
            backend:
              service:
                name: todo-ui-service
                port:
                  number: 3001
          - path: /api/(.*)
            pathType: Prefix
            backend:
              service:
                name: todo-api-service
                port:
                  number: 8080
```
**Explanation:**

Here, todo-ingress-path-based will be accessed on the host todo.com. In the URL, we are adding a wildcard
path /(.*) which means anything with todo.com/ should match this ingress rule. The pathType is 
ImplementationSpecific, and when we try to access it, it should go to todo-ui-service which is running on 
port 3001.

Now, another ingress rule is added for the todo-api application. We access the API with the host todo.com 
and the path /api/(.*), meaning anything with todo.com/api should match this ingress rule. The pathType is
ImplementationSpecific, and when we try to access it, it should go to todo-api-service running on port 8080.

**Note:** The annotation nginx.ingress.kubernetes.io/rewrite-target: /$1 captures any character in .* and 
assigns it to the placeholder $1 which is used as a parameter for the rewrite-target annotation.

For example, accessing todo.com/api/api/todos rewrites to todo.com/api/todos. The /api in the path is just
an identifier for this rule and it internally rewrites the rule to another URL.

### 2.Apply the Ingress Configuration.
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ingress-path-based.yaml
Warning: path /(.*) cannot be used with pathType Prefix
Warning: path /api/(.*) cannot be used with pathType Prefix
ingress.networking.k8s.io/todo-ingress-path-based created
```
### 3.We can verify that it was created with:
```
ubuntu@balasenapathi:~$ kubectl get ingress
NAME                      CLASS   HOSTS            ADDRESS        PORTS   AGE
nginx-ingress             nginx   nginx-demo.com   192.168.67.2   80      140m
todo-ingress-path-based   nginx   todo.com         192.168.67.2   80      3m20s
```
### 4.Map the Minikube IP Address to todo.com
```
ubuntu@balasenapathi:~$ sudo nano /etc/hosts
192.168.67.2 todo.com
```
**Note:** todo.com should resolve to 192.168.67.2, which is the Minikube IP address. On cloud platforms,
you would map the CNAME to the load balancer.

### 5.Access the Applications in the Browser.

Go to the browser and check whether you can access todo.com and todo.com/api/api/todos.
```
http://todo.com
```
![todo api](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/todo_ui_path.png)

**Note: The todo.com UI and API are loading and working**
```
http://todo.com/api/api/todos
```
![todo api](https://github.com/balusena/kubernetes-for-devops/blob/main/07-Kubernetes%20Ingress/todo_api_path.png)

**Note:** We can see that we get our data from the database.

This is called path-based routing, meaning based on the path we are routing the application into different
services like /ui and /api, etc.

# 2.Host-Based Routing/Mapping:
Host-Based Routing in Kubernetes Ingress: Host-based routing routes traffic based on the hostname in the 
request, enabling multiple websites or applications to share the same IP and port but serve content based
on distinct hostnames (e.g., todo-ui.com and todo-api.com). It's a powerful method for hosting multiple 
services on a single cluster while maintaining separation based on the requested domain.

Instead of accessing the UI and API on the same host with different paths, we can access this UI and API 
on different hosts like:

**UI ===> todo-ui.com**
**API ===> todo-api.com**

### 1.Create the file todo-ingress-host-based.yaml:
```
ubuntu@balasenapathi:~$ nano todo-ingress-host-based.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todo-ingress-host-based
spec:
  rules:
    - host: todo-ui.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: todo-ui-service
                port:
                  number: 3001
    - host: todo-api.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: todo-api-service
                port:
                  number: 8080
```
**Explanation:**

Here we are assigning the UI application to todo-ui.com and the API application to todo-api.com, separately
on different hosts within the same cluster. This setup uses the same IP and ports but different domain 
names (todo-ui.com and todo-api.com), maintaining separation within a single cluster, i.e., our custom 
created Minikube Ingress cluster.

### 2.Apply the Ingress Configuration
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ingress-host-based.yaml
ingress.networking.k8s.io/todo-ingress-host-based created
```

### 3.We can verify that it was created with:
```
ubuntu@balasenapathi:~$ kubectl get ingress
NAME                      CLASS   HOSTS                      ADDRESS        PORTS   AGE
nginx-ingress             nginx   nginx-demo.com             192.168.67.2   80      3h23m
todo-ingress-host-based   nginx   todo-ui.com,todo-api.com   192.168.67.2   80      37s
todo-ingress-path-based   nginx   todo.com                   192.168.67.2   80      65m
```

### 4.Map the Minikube IP Address to Our Hosts
```
ubuntu@balasenapathi:~$ sudo nano /etc/hosts
192.168.67.2 todo-ui.com
192.168.67.2 todo-api.com
```
**Note:** todo-ui.com and todo-api.com should resolve to 192.168.67.2, which is the Minikube IP address. 
On cloud platforms, you would map the CNAME to the load balancer.

### 5.Access the Applications in the Browser

Go to the browser and check whether you can access todo-ui.com and todo-api.com/api/todos.
```
http://todo-ui.com/
```























