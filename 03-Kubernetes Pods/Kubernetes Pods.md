# Kubernetes Pods

## 1.Introduction to Kubernetes Pods

A Pod is the smallest and simplest Kubernetes object. It represents a single instance of a running process 
in a cluster. Pods encapsulate one or more containers that share the same network namespace, storage, and 
lifecycle. Containers within a Pod communicate using `localhost` and share storage volumes, making Pods ideal
for applications requiring closely coupled components.

## 2.Sidecar Containers and Their Use Cases

Sidecar containers run alongside the main container in a Pod, sharing the same network and storage. They are
used to extend the functionality of the primary container. Common use cases include:

- **Logging and Monitoring:** A sidecar container collects logs or metrics from the main container.
- **Proxying:** Acts as a proxy server to handle network traffic.
- **Data Synchronization:** Manages data backup or synchronization tasks.

## 3.Need for a Pod

Pods are essential for:

- **Atomic Deployment:** They enable the deployment of closely related containers together, ensuring they run
  on the same host.
- **Resource Sharing:** Containers within a Pod share resources such as storage and networking.
- **Lifecycle Management:** Pods manage the lifecycle of containers as a single unit, simplifying updates and
  scaling.

## 4.Scaling Applications with Pods

Scaling applications involves adjusting the number of Pod replicas running to handle varying loads. 
Kubernetes provides several methods for scaling:

- **Manual Scaling:** Adjust the number of replicas in the deployment specification.
- **Horizontal Pod Autoscaler (HPA):** Automatically scales the number of Pod replicas based on metrics like
  CPU utilization or custom metrics.

## 5.Pod Communication

Pods communicate with each other and with other services through:
### Pod Communication

Pods in Kubernetes communicate with each other and with other services through several mechanisms:

- **Networking:** Pods within the same cluster use internal IP addresses for communication. Kubernetes 
  handles network routing and manages connectivity between pods, ensuring they can reach each other 
  seamlessly. This internal networking allows pods to interact directly by IP address or hostname, depending
  on the configuration.

- **Services:** Kubernetes Services provide a stable endpoint to access Pods. Services abstract the dynamic 
  IP addresses of the underlying Pods and offer a consistent interface for communication. They also 
  facilitate load balancing across multiple Pod instances, ensuring even distribution of traffic and fault 
  tolerance.

- **Inter-Pod Communication:** Containers within the same Pod can communicate with each other directly using
  `localhost` or `127.0.0.1`, as they share the same network namespace. For communication between different 
  Pods, Kubernetes uses service discovery and DNS. Each Service is assigned a DNS name 
  (e.g., `my-service.default.svc.cluster.local`), which resolves to the IP addresses of the Pods behind the 
  Service. This allows Pods to discover and communicate with each other efficiently, regardless of their 
  individual IP addresses.

### Pod Overview

