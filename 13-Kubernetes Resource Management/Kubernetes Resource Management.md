# kubernetes Resource Management:
Kubernetes resource management involves controlling the allocation of compute resources like CPU and memory to containers
within a cluster. It uses requests to reserve resources and limits to cap usage, ensuring efficient utilization. Resource
quotas can restrict overall usage across namespaces, while limits ranges control resource limits per container. Kubernetes
also supports auto-scaling to dynamically adjust resource allocation based on demand, maintaining optimal performance and
cost-efficiency in cloud-native environments.

![Kubernetes Resource Management](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/resource_management.png)

Once an application is scheduled onto a node, it starts consuming the node's resources. Since multiple applications can 
be deployed to a Kubernetes cluster, it's important to be cautious when deploying your application. This ensures that it
uses computing resources like CPU and memory wisely, performs well, and doesn't impact other applications.

## Key Things We Need to Understand:
- **1.Requesting CPU and Memory:** Guarantees resources for the application using requests.
- **2.Limiting CPU and Memory:** Restricts resource usage for the application using limits.
- **3.Quality of Service (QoS):** Ensures performance levels for applications based on resource requests and limits.
- **4.Setting Minimum, Maximum, and Default Resources for Pods in a Namespace:** Achieved through LimitRange.
- **5.Limiting Resources in a Namespace:** Managed using ResourceQuota.

## Computing Resources:
Every node has resources such as CPU, memory, and disk space. CPU and memory are referred to as compute resources. We 
can request these compute resources from a node for later use.

![Kubernetes Computing Resources CPU Memory](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/computing_resources_cpu_memory.png)

These computing resources, such as CPU and memory, are crucial for a node, and we should be particularly cautious when 
using them.

### 1.Requests:
In Kubernetes, requests define the minimum amount of CPU and memory that a container requires. These requests ensure 
that the container receives the guaranteed resources it needs to function properly. By specifying requests, you help  
the scheduler allocate resources effectively and avoid contention among containers for limited resources.

Let's say we have an application that requires a minimum of one CPU and 200 MB of memory to start. If this application 
is deployed on a node where sufficient resources are not available, it may not start.

![Kubernetes Requests 1](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/requests_1.png)

What if there is a way to instruct Kubernetes to deploy this application or pod onto a node that has the minimum required
capacity? For every container, we can define the resources and request Kubernetes to deploy the application onto a node 
where the minimum CPU and memory are available by using resource requests. When we define the request, Kubernetes will 
guarantee to schedule the pod on a node where the minimum capacity is available. Once this pod is scheduled, it immediately
uses the resources of Node2, reducing the allocatable resources on Node2 to 400 MB of memory and 200m of CPU.

![Kubernetes Requests 2](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/requests_2.png)

If there are no nodes with the minimum capacity defined in the pod, the pod will remain in a pending state until the 
required resources become available.

![Kubernetes Requests 3](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/requests_3.png)

Once the resources become available on any node, the Kubernetes scheduler will schedule the pod onto that node. In short,
Kubernetes compares the resource requests defined in the pod with the available resources on each node in the cluster and
automatically assigns a suitable node to the pod. If we don't specify CPU or memory requests, we are implicitly indicating
that we don't have specific requirements for how much CPU time the process running in the container is allocated. In the 
worst-case scenario, the process may not receive any CPU time at all.

![Kubernetes Requests 4](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/requests_4.png)

Let's say there is suddenly a memory leak in the first application, causing Pod1 to use a lot of memory. What happens to
the second application on that node, which is Pod3? Pod3 might get terminated due to insufficient memory if it doesn't 
have enough to function properly. However, if you had instructed Kubernetes to limit the amount of memory Pod1 could use,
this situation could have been avoided. Fortunately, Kubernetes provides a safeguard against this scenario by allowing 
you to set resource limits, which prevent a pod from using more memory than specified.

![Kubernetes Requests 5](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/requests_5.png)

### 2.Limits:
Limits in Kubernetes set the maximum CPU and memory a container can use. If a container exceeds these limits, Kubernetes
may restrict its usage or even terminate it. This prevents one container from consuming too many resources, ensuring fair
distribution across all applications on the node.

![Kubernetes Limits](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/limits.png)

In Kubernetes, we can instruct a container to limit its CPU and memory usage by setting limits in the resources section 
of the pod specification.

