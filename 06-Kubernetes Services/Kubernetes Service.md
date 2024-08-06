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

That's where Kubernetes Services come into the picture. The main objective of the services is to abstract 
the pod IP address from the end user. When a service is created, an IP address is assigned to that service
and this IP address doesn't change as long as the service exists. Users can call a single stable IP address
instead of calling each pod individually, and the service forwards the request to the pods. Now, even if 
the pod IP address changes, it doesn't matter, as the service takes care of routing to the appropriate pods.
Services are not created on any node, unlike pods.

**Other advantages of Kubernetes Services:**

**1.Load balancing:** Services distribute network traffic across multiple pods to ensure no single pod 
becomes a bottleneck. By spreading the load, Kubernetes Services enhance the reliability and performance 
of applications, preventing any single pod from becoming overwhelmed. This built-in load balancing is 
crucial for handling varying loads and ensuring that applications remain responsive and efficient, 
especially under high traffic conditions.

**2.Service discovery:** Services make it easy for pods to find and communicate with each other,simplifying
the development of distributed systems.Kubernetes provides DNS names for services,allowing applications to
use service names instead of hard coding IP addresses.This automatic service discovery mechanism facilitates
smoother interactions between microservices,enabling developers to build scalable and modular applications
without worrying about dynamic IP address changes.

**3.Zero downtime deployments:** Services enable seamless updates and scaling of applications without 
downtime, ensuring continuous availability. Kubernetes supports rolling updates and blue-green deployments,
allowing new versions of applications to be deployed without affecting the running instances. This means
that users experience uninterrupted service while updates or maintenance tasks are performed in the 
background, thus improving the overall user experience and system reliability.

**Use case-2:**
Now we have two pods of the same application, which is Nginx. In real-time scenarios, we may have many 
replicas of the same application. In that case, when we make a request, to which pod does our request go?
Services take care of load balancing. This means that the service provides load balancing when you have 
pod replicas; it picks a pod randomly and forwards the request to it. So, we don't need to worry about 
load balancing when we have multiple pods running the same application. Services also offer other 
advantages like service discovery and zero downtime deployments.

### The different Service types in Kubernetes are:

- 1.ClusterIP Service
- 2.Multi-Port Service
- 3.NodePort Service
- 4.LoadBalancer Service

### 1.To get the apiVersion of services in kubernetes
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep services
services             svc          v1                                     true         Service
apiservices                       apiregistration.k8s.io/v1              false        APIService
```
### 2.Create a service using file nginx-service.yaml
```
ubuntu@balasenapathi:~$ nano nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 8082
      targetPort: 80
