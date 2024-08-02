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