- **To summarize both Requests and Limits:**
- Requests define the minimum resources required on a node to schedule a pod.
- Limits set the maximum resources a pod can use. If neither requests nor limits are defined, Kubernetes may schedule 
  all pods on the same node without considering resource constraints. This can lead to node overload, instability, and 
  degraded application performance, as resource contention may cause throttling and potential pod termination due to 
  insufficient memory.
  
### Stress Tool:
When dealing with CPU and memory, a tool like Stress can create load without the need for complex code. With this tool, 
you can easily specify how many CPUs to use and how much memory to consume, effectively simulating CPU and memory usage 
within a pod. For example, running the Stress process can utilize 8 CPUs and 128 MB of memory. A Docker image 
(polinux/stress) is available for this tool, so when you use this image, your pod comes with the Stress tool pre-installed,
ready to simulate the desired resource usage.

**Requests and Limits:**

- Note: Now we are going to use the stress tool:

### 1.Create a pod in minikube cluster.
```
ubuntu@balasenapathi:~$ nano resources-demo-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources-demo
spec:
  containers:
  - name: resources-demo
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "2", "--vm", "1", "--vm-bytes", "1G", "--vm-hang", "1"]
    resources:
      requests:
        cpu: "1"
        memory: "10Gi"
```
Note: In this example, we are using a Stress image that utilizes 2 CPUs and 1 GB of memory. In the requests section, we 
specify 1 CPU and 10 GB of memory. By doing this, we are instructing Kubernetes to schedule this pod onto a node where 
at least 1 CPU and 10 GB of memory are available.

### 2.To list down the nodes of the minikube cluster.
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   22m   v1.27.3
```
**Note:** We have one node i.e, minikube

### 3.To describe this minikube node:
```
ubuntu@balasenapathi:~$ kubectl describe node minikube
Capacity:
  cpu:                2
  ephemeral-storage:  102107096Ki
  hugepages-2Mi:      0
  memory:             3969528Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  102107096Ki
  hugepages-2Mi:      0
  memory:             3969528Ki
  pods:               110
```  
**Note:** In this scenario, the node has a capacity of 2 CPUs and 3 GB of memory, but we have requested 10 GB of memory.
Since no node has the minimum required 10 GB of memory available, the expectation is that the pod will not be scheduled 
on any node.

### 4.Now try to apply this changes into into the minikube cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f resources-demo-pod.yaml
pod/resources-demo created
```
### 5.To get the status of this pod.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
resources-demo   0/1     Pending   0          49s
```
**Note:** As we can see the status is pending,this pod never gets scheduled as there is no node with that memory available.

### 6.Now change the memory in the manifest file to 2GB and see that the pod gets scheduled.
```
ubuntu@balasenapathi:~$ nano resources-demo-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources-demo
spec:
  containers:
  - name: resources-demo
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "2", "--vm", "1", "--vm-bytes", "1G", "--vm-hang", "1"]
    resources:
      requests:
        cpu: "1"
        memory: "2Gi" 
```
### 7.Now try to apply this changes into into the minikube cluster.       
```
ubuntu@balasenapathi:~$ kubectl apply -f resources-demo-pod.yaml
The Pod "resources-demo" is invalid: spec: Forbidden: pod updates may not change fields other than 
`spec.containers[*].image`,`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations` (only additions to existing 
tolerations),`spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
  core.PodSpec{
  	Volumes:        {{Name: "kube-api-access-pxw5v", VolumeSource: {Projected: &{Sources: {{ServiceAccountToken: &{ExpirationSeconds: 
  	3607, Path: "token"}}, {ConfigMap: &{LocalObjectReference: {Name: "kube-root-ca.crt"}, Items: {{Key: "ca.crt", Path: 
  	"ca.crt"}}}}, {DownwardAPI: &{Items: {{Path: "namespace", FieldRef: &{APIVersion: "v1", FieldPath: "metadata.namespace"}}}}}}, 
  	DefaultMode: &420}}}},
  	InitContainers: nil,
  	Containers: []core.Container{
  		{
  			... // 6 identical fields
  			EnvFrom: nil,
  			Env:     nil,
  			Resources: core.ResourceRequirements{
  				Limits: nil,
  				Requests: core.ResourceList{
  					s"cpu":    {i: {...}, s: "1", Format: "DecimalSI"},
- 					s"memory": {i: resource.int64Amount{value: 10737418240}, s: "10Gi", Format: "BinarySI"},
+ 					s"memory": {i: resource.int64Amount{value: 2147483648}, s: "2Gi", Format: "BinarySI"},
  				},
  				Claims: nil,
  			},
  			ResizePolicy: nil,
  			VolumeMounts: {{Name: "kube-api-access-pxw5v", ReadOnly: true, MountPath: "/var/run/secrets/kubernetes.io/
  			serviceaccount"}},
  			... // 12 identical fields
  		},
  	},
  	EphemeralContainers: nil,
  	RestartPolicy:       "Always",
  	... // 28 identical fields
  }
```
**Note:** We get an error stating that we cannot update fields other than specific ones. Once a pod is created, changes 
to resource requests and limits are not allowed.