```
**Note:**
- 1.Port on which Service will receive the request ---> 8082
- 2.targetPort on which the container is running   ---> 80

**1.ClusterIP-Service:**
A `ClusterIP` service exposes the ports on an IP Address that is internal to the cluster. This means the IP
Address cannot be accessed from outside of the cluster. This type of service is useful when you don't want
to expose your application to the outside world but want all the pods inside the cluster (local-cluster) to
access it.

- Example: Database Service

**Note:** This service is the default service. If no type is specified, it will be treated as a `ClusterIP`
service.

## Understanding Services

A `Service` in Kubernetes is an abstraction for a set of Pods. To communicate with the Pods, you should 
communicate through the `Service` instead of directly interacting with the Pods. If there are hundreds of
Pods, the `Service` uses labels to know to which Pods it should forward the request. This works similarly
to how Kubernetes deployments work.

### cluster-ip-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 8082
      targetPort: 80
```
![Kubernetes ClusterIP Service](https://github.com/balusena/kubernetes-for-devops/blob/main/06-Kubernetes%20Services/clusterip_service.png)

**Notes:**
- In the spec section, you should provide labels with the selector attribute. Here, you should specify the
  Pod labels defined as part of the deployment. In this example, we have given app: nginx which matches 
  those Pods.

- If multiple labels are defined as a selector,all the labels should match with the Pod labels,not just one.

This setup ensures that the Service properly routes traffic to the appropriate set of Pods based on the 
label selector.

### 3.Now apply the changes to the services into kubernetes:
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-service.yaml
service/nginx-service created
```

### 4.Now list down all the services
```
ubuntu@balasenapathi:~$ kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP    2d3h
nginx-service   ClusterIP   10.103.232.64   <none>        8082/TCP   101m

ubuntu@balasenapathi:~$ kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP    2d3h
nginx-service   ClusterIP   10.103.232.64   <none>        8082/TCP   102m
```
**Note:** Whenever a Service is created it gets a Private Cluster IP Address "10.103.232.64" as this cannot
be accessed externally there is no EXTERNAL-IP <none> for this ClusterIP Service and this service can be 
accessible on 8082 Port.

**Verifying External Access:**

### 5.This service is not accessible from outside of the cluster lets verify:
```
ubuntu@balasenapathi:~$ curl 10.103.232.64:8082
^C
```
**Note:** As we can see it is not able to reach the service,as we discussed that this service can be 
accessed from any of the pods from the cluster(local-cluster)

**Verifying Internal Access:**

### 6.This service can be accessible from any of the Pods with in the cluster lets verify this by getting into any of the Pods in cluster:

### To get the list of all Pods Running in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-785c55b987-nxzr8   1/1     Running   0          2m22s
nginx-deployment-785c55b987-wkw6d   1/1     Running   0          115s
```
**Accessing ClusterIP Service using IP address of the service:**

### Using kubectl exec to enter one of the pods and trying to access the service internally.
```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-deployment-785c55b987-nxzr8 -- /bin/bash
/# curl 10.103.232.64:8082
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
**Note:** By using curl 10.103.232.64:8082 inside the Pod in the cluster, we can access the Nginx 
service/container using the ClusterIP service. This demonstrates that ClusterIP services cannot be 
accessed from outside of the cluster, but these services can be used to access from all the pods in 
the cluster. As the name suggests, these services are restricted to the cluster.

**Accessing ClusterIP Service using name of the service:**

**Note:**We can also access the ClusterIP Service not only with the IP Address of the Service we can also 
access with the name of the Service
```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-deployment-785c55b987-nxzr8 -- /bin/bash
/# curl nginx-service:8082
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
**Note:** We got the response by using the name of the service ====> "curl nginx-service:8082"

**This shows that we can access the ClusterIP Services in two ways:**
- 1.Using the IP Address           ===> "curl 10.103.232.64:8082"
- 2.Using the Name of the Service  ===> "curl nginx-service:8082"

***Accessing Services Using Port-Forwarding:**

There might be cases where we want to debug our `ClusterIP` service and need to access it from our local 
machine. We can achieve this by using port-forwarding on services, similar to how we do it on Pods and 
Deployments.

### 7.Port-Forwarding a Service
Use the following command to port-forward the `nginx-service` from port `8082` to port `8083` on your 
local machine:

```
ubuntu@balasenapathi:~$ kubectl port-forward service/nginx-service 8083:8082
Forwarding from 127.0.0.1:8083 -> 80
Forwarding from [::1]:8083 -> 80
Handling connection for 8083
Handling connection for 8083
```
**Note:** By this command, we are port-forwarding the services that are running on port 8082 to port 8083.

### 8.Verifying in a Web Browser

- Now, go to your web browser and check the following URL: 
```
localhost:8083
```
- You should see the following page:
```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```
**Note:** This shows that we can access the services by using port-forwarding.

**Services Offers Load balancing:**

### 9.Testing Load Balancing with Kubernetes Services

Kubernetes Services provide load balancing to distribute traffic across multiple Pods. To verify that load
balancing is working, we can generate some load by continuously accessing the Nginx service multiple times.

### To get the list of all Pods Running in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-785c55b987-nxzr8   1/1     Running   0          12m
nginx-deployment-785c55b987-wkw6d   1/1     Running   0          12m
```
### To get into the pod with sh:
```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-deployment-785c55b987-nxzr8 -- sh 
/ #
```
### Automating Load Testing with Shell Script

To test load balancing by continuously accessing the service, you can use a shell script to automate the 
process. This script will iterate 20 times, sending requests to the Nginx service to observe how the load
is distributed between the Pods.

**Shell Script for Load Testing:**

You can use the following shell script to automate the load testing:
```
i=1
while [ "$i" -le 20 ]; do
  curl nginx-service:8082
  i=$(( i + 1 ))
done
```
```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-deployment-785c55b987-nxzr8 -- sh
/ # i=1
/ # while [ "$i" -le 20 ]; do
>   curl nginx-service:8082;
>   i=$(( i + 1 ))
> done 
```
### Verifying Load Balancing by Monitoring Pod Logs
To confirm that load balancing is working, you can check the logs of the Pods to see if the requests are
being distributed between them. This is done by monitoring the logs of both Pods while generating load.

**Monitoring Pod Logs:**

**Logs of Pod 1:** ['nginx-deployment-785c55b987-wkw6d']

```
ubuntu@balasenapathi:~$ kubectl logs nginx-deployment-785c55b987-wkw6d -f
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/06 21:13:17 [notice] 1#1: using the "epoll" event method
2023/09/06 21:13:17 [notice] 1#1: nginx/1.25.2
2023/09/06 21:13:17 [notice] 1#1: built by gcc 12.2.1 20220924 (Alpine 12.2.1_git20220924-r10) 
2023/09/06 21:13:17 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/06 21:13:17 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/06 21:13:17 [notice] 1#1: start worker processes
2023/09/06 21:13:17 [notice] 1#1: start worker process 31
2023/09/06 21:13:17 [notice] 1#1: start worker process 32
10.244.1.2 - - [06/Sep/2023:21:18:16 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
```
**Logs of Pod 2:** ['nginx-deployment-785c55b987-nxzr8']
```
ubuntu@balasenapathi:~$ kubectl logs nginx-deployment-785c55b987-nxzr8 -f
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/06 21:13:05 [notice] 1#1: using the "epoll" event method
2023/09/06 21:13:05 [notice] 1#1: nginx/1.25.2
2023/09/06 21:13:05 [notice] 1#1: built by gcc 12.2.1 20220924 (Alpine 12.2.1_git20220924-r10) 
2023/09/06 21:13:05 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/06 21:13:05 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/06 21:13:05 [notice] 1#1: start worker processes
2023/09/06 21:13:05 [notice] 1#1: start worker process 30
2023/09/06 21:13:05 [notice] 1#1: start worker process 31
127.0.0.1 - - [06/Sep/2023:21:23:29 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
127.0.0.1 - - [06/Sep/2023:21:23:30 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
```
**Note:** The -f flag is used to watch or stream the logs in real-time.

If you observed previously, when we hit the service through the browser, the request was directed to pod2.
However, when we use port-forwarding, the load balancing does not work; it simply selects one Pod and 
continuously sends the requests to that Pod. This is why demonstrating load balancing from within the Pod
is more effective than using port-forwarding.

### Running the Shell Script

To verify load distribution, we can run the shell script to access the `nginx-service` continuously. 
Here's how you can do it:

1.**Access the Pod**:

```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-deployment-785c55b987-nxzr8 -- sh
/ #
```

2.**Run the Shell Script**:

```
/ # i=1
/ # while [ "$i" -le 20 ]; do
>   curl nginx-service:8082;
>   i=$(( i + 1 ))
> done
```

3.**Script Output**:

After running the script, you should see the following output, which indicates that the `nginx-service` was
accessed 20 times:

```
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to nginx!</title>
    <style>
        html { color-scheme: light dark; }
        body { width: 35em; margin: 0 auto; font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
</head>
<body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.
    </p>

    <p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
- The output will repeat for each request made by the script.

**Note:** The script has successfully hit the `nginx-service` 20 times.

### Pod Logs and Load Balancing Verification

**Pod 1 Logs:**
```
ubuntu@balasenapathi:~$ kubectl logs nginx-deployment-785c55b987-wkw6d -f
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/06 21:13:17 [notice] 1#1: using the "epoll" event method
2023/09/06 21:13:17 [notice] 1#1: nginx/1.25.2
2023/09/06 21:13:17 [notice] 1#1: built by gcc 12.2.1 20220924 (Alpine 12.2.1_git20220924-r10) 
2023/09/06 21:13:17 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/06 21:13:17 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/06 21:13:17 [notice] 1#1: start worker processes
2023/09/06 21:13:17 [notice] 1#1: start worker process 31
2023/09/06 21:13:17 [notice] 1#1: start worker process 32
10.244.1.2 - - [06/Sep/2023:21:18:16 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.2 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.2 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.2 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.2 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.2 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.2 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
```
**Pod 2 Logs:**
```
ubuntu@balasenapathi:~$ kubectl logs nginx-deployment-785c55b987-nxzr8 -f
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/06 21:13:05 [notice] 1#1: using the "epoll" event method
2023/09/06 21:13:05 [notice] 1#1: nginx/1.25.2
2023/09/06 21:13:05 [notice] 1#1: built by gcc 12.2.1 20220924 (Alpine 12.2.1_git20220924-r10)
2023/09/06 21:13:05 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/06 21:13:05 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/06 21:13:05 [notice] 1#1: start worker processes
2023/09/06 21:13:05 [notice] 1#1: start worker process 30
2023/09/06 21:13:05 [notice] 1#1: start worker process 31
127.0.0.1 - - [06/Sep/2023:21:23:29 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
127.0.0.1 - - [06/Sep/2023:21:23:30 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
10.244.1.1 - - [06/Sep/2023:21:59:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-"
10.244.1.1 - - [06/Sep/2023:21:59:44 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.2.1" "-
```
**Note:** The logs from both pods show multiple requests being processed, indicating that the load is 
being distributed between the pods. This confirms that load balancing is functioning as expected.

### 10.Pods Associated with Services

To see what pods are associated with the services.

```
ubuntu@balasenapathi:~$ kubectl get endpoints
NAME            ENDPOINTS                     AGE
kubernetes      192.168.58.2:8443             2d4h
nginx-service   10.244.1.2:80,10.244.1.3:80   151m
```
**Note:** Whenever a service is created, an endpoint object with the same name as the service 
(e.g., "nginx-service") is also created. As shown, two pods are associated with this service: 
- 1 Pod IP and Port Number ===> 10.244.1.2:80 
- 2 Pod IP and Port Number ===> 10.244.1.3:80

We can also check which pods are associated with a service by describing the service:

```
ubuntu@balasenapathi:~$ kubectl describe service/nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.103.232.64
IPs:               10.103.232.64
Port:              <unset>  8082/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.2:80,10.244.1.3:80
Session Affinity:  None
Events:            <none>
```
**Note:** The Endpoints field shows the pods associated with the nginx-service, which are:
- 1 Pod IP and Port Number ===> 10.244.1.2:80
- 2 Pod IP and Port Number ===> 10.244.1.3:80

### Checking Pods and Endpoints

When a pod is added or deleted with the label that we have given in the service,the IP addresses associated
with the service endpoints change seamlessly.

**Note:** We can add or delete the pods by using ReplicaSets or Deployments, we can simply change the
replicas count or number in spec section in the yaml or manifest files and re-apply the services.

**Option 1:** Scaling Using kubectl scale
- To set the number of replicas to 3
```
ubuntu@balasenapathi:~$ kubectl scale deployment nginx-deployment --replicas=3
deployment.apps/nginx-deployment scaled
```
**Option 2:** Editing the Deployment YAML
- Edit the deployment YAML file directly to change the number of replicas:
```
ubuntu@balasenapathi:~$ kubectl edit deployment nginx-deployment
Find the spec section and update the replicas field:
spec:
replicas: 3

ubuntu@balasenapathi:~$ kubectl edit deployment nginx-deployment
deployment.apps/nginx-deployment edited
```
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7c4c499fd5-ppcnb   1/1     Running   0          76s
nginx-deployment-7c4c499fd5-wz67p   1/1     Running   0          76s
```
```
ubuntu@balasenapathi:~$ kubectl get endpoints
NAME            ENDPOINTS                     AGE
kubernetes      192.168.58.2:8443             11m
nginx-service   10.244.2.2:80,10.244.2.3:80   86s
```
```
ubuntu@balasenapathi:~$ kubectl describe service/nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.100.253.136
IPs:               10.100.253.136
Port:              <unset>  8082/TCP
TargetPort:        80/TCP
Endpoints:         10.244.2.2:80,10.244.2.3:80
Session Affinity:  None
Events:            <none>
```
**Note:** The Endpoints field has updated to reflect the new IP addresses 10.244.2.2:80 and 10.244.2.3:80 
as new pods have been added or existing pods have been replaced. This demonstrates how Kubernetes services
handle dynamic changes in the pod infrastructure seamlessly.

- The Endpoints field shows the new pods associated with the nginx-service, which are:
- 1 Pod IP and Port Number ===> 10.244.2.2:80
- 2 Pod IP and Port Number ===> 10.244.2.3:80

**2.Multi-Port Service:**

If we have multiple containers in a pod and want to expose two containers through services, we can define
multiple ports in a single service. As we can see, ports is an array. For multi-port services, the name is
mandatory. Our service will accept requests on two ports as defined here: one on port 8081 and another on 
port 8082. One container is running on port 80 and the second one is running on port 8080. Now, we can 
access our service on port 8082, and the request goes to port 80 of the container. When we access our 
service on port 8081, the request goes to port 8080 of the container. These multi-port services are very 
helpful when we have sidecar containers, just like our Prometheus exporters.

### multi-port-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: nginx
  ports:
    - name: proxy
      port: 8082
      targetPort: 80
    - name: application
      port: 8081
      targetPort: 8080
```

![Kubernetes Multi-Port Service](https://github.com/balusena/kubernetes-for-devops/blob/main/06-Kubernetes%20Services/multiport_service.png)

**With this configuration:**

- 1.The service listens on port 8082 and forwards requests to port 80 of the target container.

- 2.The service listens on port 8081 and forwards requests to port 8080 of the target container.

This setup is particularly useful for scenarios involving sidecar containers, where auxiliary containers 
(such as Prometheus exporters) run alongside the main application container within the same pod.

**3.NodePort Service:**

As the ClusterIP Service is not accessible from outside the cluster, we use the NodePort Service to expose
our application to the public. This type of service exposes the pod on each node at a static port called 
nodePort. We can access the NodePort Service from outside the cluster by requesting `nodeIP:nodePort` 
(e.g., `192.168.58.2:30000`). Any request to our cluster on the given nodePort gets forwarded to the 
service (`nginx-service`), which internally acts like a ClusterIP and forwards the request to the 
appropriate pods.

## Important Points
- **NodePort Range**: The nodePort can range from 30000 to 32767.
- **Automatic Port Assignment**: If the `nodePort` is not specified, Kubernetes automatically assigns a port that is not in use.
- **Port Handling**: If `targetPort` is not specified, it defaults to the value of `port`.

### nginx-service.yaml

**Note:** Use this yaml file to configure the NodePort Services in Kubernetes Cluster.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 8082
      targetPort: 80
      nodePort: 30000
```
**In this configuration:**

- 1.The service is of type NodePort, which means it will expose the service on a specific port on each node.

- 2.The service listens on port 8082 and forwards requests to port 80 of the target pod.

- 3.The nodePort field specifies the static port (30000) on each node's IP that the service is exposed on.

![Kubernetes NodePort Service](https://github.com/balusena/kubernetes-for-devops/blob/main/06-Kubernetes%20Services/nodeport_service.png)

### 1.Apply the changes of the service in the kubernetes cluster:
```
ubuntu@balasenapathi:~$  kubectl apply -f nginx-service.yaml
service/nginx-service created
```

### 2.List down all the services in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get services
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          140m
nginx-service   NodePort    10.100.253.136   <none>        8082:30000/TCP   130m
```
**Note:** In this configuration, the type of the service has been changed from ClusterIP to NodePort. As a
result, port 30000 is opened on the node, and it is mapped to port 8082 of the service. This is reflected 
as 8082:30000/TCP.

### 3.To get the IP address in the minikube of my local-cluster:

```
ubuntu@balasenapathi:~$ minikube ip -p local-cluster
192.168.58.2
```
Now go to the web browser and access the nginx-service with the local-cluster IP and the node port that we
have used (30000):
```
[192.168.58.2:30000]

Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```
**Note:** As we can see, we can access our nginx with the nodeIP , i.e., 192.168.58.2:30000, as this is a NodePort Service.

There is a shortcut for this; instead of getting the IP address and typing it in the browser, we can run 
the following command:
```
ubuntu@balasenapathi:~$ minikube ip -p local-cluster
192.168.58.2

ubuntu@balasenapathi:~$ minikube service nginx-service -p local-cluster
|-----------|---------------|-------------|---------------------------|
| NAMESPACE |     NAME      | TARGET PORT |            URL            |
|-----------|---------------|-------------|---------------------------|
| default   | nginx-service |        8082 | http://192.168.58.2:30000 |
|-----------|---------------|-------------|---------------------------|
🎉  Opening service default/nginx-service in default browser...
ubuntu@balasenapathi:~$ Opening in existing browser session.
```
```
[192.168.58.2:30000]

Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```
**Note:** The downside to this NodePort Service is that we are accessing this service with nodeIP, but 
nodeIP may change (e.g., when we restart our node). Also, it is not secure as we are opening ports on 
the node, so it is not advised to use NodePort Service in production. To overcome this, we use another 
type of service called LoadBalancer Service.

**4.LoadBalancer Service:**

A LoadBalancer Service exposes the pods externally using cloud providers like AWS, Azure, or GCP. This 
type of service acts like a combination of ClusterIP and NodePort internally. The LoadBalancer is the 
preferred solution for exposing a cluster to the wider internet.

To convert a service into a LoadBalancer Service, you simply need to set the `type` to `LoadBalancer` in
your service definition. Kubernetes will then request the cloud provider hosting the cluster to set up a 
LoadBalancer that points to the external node IPs on a specific NodePort. If the cloud provider does not 
support LoadBalancers, Kubernetes will fall back to using a NodePort.

For example, in Minikube, which does not support external LoadBalancers, the LoadBalancer Service will 
function similarly to a NodePort Service.

### nginx-service.yaml

**Note:** Use this yaml file to configure the LoadBalancer Services in Kubernetes Cluster.

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
      nodePort: 30000
```
**In this configuration:**

- type: LoadBalancer specifies that the service should be exposed via a LoadBalancer.

- port: 8082 is the port that the LoadBalancer listens on.

- targetPort: 80 is the port on the pod that the LoadBalancer forwards traffic to.

- nodePort: 30000 is the port on each node that the LoadBalancer uses (this field is optional and may be omitted if not needed).

**Note:** In Minikube, the LoadBalancer will not create an external LoadBalancer but will instead behave like a NodePort Service.

![Kubernetes LoadBalancer Service](https://github.com/balusena/kubernetes-for-devops/blob/main/06-Kubernetes%20Services/loadbalancer_service.png)

### 1.Apply the changes of the service in the kubernetes cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-service.yaml
service/nginx-service created
```
### 2.List down all the services in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get services
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP          3h36m
nginx-service   LoadBalancer   10.100.253.136   <pending>     8082:30000/TCP   3h26m
```
**Note:** As shown, the `TYPE` of the service has been changed to `LoadBalancer`, and the `EXTERNAL-IP` 
is set to `<pending>`. If your cluster is running on a cloud provider, you would typically see an actual
`EXTERNAL-IP` instead of `<pending>`, which can be accessed over the internet. Additionally, the 
`nodePort: 30000` and `targetPort: 8082` are visible. The LoadBalancer Service internally functions 
like both a NodePort and a ClusterIP Service. In Minikube, you can access this LoadBalancer Service using
the same `minikube service` command.

### 3.Accessing the LoadBalancer Service.
```
ubuntu@balasenapathi:~$ minikube service nginx-service -p local-cluster
|-----------|---------------|-------------|---------------------------|
| NAMESPACE |     NAME      | TARGET PORT |            URL            |
|-----------|---------------|-------------|---------------------------|
| default   | nginx-service |        8082 | http://192.168.58.2:30000 |
|-----------|---------------|-------------|---------------------------|
🎉  Opening service default/nginx-service in default browser...
ubuntu@balasenapathi:~$ Opening in existing browser session.
[192.168.58.2:30000]

Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```















