![Kubernetes Pod](https://github.com/balusena/kubernetes-for-devops/blob/main/03-Kubernetes%20Pods/kubernetes_pod.png)

#### Kubernetes Pods: Key Points

- **Kubernetes Pod**: The smallest unit in Kubernetes.

- **Container Management**: Kubernetes doesn’t run containers directly. Instead, it wraps one or more containers into a higher-level structure called a pod.

- **Multi-container Pods**: One pod can contain multiple containers, allowing for tightly coupled applications with multi-tier containers in a single pod.

- **Container Runtime Support**: Kubernetes was created to support various container runtime environments, so pods can use multiple runtimes like Docker, containerd, etc.

- **Deployment Unit**: Pods are the smallest deployable units that can be created, scheduled, and managed on a Kubernetes cluster. Each pod is assigned a unique IP address within the cluster.

- **Network and Port Usage**: Containers within a pod share the same network namespace. If one container uses a port (e.g., port 80), other containers in the same pod cannot use the same port.

- **Resource Management**: Although pods can hold multiple containers, it’s advisable to limit the number of containers in a pod. Since pods are scaled as a unit, all containers must scale together, which can lead to inefficient resource use.

- **Shared Resources**: Containers in the same pod share the same storage volumes and network resources and communicate via localhost.

- **Pod Spec**: Kubernetes uses YAML to describe the desired state of the containers in a pod, known as a Pod Spec. These specifications are passed to the kubelet through the API server.

- **Replication**: Pods are used as the unit of replication. Kubernetes can deploy new replicas of a pod to handle increased load as necessary.

- **Volume Sharing**: Volumes can be shared between multiple pods.

- **Kubectl**: Interact with Kubernetes and manage pods using `kubectl` commands.

## 6.Creating Pods with kubectl:
```
#1.Creating a nginx pod using kubectl.

ubuntu@balasenapathi:~$ kubectl run nginx-pod --image=nginx
pod/nginx-pod created
```
```
#2.To see list of pods in kubernetes cluster:

ubuntu@balasenapathi:~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          102s
```
## 7.Creating Pods with YAML:
```
#1.Clone this repository and move to example folder

git clone https://github.com/balusena/kubernetes-for-devops.git
cd  03-Kubernetes Pods/examples
```
```
# 2.To get the apiVersion of pods in kubernetes:

ubuntu@balasenapathi:~$ kubectl api-resources | grep pods
pods                              po           v1                                     true         Pod
```
```
#3.Create the yaml file nginx-pod.yaml

ubuntu@balasenapathi:~$ nano nginx-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod1
  labels:
    team: integrations
    app: todo
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```
```
#4.Creating a Pod

ubuntu@balasenapathi:~$ kubectl apply -f nginx-pod.yaml
pod/nginx-pod1 created
```
```
#5.Listing the Pods

ubuntu@balasenapathi:~$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod    1/1     Running   0          27m
nginx-pod1   1/1     Running   0          54s
```
```
#6.Deleting a Pod

ubuntu@balasenapathi:~$ kubectl delete pod nginx-pod1
pod "nginx-pod1" deleted
```
## 8.Filtering Pods 

```
#1.Filtering Pods by Label.

ubuntu@balasenapathi:~$ kubectl get pods -l team=integrations
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod1   1/1     Running   0          5m15s
```
```
#2.Filtering Pods with Multiple Labels

ubuntu@balasenapathi:~$ kubectl get pods -l team=integrations,app=todo
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod1   1/1     Running   0          6m54s
```
```
#3.Getting Detailed Pod Information

ubuntu@balasenapathi:~$ kubectl get pod nginx-pod1 -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
nginx-pod1   1/1     Running   0          9m50s   10.244.1.3   local-cluster-m02   <none>    
```
```
#4.Getting Pod Information in YAML Format

ubuntu-dsbda@ubuntudsbda-virtual-machine:~$ kubectl get pod nginx-pod1 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"todo","team":"integrations"},"name":"nginx-
      pod1","namespace":"default"},"spec":{"containers":[{"image":"nginx:latest","name":"nginx-container","ports":[{"containerPort":
      80}]}]}}
  creationTimestamp: "2023-09-03T14:39:08Z"
  labels:
    app: todo
    team: integrations
  name: nginx-pod1
  namespace: default
  resourceVersion: "19270"
  uid: bb7f8789-ee2d-49a7-998f-bcceacd6eb83
spec:
  containers:
  - image: nginx:latest
    imagePullPolicy: Always
    name: nginx-container
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-8rwzz
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: local-cluster-m02
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-8rwzz
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-09-03T14:39:08Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-09-03T14:39:14Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-09-03T14:39:14Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-09-03T14:39:08Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://fcc3768f72a48103c62caa155a58016630401a6d98ad79a30a264091666d28cd
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:104c7c5c54f2685f0f46f3be607ce60da7085da3eaa5ad22d3d9f01594295e9c
    lastState: {}
    name: nginx-container
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2023-09-03T14:39:14Z"
  hostIP: 192.168.58.3
  phase: Running
  podIP: 10.244.1.3
  podIPs:
  - ip: 10.244.1.3
  qosClass: BestEffort
  startTime: "2023-09-03T14:39:08Z"
```
## 9.Listing Pods with details:
```
#1.To describe the pod with detail inforation about it.

ubuntu@balasenapathi:~$ kubectl describe pod nginx-pod1
Name:             nginx-pod1
Namespace:        default
Priority:         0
Service Account:  default
Node:             local-cluster-m02/192.168.58.3
Start Time:       Sun, 03 Sep 2023 20:09:08 +0530
Labels:           app=todo
team=integrations
Annotations:      <none>
Status:           Running
IP:               10.244.1.3
IPs:
IP:  10.244.1.3
Containers:
nginx-container:
Container ID:   docker://fcc3768f72a48103c62caa155a58016630401a6d98ad79a30a264091666d28cd
Image:          nginx:latest
Image ID:       docker-pullable://nginx@sha256:104c7c5c54f2685f0f46f3be607ce60da7085da3eaa5ad22d3d9f01594295e9c
Port:           80/TCP
Host Port:      0/TCP
State:          Running
Started:      Sun, 03 Sep 2023 20:09:14 +0530
Ready:          True
Restart Count:  0
Environment:    <none>
Mounts:
/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8rwzz (ro)
Conditions:
Type              Status
Initialized       True
Ready             True
ContainersReady   True
PodScheduled      True
Volumes:
kube-api-access-8rwzz:
Type:                    Projected (a volume that contains injected data from multiple sources)
TokenExpirationSeconds:  3607
ConfigMapName:           kube-root-ca.crt
ConfigMapOptional:       <nil>
DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
Normal  Scheduled  14m   default-scheduler  Successfully assigned default/nginx-pod1 to local-cluster-m02
Normal  Pulling    14m   kubelet            Pulling image "nginx:latest"
Normal  Pulled     14m   kubelet            Successfully pulled image "nginx:latest" in 1.78972478s (1.78973595s including waiting)
Normal  Created    14m   kubelet            Created container nginx-container
Normal  Started    14m   kubelet            Started container nginx-container
```
## 10.Getting Into a Pod

### 1.Accessing a Single Container Pod

To access a pod with a single container, use the following command to open a shell session within the pod:

```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-pod1 -- bash
root@nginx-pod1:/# ls
bin   dev		   docker-entrypoint.sh  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   lib64  media   opt  root  sbin  sys  usr
root@nginx-pod1:/# exit
exit
```
### 2.Accessing a Specific Container in a Multi-Container Pod
When a pod contains multiple containers, specify the container you want to access using the -c option. 
This command opens a shell session within the specified container:
```
ubuntu@balasenapathi:~$ kubectl exec -it nginx-pod1 -c nginx-container -- bash
root@nginx-pod1:/# ls
bin   dev		   docker-entrypoint.sh  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   lib64  media   opt  root  sbin  sys  usr
root@nginx-pod1:/# exit
exit
```
**Note:**
Single Container Pods: If there is only one container in the pod, you do not need to specify the container
name; it automatically accesses the only container present.

## 11.Port Forwarding

### Introduction

When you deploy an application in Kubernetes, accessing the pod directly from outside the node is not 
possible. Instead, you can access it from within the node. In this section, we will see how to access 
a pod from outside of the node using port forwarding.

### 1.Accessing the Pod from Within the Node

If you have access to the cluster (e.g., `local-cluster`), you can use the port-forwarding feature to 
access the pod. Here’s how you can do it:
```
ubuntu@balasenapathi:~$ kubectl port-forward nginx-pod1 8083:80
Forwarding from 127.0.0.1:8083 -> 80
Forwarding from [::1]:8083 -> 80
Handling connection for 8083
Handling connection for 8083
```
### 2.Accessing the Service
In the example above, we are forwarding port 8083 on the local machine to port 80 of the container in the
pod nginx-pod1. This means that any request to localhost:8083 will be forwarded to port 80 of the nginx 
container running inside the pod.
```
====> Visit http://localhost:8083 in your browser to see the nginx welcome page:

Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```
### 3.Stopping Port Forwarding
To stop port forwarding, use Ctrl+C to terminate the port forwarding process. Here’s what happens when you
stop it:
```
ubuntu@balasenapathi:~$ kubectl port-forward nginx-pod1 8083:80
Forwarding from 127.0.0.1:8083 -> 80
Forwarding from [::1]:8083 -> 80
Handling connection for 8083
Handling connection for 8083
^C ------> Ctrl+c
```
```
After stopping port forwarding, trying to access http://localhost:8083 will result in an error:

plaintext
Copy code
This site can’t be reached
localhost refused to connect.
Try:

Checking the connection
Checking the proxy and the firewall
ERR_CONNECTION_REFUSED
```
**Note:**
Port forwarding is particularly useful for debugging purposes as it allows you to access services running
inside your pods directly from your local machine.

## 12.Checking Pod Logs

To view the logs of a pod, you can use the `kubectl logs` command. This command retrieves the logs from 
the containers in the specified pod.
```
ubuntu@balasenapathi:~$ kubectl logs nginx-pod1
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/03 14:39:14 [notice] 1#1: using the "epoll" event method
2023/09/03 14:39:14 [notice] 1#1: nginx/1.25.2
2023/09/03 14:39:14 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2023/09/03 14:39:14 [notice] 1#1: OS: Linux 5.15.0-82-generic
2023/09/03 14:39:14 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/03 14:39:14 [notice] 1#1: start worker processes
2023/09/03 14:39:14 [notice] 1#1: start worker process 29
2023/09/03 14:39:14 [notice] 1#1: start worker process 30
127.0.0.1 - - [03/Sep/2023:16:00:32 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
127.0.0.1 - - [03/Sep/2023:16:00:32 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8083/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
2023/09/03 16:00:32 [error] 29#29: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8083", referrer: "http://localhost:8083/"
127.0.0.1 - - [03/Sep/2023:16:25:28 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" "-"
```
This command outputs the logs generated by the container running in the specified pod. The logs include 
details of the application's execution, errors, and requests received.

### Log Analysis

**1.Initialization Logs:** These entries show the configuration and startup messages, such as configuration
file checks and setup messages.

**2.Access Logs:** These lines show HTTP requests handled by the application, including status codes and
request details.

**3.Error Logs:** Any issues encountered, such as missing files or failed operations, will be logged here.

This command is useful for debugging and monitoring the application running inside your pod.

## 13.Deleting Pods

In Kubernetes, you can delete pods using different methods depending on how they were created. Here’s how
you can manage pod deletions:

### 1.Deleting Pods Using a YAML File
If you have a YAML file that defines your pod, you can delete the pod (or pods) defined in that file using
the following command:
```
ubuntu@balasenapathi:~$ kubectl delete -f nginx-pod.yaml
pod "nginx-pod1" deleted
```
```
#1.After running this command, you can check the remaining pods in the cluster.

ubuntu@balasenapathi:~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          137m
```
### 2.Deleting Pods Created with kubectl
If you created the pod using kubectl commands directly (not via a YAML file), you should specify the 
correct resource type when deleting:
```
#1.Incorrect Command:

ubuntu@balasenapathi:~$ kubectl delete nginx-pod
error: the server doesn't have a resource type "nginx-pod"
```
**Note:**This command fails because nginx-pod is not a recognized resource type.
```
#2.Correct Command:

ubuntu@balasenapathi:~$ kubectl delete pod nginx-pod
pod "nginx-pod" deleted
```
**Note**
Here, pod is the correct resource type, and nginx-pod is the name of the pod you want to delete.

## 14.Verifying Deletion
To ensure the pod has been deleted, you can run:
```
ubuntu@balasenapathi:~$ kubectl get pods
No resources found in default namespace.
```
**Note**
This command confirms that there are no remaining pods in the default namespace.