### 8.Now delete this pod.
```
ubuntu@balasenapathi:~$ kubectl delete -f resources-demo-pod.yaml
pod "resources-demo" deleted
```
### 9.Now try to apply this changes into into the minikube cluster.       
```
ubuntu@balasenapathi:~$ kubectl apply -f resources-demo-pod.yaml
pod/resources-demo created
```
### 10.To get the list of pods in cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
resources-demo   1/1     Running   0          44s
```
### 11.To get the status of pod in cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
resources-demo   1/1     Running   0          91s   10.244.0.3   minikube   <none>           <none>
```
**Note:** The pod is running on minikube node where we had around 3GB of memory allocatable.

### 12.To check how many resources a pod is consuming.
```
ubuntu@balasenapathi:~$ kubectl top pods
error: Metrics API not available
```
**Note:** The kubelet collects metrics on each node, such as CPU and memory usage from our pods. The metrics server then
collects and aggregates these metrics from all kubelets. The kubectl top command uses the metrics exposed by the metrics 
server via the API server. To check pod resource consumption, we need to enable the metrics server in our cluster. By 
default, the metrics server is already enabled in many clusters, such as EKS.

![Kubernetes Metrics Server](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/metrics_server.png)

### 13.To enable metrics server in minikube.
```
ubuntu@balasenapathi:~$ minikube addons enable metrics-server
    ▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.6.3
🌟  The 'metrics-server' addon is enabled
```
```
ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   default

ubuntu@balasenapathi:~$ kubectl config use-context minikube
Switched to context "minikube".
```
### 14.To check how many resources a pod is consuming:
```
ubuntu@balasenapathi:~$ kubectl top pods
NAME             CPU(cores)   MEMORY(bytes)   
resources-demo   1717m        858Mi           
```
**Note:** As we can see, this pod is using around 1717 millicores (approximately 2 CPUs) and 858 MiB (around 1 GB) of 
memory. This high usage is due to the load created by the Stress tool. Since no resource limits were defined, the pod 
can consume as much CPU and memory as available, without any restrictions. However, if other pods are running on the 
same node, they may be impacted by this unlimited resource usage. Therefore, it is always a good idea to compute the 
resource requirements for our pod and define limits. Limits can be set in the manifest file, just like CPU and memory 
requests.

### 15.Now lets create a pod in minikube cluster:
```
ubuntu@balasenapathi:~$ nano resources-demo-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources-demo
spec:
  containers:
  - name: resources-demo
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "2", "--vm", "1", "--vm-bytes", "4G", "--vm-hang", "1"]
    resources:
      requests:
        cpu: "1"
        memory: "2Gi"
      limits:
        cpu: "2"
        memory: "3Gi"
```

**Note:** By setting this limit, we are instructing Kubernetes not to allow this pod to use more than 2 CPUs and 3 GB of
memory. In this case, we intentionally impose a load of 4 GB, which exceeds the 3 GB memory limit. Let's see what happens
to the pod when it tries to exceed this limit.

### 16.First delete the pod:
```
ubuntu@balasenapathi:~$ kubectl delete -f resources-demo-pod.yaml
pod "resources-demo" deleted
```
### 17.Now apply the changes to the pod:
```
ubuntu@balasenapathi:~$ kubectl apply -f resources-demo-pod.yaml
pod/resources-demo created
```
### 18.Now list down the pod:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME             READY   STATUS      RESTARTS       AGE
resources-demo   0/1     OOMKilled   1 (10s ago)    19s
```
**Note:** OOM, which stands for Out-Of-Memory-Killed, occurs because we limited our pod to use 3 GB of memory, but the 
pod is trying to use 4 GB, exceeding the set limit. As we discussed, memory is not a compressible resource, meaning if 
the pod uses more memory than the defined limit, it will be killed and restarted according to the restartPolicy.

### 19.Now lets create a pod in minikube cluster:
```
ubuntu@balasenapathi:~$ nano resources-demo-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources-demo
spec:
  containers:
  - name: resources-demo
    image: polinux/stress
    imagePullPolicy: Always  # Add this line
    command: ["stress"]
    args: ["--cpu", "2", "--vm", "1", "--vm-bytes", "1.5G", "--vm-hang", "1"]
    resources:
      requests:
        cpu: "1"       # Request 1 CPU core
        memory: "2Gi"  # Request 2 gigabytes of memory
      limits:
        cpu: "1"       # Limit to 1 CPU cores
        memory: "3Gi"  # Limit to 3 gigabytes of memory
