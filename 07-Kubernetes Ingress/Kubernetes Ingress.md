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

## 1.Create a new cluster in minikube with name ingress-cluster:
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

## 2.We are using this nginx-deployment file in new cluster for our deployment:
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

## 3.We are using below nginx-service file to create service in the cluster:
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
## 4.To get the information about all services running in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP    4m
nginx-service   ClusterIP   10.107.16.240   <none>        8082/TCP   12s
```

## 5.To get the information about all pods,services,deploments,replicasets:
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

## 1.Create a new cluster in minikube with name ingress-cluster:
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

## 2.To deploy nginx-ingress-controller in minikube:
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

## 3.To verify whether nginx-ingress-controller is enabled or not:
```
ubuntu@balasenapathi:~$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-bjgkn        0/1     Completed   0          4m12s
ingress-nginx-admission-patch-fgwsx         0/1     Completed   0          4m12s
ingress-nginx-controller-7799c6795f-g85mf   1/1     Running     0          4m10s
```

**Note:** Here -n denotes the namespace, as we can see that ingress-nginx-controller pod is running just
like other applications.


