```        
**Note:** Let's change the memory limit to 3 GB and set the CPU limit to 1 CPU. We will then impose a load equivalent 
to 2 CPUs in the arguments. This configuration will restrict the pod to use only 1 CPU, even though the load requires 
2 CPUs. Let's see how the pod behaves under this condition.      

### 20.First delete the pod:
```
ubuntu@balasenapathi:~$ kubectl delete -f resources-demo-pod.yaml
pod "resources-demo" deleted
```
### 21.Now apply the changes to the pod:
```
ubuntu@balasenapathi:~$ kubectl apply -f resources-demo-pod.yaml
pod/resources-demo created
```
### 22.Now list down the pods and see the status:
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME             READY   STATUS    RESTARTS   AGE
resources-demo   1/1     Running   0          12s
```
**Note:** As we can see, the pod is still running even though we have exceeded the CPU limit. This demonstrates that 
when our pod uses more CPU than the defined limit, the CPU is throttled, but the pod is not killed. However, if the pod 
exceeds the memory limit, it will be killed and restarted. This highlights the difference in how Kubernetes handles 
CPU and memory limits: CPU usage is throttled, while exceeding memory limits results in the pod being terminated.

Exactly! When a pod exceeds its CPU limit, Kubernetes slows it down through throttling. However, if a pod uses more 
memory than its limit, it's terminated and restarted. CPU throttling results in slower performance, while memory overuse
can lead to the pod crashing and restarting.

![Kubernetes CPU Memory Intensive](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/cpu_memory_intensive.png)

**Note:** Requests and Limits are defined at the container level,not at the pod level.When a pod has multiple containers,
the resources requested and limited for each container are summed up and compared against the available resources on the
node during scheduling. By configuring Requests and Limits for each container, we can make efficient use of the CPU and 
memory resources available on our cluster nodes.

### Memory Management:
Even if we carefully define resources in our pod configuration, there may be instances where the pod exceeds its CPU and
memory limits. While exceeding CPU limits isn't a major issue as it just results in throttling, exceeding memory limits 
can be problematic because it will cause the pod to be killed.

Let's say we have a node with 8 GiB of memory, and we're scheduling two pods, Pod1 and Pod2, each with a request of 3 GiB.
After scheduling, we're left with 2 GiB of unallocated memory. However, Pod1 and Pod2 are using less memory than requested,
say 3 GiB and 2 GiB, respectively. Now, if we try to schedule another pod, Pod3, with a 3 GiB memory request, it won't be
scheduled even though there’s technically 3 GiB of free memory on the node. This is because the Kubernetes scheduler looks
at unallocated resources, not free memory, when deciding where to place a pod.

Additionally, if we define memory limits that exceed the node's capacity, such as setting limits of 6 GiB for both Pod1 
and Pod2, Kubernetes will still allow these pods to be scheduled because it only considers the requests, not the limits, 
during scheduling. This situation is known as "overcommitting," where the combined limits of the scheduled pods exceed 
the node's total capacity.

![Kubernetes Memory Management 1](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/memory_management_1.png)

Now, let's say Pod2 suddenly starts using its full memory limit of 6 GiB, which is equal to the limit we defined. There's
no issue with exceeding the limit, but now both pods together are using 9 GiB of memory, which is more than the node's 
capacity. One of these pods will need to be killed. The question is, which pod should be killed? Since both pods are within
their limits and there is no fault with either, Kubernetes will smartly decide which pod to kill by categorizing them into
three classes, specifically the Quality of Service (QoS) classes.

![Kubernetes Memory Management 2](https://github.com/balusena/kubernetes-for-devops/blob/main/13-Kubernetes%20Resource%20Management/memory_management_2.png)
















 
 






