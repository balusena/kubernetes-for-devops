# Kubernetes Autoscaling:
Kubernetes autoscaling dynamically adjusts the number of pods or nodes in a cluster based on resource usage or custom metrics.

- **Horizontal Pod Autoscaler (HPA):** Scales the number of pod replicas.
- **Vertical Pod Autoscaler (VPA):** Adjusts resource requests for individual pods.
- **Cluster Autoscaler (CA):** Adds or removes nodes to meet the cluster's resource demands.

This ensures efficient resource utilization,cost management,and high availability of applications based on workload needs.

In deployments, we have seen how to scale our deployments manually. This allows us to adjust the number of replicas 
whenever there is unusual traffic. However, monitoring traffic continuously and manually scaling our applications to 
handle such traffic spikes can be a tedious process. What if there were a way to monitor our pods and automatically 
scale them whenever there is an increase in CPU usage, memory, or other metrics like QPS (Queries Per Second)? This 
is called **Autoscaling**—scaling automatically based on specific metrics.

![Kubernetes Autoscaling Intro](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/autoscaling.png)
  
Kubernetes can handle scaling not only for pods but also for the underlying infrastructure. If we're running on a cloud 
environment, Kubernetes can automatically spin up additional nodes when the existing nodes cannot accommodate more pods.
This ensures that our applications remain scalable and responsive, even during periods of high demand.

## Types of Autoscalers:
Kubernetes offers 3 types of Autoscalers.
- **1.HPA(Horizontal Pod Autoscaler)
- **2.VPA(Vertical Pod Autoscaler)
- **3.CA(Cluster Autoscaler)

### 1.HPA(Horizontal Pod Autoscaler):
The Horizontal Pod Autoscaler (HPA) increases the number of replicas whenever there is a spike in CPU, Memory, QPS, or 
other metrics, distributing the load among the pods. This process is known as scaling up. However, increasing the number
of replicas with HPA is not always the best solution. For example, to make a database handle more connections, we might 
need to increase the memory and CPU of existing pods instead of creating new ones.This can be achieved using the Vertical
Pod Autoscaler (VPA).

![Kubernetes Horizontal Pod Autoscaler](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa.png)

### 2.VPA(Vertical Pod Autoscaler):
The Vertical Pod Autoscaler (VPA) analyzes the resource usage of a deployment and adjusts them accordingly to handle the
load. With VPA, instead of creating new pods, we increase the resources (like CPU and memory) of existing pods.This allows
the application to handle higher demand without needing to scale out horizontally.

![Kubernetes Vertical Pod Autoscaler](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/vpa.png)

### 3.CA(Cluster Autoscaler):
Finally if there is no capacity in the cluster it doesnt make any sense to increase the number of pods with horizontal
pods autoscaler or increase the resources with VPA(Vertical Pod Autoscaler) as needed we should be able to add nodes
to the cluster automatically to accommodate the more number the more number of pods.CA(Cluster Autoscaler) can do that
job for us it adds the nodes to the cluster if there are any pods struck in the pending state because of lack of resources
in the cluster.

![Kubernetes Cluster Autoscaler](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/ca.png)

### Scale Up and Scale Down in (HPA, VPA, CA) Autoscaling

#### 1. Scale Up
In autoscaling, "scale up" refers to increasing the number of pod replicas or resources allocated to existing pods to 
handle higher demand, thereby distributing the load more effectively.

- **Horizontal Pod Autoscaler (HPA)**
  - **Scale Up:** Increases the number of pod replicas based on metrics like CPU usage or memory, distributing the load 
    across more pods.

- **Vertical Pod Autoscaler (VPA)**
  - **Scale Up:** Increases resource requests (CPU and memory) for existing pods to handle higher loads, adjusting the 
    resources allocated to each pod.

- **Cluster Autoscaler (CA)**
  - **Scale Up:** Adds new nodes to the cluster when there are insufficient resources to schedule new pods, helping to 
    manage increased pod demands.
    
![Kubernetes Autoscaling ScaleUp](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/scaleup.png)  

#### 2. Scale Down
In autoscaling, "Scale down" involves reducing the number of pod replicas or resources when demand decreases, optimizing
resource usage and reducing costs.

- **Horizontal Pod Autoscaler (HPA)**
  - **Scale Down:** Decreases the number of pod replicas when metrics fall below the threshold, reducing resource consumption.

- **Vertical Pod Autoscaler (VPA)**
  - **Scale Down:** Decreases resource requests for existing pods when the load decreases, optimizing resource usage.

- **Cluster Autoscaler (CA)**
  - **Scale Down:** Removes underutilized nodes from the cluster to reduce costs when there are fewer pods requiring resources.
  
![Kubernetes Autoscaling ScaleDown](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/scaledown.png)

### Working examples of Autoscalers:

#### 1.Horizontal Pod Autoscaler (HPA):
Let's say our application is an e-commerce application and during a big billion sale, we get so many orders. When there 
is such unusual traffic, our application should be able to serve all the users without any downtime. For that, 

when the demand increases, the app should scale up, which means increasing the number of replicas to stay responsive.

![Kubernetes HPA ScaleUp](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_scaleup.png)

When the demand decreases, the app should scale down, which means decreasing the number of replicas to avoid wasting any
resources.

![Kubernetes HPA ScaleDown](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_scaledown.png)

But how does Kubernetes know when to scale up and when to scale down a deployment? For that, Kubernetes offers a resource
named Horizontal Pod Autoscaler (HPA). With HPA, we can instruct Kubernetes when and how to scale our deployment based on
specific metrics.

#### Horizontal Pod Autoscaler (HPA) Workflow:
Let's see how HPA works in Kubernetes. In the Kubernetes architecture, we have learned that on every worker node, kubelet
runs, and there is an agent in kubelet called cAdvisor (Container Advisor) by Google. When our pods are running on the 
worker node, cAdvisor can scrape the pod's CPU and memory usage every 10 seconds. Every minute, the metrics server aggregates
these metrics and exposes them to the Kubernetes API server. The HPA controller queries the API server every 15 minutes 
for these metrics. Once the controller gets the desired pod metrics, it checks them against our definitions and decides 
to scale up or scale down our replicas. The HPA controller updates the replica count in the target deployment. Spinning 
up new pods or deleting existing pods is managed by the replication controller, and cAdvisor starts collecting metrics 
from the new pods as well.

![Kubernetes HPA Worflow](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_workflow.png)

**HPA(Horizontal Pod Autoscaler) uses the below formula to calculate number of replicas:**

![Kubernetes HPA Replica Formula](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_replica_formula.png)

**Usecase:**

Let's say we have a deployment with two (2) pods, and both pods are consuming 90% of CPU on average. Now, assume that we
define the deployment to scale when CPU usage exceeds 70%. Every 15 seconds, the HPA (Horizontal Pod Autoscaler) calculates
the desired number of replicas. This results in three (3) pods instead of two, so that the load is distributed more 
effectively, preventing the initial pods from struggling. Not only deployments, but we can also scale StatefulSets and 
ReplicaSets with HPA (Horizontal Pod Autoscaler).

![Kubernetes HPA Replica Formula Example](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/hpa_example.png)

We can even define multiple metrics for scaling, such as CPU and memory usage. When AutoScaling is based on multiple 
metrics, the autoscaler calculates the replica count for each metric individually and then takes the highest value—in 
this case, 4 replicas.

In summary, the purpose of HPA (Horizontal Pod Autoscaler) is to calculate the replica count that brings the current 
metrics value as close as possible to the target value.

- **Create a simple Spring Boot API with a `/stress` endpoint that generates CPU load when invoked:**

### 1.Now deploy this API with kubernetes deployment.
```
ubuntu@balasenapathi:~$ nano deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: utility-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: utility-api
  template:
    metadata:
      name: utility-api-pod
      labels:
        app: utility-api
    spec:
      containers:
        - name: utility-api
          image: balasenapathi/utility-api
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: 20Mi
              cpu: "0.25"
            limits:
              memory: 400Mi
              cpu: "1"
```
**Note: This is the deployment manifest for the Utility API here we are giving the 2 replicas.**

### 2.Now create a kubernetes service in cluster to access this application.
```
ubuntu@balasenapathi:~$ nano service.yaml
apiVersion: v1
kind: Service
metadata:
  name: utility-api-service
spec:
  selector:
    app: utility-api
  ports:
    - port: 8080
      targetPort: 8080
```
### 3.Now create/define a HPA-Horizontal Pod AutoScaler for the above deployment.
```
ubuntu@balasenapathi:~$ nano hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: utility-api
spec:
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - resource:
        name: cpu
        target:
          averageUtilization: 70
          type: Utilization
      type: Resource
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: utility-api
```
**Note:** The minReplicas and maxReplicas parameters indicate that our deployment will scale up to a maximum of 5 replicas
if the average CPU usage exceeds 70%, and scale down to a minimum of one replica when CPU usage is low.

### 4.First apply our deployment of spring boot application.
```
ubuntu@balasenapathi:~$ kubectl apply -f deployment.yaml
deployment.apps/utility-api created
```
### 5.Now apply the service to access this application.
```
ubuntu@balasenapathi:~$ kubectl apply -f service.yaml
service/utility-api-service created
```
### 6.Now apply the HPA in cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f hpa.yaml
horizontalpodautoscaler.autoscaling/utility-api created
```
**Note:** The expectation is that the HPA controller should scrape the metrics of our pods because we specified 
scaleTargetRef as the utility-api that was deployed.

### 7.Now list down all the pods in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
utility-api-5b4b9b5776-g2fkl   1/1     Running   0          7m21s
utility-api-5b4b9b5776-z4tp8   1/1     Running   0          7m21s
```
### 8.Now lets list down the metrics of these pods.
```
ubuntu@balasenapathi:~$ minikube addons enable metrics-server
💡  metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.6.3
🌟  The 'metrics-server' addon is enabled


ubuntu@balasenapathi:~$ kubectl top pods
NAME                           CPU(cores)   MEMORY(bytes)   
utility-api-5b4b9b5776-g2fkl   2m           99Mi                
```
**Note:** As we can see, the CPU usage is 2m and the memory consumption is 99Mi. Although we defined 2 replicas in our 
deployment, we currently see only one pod.

### 9.Now list down the pods again.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
utility-api-5b4b9b5776-g2fkl   1/1     Running   0          12m
```
**Note:** This is because the average CPU utilization of our deployment is less than 70%. We have defined that the 
application should scale down to one replica if there is no significant load on the pods. Since our application is 
consuming only 2m millicores, it has scaled down to one pod

### 10.We can verify that HPA scaled it down by describing the HPA.
```
ubuntu@balasenapathi:~$ kubectl get  hpa
NAME          REFERENCE                TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
utility-api   Deployment/utility-api   1%/70%    1         5         1          15m
```
**Note:** We can see only 1% of the CPU is being used, and the current replicas are 1.

### 11.Now try to describe this HPA.
```
ubuntu@balasenapathi:~$ kubectl describe hpa utility-api
Name:                                                  utility-api
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Wed, 04 Oct 2023 01:57:11 +0530
Reference:                                             Deployment/utility-api
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (2m) / 70%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       1 current / 1 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent recommendation
  ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange   the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  12m   horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
```
**Note:** As we can see, the new size is 1 with the reason being 'All metrics below target.' This is how HPA works. We 
observed that the HPA automatically scaled down the pods because the CPU usage was less than 70%.

- **Now, let's try to add some load onto the pods and see what happens:**

**Note:** For this, we are creating a simple traffic-generator Alpine pod that can easily communicate with the utility-api.

### 12.Now create a manifest file for traffic-generator.yaml.
```
ubuntu@balasenapathi:~$ nano traffic-generator.yaml
apiVersion: v1
kind: Pod
metadata:
  name: traffic-generator
spec:
  containers:
  - name: alpine
    image: alpine
    args:
    - sleep
    - "100000000"
```    
### 13.Now apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f  traffic-generator.yaml
pod/traffic-generator created
```
**Note:** It created a simple alpine pod.

### 14.To get the list of pods running in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
traffic-generator              1/1     Running   0          2m52s
utility-api-5b4b9b5776-g2fkl   1/1     Running   0          32m
```
### 14.Now try to get into this pod:
```
ubuntu@balasenapathi:~$ kubectl exec -it traffic-generator -- sh
/ # 
```
**Note:** Now we are in the traffic-generator alpine pod. Our job is to generate load on the utility-api pods so that the
CPU utilization goes beyond 70%, allowing us to observe how the pods automatically scale up. To achieve this, we need to
continuously call the stress API, causing the CPU usage to increase.

Note: Here, we introduce the HTTP benchmarking tool called wrk. With wrk, we can seamlessly generate load on the pods by
running a simple command in the Alpine pod. For more information, you can visit the wrk GitHub repository.
```
This is the link of wrk in GitHub ===> https://github.com/wg/wrk
```
### 15.To access the Traffic Generator Pod.
```
ubuntu@balasenapathi:~$ kubectl exec -it traffic-generator -- sh
/ # apk add wrk
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/community/x86_64/APKINDEX.tar.gz
(1/3) Installing libgcc (12.2.1_git20220924-r10)
(2/3) Installing luajit (2.1_p20230410-r1)
(3/3) Installing wrk (4.2.0-r0)
Executing busybox-1.36.1-r2.trigger
OK: 9 MiB in 18 packages
```
**Note:** We can see that the wrk tool is successfully installed in our alpine pod.

### 16.Now lets simulate the load with this command in the pod.
```
ubuntu@balasenapathi:~$ kubectl exec -it traffic-generator -- sh
/ # apk add wrk  
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/community/x86_64/APKINDEX.tar.gz
(1/3) Installing libgcc (12.2.1_git20220924-r10)
(2/3) Installing luajit (2.1_p20230410-r1)
(3/3) Installing wrk (4.2.0-r0)
Executing busybox-1.36.1-r2.trigger
OK: 9 MiB in 18 packages
/ # wrk -c 5 -t 5 -d 300s -H "Connection: Close" http://utility-api-service:8080/api/stress
Running 5m test @ http://utility-api-service:8080/api/stress
  5 threads and 5 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    30.06ms   95.27ms   1.87s    96.95%
    Req/Sec    58.17     26.37   160.00     63.21%
  80420 requests in 5.00m, 18.56MB read
  Socket errors: connect 0, read 0, write 480, timeout 4
Requests/sec:    267.99
Transfer/sec:     63.33KB  

wrk -c 5 -t 5 -d 300s -H "Connection: Close" http://utility-api-service:8080/api/stress
``` 
**Note:** Here we are making 5 connections with five threads for a duration of 300 seconds to the utility-api-service/api
service and finally, we are firing the stress endpoint.

### 17.Now lets open the new tab and see the pod matrix.
```
ubuntu@balasenapathi:~$ kubectl top pods
NAME                           CPU(cores)   MEMORY(bytes)   
traffic-generator              112m         5Mi             
utility-api-5b4b9b5776-g2fkl   277m         129Mi           
```
Note: As we can see, the CPU usage has increased to 112m millicores and memory usage to 129Mi. Let’s wait for a while 
because it takes some time for cAdvisor to collect CPU metrics and for the metrics server to aggregate them before the 
Autoscaler can take action.

### 18.Now lets open the new tab and see the pod matrix.
```
ubuntu@balasenapathi:~$ kubectl top pods
NAME                           CPU(cores)   MEMORY(bytes)   
traffic-generator              112m         5Mi             
utility-api-5b4b9b5776-g2fkl   277m         129Mi           
utility-api-5b4b9b5776-hzqrj   447m         122Mi           
utility-api-5b4b9b5776-kmc8w   275m         126Mi           
utility-api-5b4b9b5776-rt5s6   434m         121Mi
utility-api-5b4b9b5576-mr1ht   312m         123Mi
```
**Note:** We can see that the api pods are increased to 5.This is the power of HPA.

### 19.Now lets try to list the HPA from the cluster.
```
ubuntu@balasenapathi:~$ kubectl get hpa
NAME          REFERENCE                TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
utility-api   Deployment/utility-api   143%/70%   1         5         5          47m
```
**Note:** We can see, the CPU usage is above 70%. Since we set the maximum number of replicas to 5, the deployment scaled
up to 5 pods. If a higher maximum value had been set, more pods would have been created. The load is now distributed across
these pods.

### 20.We can also verify this by describing HPA.
```
ubuntu@balasenapathi:~$ kubectl describe hpa utility-api
Name:                                                  utility-api
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Wed, 04 Oct 2023 01:57:11 +0530
Reference:                                             Deployment/utility-api
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  92% (231m) / 70%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       5 current / 5 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent 
  recommendation
  ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from cpu resource utilization 
  (percentage of request)
  ScalingLimited  True    TooManyReplicas      the desired replica count is more than the maximum replica count
Events:
  Type    Reason             Age    From                       Message
  ----    ------             ----   ----                       -------
  Normal  SuccessfulRescale  43m    horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
  Normal  SuccessfulRescale  5m20s  horizontal-pod-autoscaler  New size: 2; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  4m20s  horizontal-pod-autoscaler  New size: 4; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  2m20s  horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
```
**Note:** As we can see,the new size increased from 1 to 4,and then to 5,due to CPU resource utilization exceeding the
target percentage.Now that the 300 seconds have elapsed and the wrk tool has stopped firing the stress API,we expect the
CPU usage of the pods to decrease.Consequently,the extra pods created should be deleted as there is no longer high CPU
usage.

### 21.Now lets list down the metrics of the pod.
```
ubuntu@balasenapathi:~$ kubectl top pods
NAME                           CPU(cores)   MEMORY(bytes)   
traffic-generator              0m           5Mi             
utility-api-5b4b9b5776-g2fkl   2m           129Mi           
utility-api-5b4b9b5776-hzqrj   2m           122Mi           
utility-api-5b4b9b5776-kmc8w   2m           126Mi           
utility-api-5b4b9b5776-rt5s6   2m           121Mi
utility-api-5b4b9b5576-mr1ht   2m           123Mi
```
**Note:** As we can see, the CPU usage has decreased to 2m millicores in each pod. The HPA controller should now take 
action to scale the pods down to 1 replica, as this CPU usage is below the 70% threshold.

### 22.Now lets get the HPA.
```
ubuntu@balasenapathi:~$ kubectl get hpa
NAME          REFERENCE                TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
utility-api   Deployment/utility-api   0%/70%    1         5         1          132m
```
**Note:** We can see that the CPU usage is almost 0%, so now lets wait for the HPA Controller to take the action.

### 23.Now list down the pods in cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
traffic-generator              1/1     Running   0          109m
utility-api-5b4b9b5776-g2fkl   1/1     Running   0          139m
```
**Note:** As we can see,we are left with only 1 pod running in the cluster due to the lack of CPU usage.This demonstrates
how the HPA (Horizontal Pod Autoscaler) can effectively scale the number of replicas up and down based on load.

### 24.Now lets get the HPA.
```
ubuntu@balasenapathi:~$ kubectl get hpa
NAME          REFERENCE                TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
utility-api   Deployment/utility-api   0%/70%    1         5         1          21m
```
### 25.We can also verify this by describing HPA.
```
ubuntu@balasenapathi:~$ kubectl describe hpa utility-api
Name:                                                  utility-api
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Wed, 04 Oct 2023 17:20:13 +0530
Reference:                                             Deployment/utility-api
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (2m) / 70%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       1 current / 1 desired
Conditions:
  Type            Status  Reason            Message
  ----            ------  ------            -------
  AbleToScale     True    ReadyForNewScale  recommended size matches current size
  ScalingActive   True    ValidMetricFound  the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  True    TooFewReplicas    the desired replica count is less than the minimum replica count
Events:
  Type    Reason             Age                 From                       Message
  ----    ------             ----                ----                       -------
  Normal  SuccessfulRescale  12m                 horizontal-pod-autoscaler  New size: 4; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  12m                 horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  3m5s (x2 over 16m)  horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
```
#### 2.Vertical Pod Autoscaler (VPA)
Vertical Scaling involves increasing or decreasing the compute resources of a replica. Without Vertical Pod Autoscaler 
(VPA), over-allocating resources with high requests and limits can lead to increased costs if the resources are not fully
utilized. Conversely, under-allocating resources can degrade application performance and may result in Kubernetes killing
the pods if they become resource-starved. Vertical Pod Autoscaler (VPA) helps address these issues by dynamically adjusting
resource allocations based on actual usage.

#### Vertical Pod Autoscaler (VPA) Workflow:
Vertical Pod Autoscaler (VPA) performs three steps:

![Kubernetes VPA Worflow](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/vpa_workflow.png)

- **1.Reads Resource Metrics:** It monitors the resource metrics of your deployment, similar to how HPA does.
- **Recommends Resources:** Based on these metrics, it provides recommendations for resource requests.
- **Updates Resources (Optional):** If auto-update is enabled, it adjusts the resource allocations of the pods accordingly.

### 1.Now create a vpa.yaml manifest file.
```
ubuntu@balasenapathi:~$ nano vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: utility-api
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: utility-api
  updatePolicy:
    updateMode: "Off"
```
**Note:** Here, our target is the same utility-api, and the updateMode is set to Off. The updateMode has three values:

- Auto: Applies the recommendations suggested by the VPA directly by updating the pods.
- Off: Provides recommendations but does not update the pods.
- Initial: Applies the recommended values only to newly created pods.

Please note: updateMode: Off is preferred in production environments because applying changes with VPA can restart pods,
potentially causing workload disruption. The best practice is to set updateMode: Off in production, feed the recommendations
to a capacity monitoring dashboard like Grafana, and apply the recommendations in the next deployment cycle.

### 2.Now apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f vpa.yaml
error: resource mapping not found for name: "utility-api" namespace: "" from "vpa.yaml": no matches for kind "VerticalPodAutoscaler" in 
version "autoscaling.k8s.io/v1"
ensure CRDs are installed first
```
**Note:** There is an error indicating that VerticalPodAutoscaler is not found. The reason is that, by default, VPA is not 
available in our cluster. We need to install it explicitly by cloning the VPA repository.

### 3.Cloning VPA from the Git.
```
ubuntu@balasenapathi:~$ git clone https://github.com/kubernetes/autoscaler.git
Cloning into 'autoscaler'...
remote: Enumerating objects: 184223, done.
remote: Counting objects: 100% (3220/3220), done.
remote: Compressing objects: 100% (2306/2306), done.
remote: Total 184223 (delta 1385), reused 1803 (delta 823), pack-reused 181003
Receiving objects: 100% (184223/184223), 205.46 MiB | 3.19 MiB/s, done.
Resolving deltas: 100% (116352/116352), done.
Updating files: 100% (30568/30568), done.
```
**Note:** The repository is cloned and this repository is being maintained by the kubernetes.

### 4.Now get into the autoscaler repository/directory run this command to install VPA .
```
ubuntu@balasenapathi:~$ cd autoscaler
ubuntu-dsbda@ubuntudsbda-virtual-machine:~/autoscaler$ ./vertical-pod-autoscaler/hack/vpa-up.sh
customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalercheckpoints.autoscaling.k8s.io created
customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalers.autoscaling.k8s.io created
clusterrole.rbac.authorization.k8s.io/system:metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:vpa-actor created
clusterrole.rbac.authorization.k8s.io/system:vpa-status-actor created
clusterrole.rbac.authorization.k8s.io/system:vpa-checkpoint-actor created
clusterrole.rbac.authorization.k8s.io/system:evictioner created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-actor created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-actor created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-checkpoint-actor created
clusterrole.rbac.authorization.k8s.io/system:vpa-target-reader created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-target-reader-binding created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-evictioner-binding created
serviceaccount/vpa-admission-controller created
serviceaccount/vpa-recommender created
serviceaccount/vpa-updater created
clusterrole.rbac.authorization.k8s.io/system:vpa-admission-controller created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-admission-controller created
clusterrole.rbac.authorization.k8s.io/system:vpa-status-reader created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-reader-binding created
deployment.apps/vpa-updater created
deployment.apps/vpa-recommender created
Generating certs for the VPA Admission Controller in /tmp/vpa-certs.
Generating RSA private key, 2048 bit long modulus (2 primes)
..................................................................................+++++
...................................................................................+++++
e is 65537 (0x010001)
Generating RSA private key, 2048 bit long modulus (2 primes)
...........................+++++
.........................+++++
e is 65537 (0x010001)
Signature ok
subject=CN = vpa-webhook.kube-system.svc
Getting CA Private Key
Uploading certs to the cluster.
secret/vpa-tls-certs created
Deleting /tmp/vpa-certs.
deployment.apps/vpa-admission-controller created
service/vpa-webhook created
```
**Note:** Now we can see that the VPA has been installed.

### 5.Now apply the changes in the cluster for VPA:
```
ubuntu@balasenapathi:~$ kubectl apply -f vpa.yaml
verticalpodautoscaler.autoscaling.k8s.io/utility-api created
```
**Note:** We can see that the VPA has created

### 6.We can verify that by using:
```
ubuntu@balasenapathi:~$ kubectl get vpa
NAME          MODE   CPU   MEM       PROVIDED   AGE
utility-api   Off    25m   262144k   True       68s 

ubuntu@balasenapathi:~$ kubectl get vpa utility-api
NAME          MODE   CPU   MEM       PROVIDED   AGE
utility-api   Off    25m   262144k   True       105s
```
### 7.Now try to describe this VPA:
```
ubuntu@balasenapathi:~$ kubectl describe vpa utility-api
Name:         utility-api
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  autoscaling.k8s.io/v1
Kind:         VerticalPodAutoscaler
Metadata:
  Creation Timestamp:  2023-10-04T17:49:56Z
  Generation:          1
  Resource Version:    2463
  UID:                 3a320cf3-2deb-426c-afc5-aa2a3c6b6625
Spec:
  Target Ref:
    API Version:  apps/v1
    Kind:         Deployment
    Name:         utility-api
  Update Policy:
    Update Mode:  Off
Status:
  Conditions:
    Last Transition Time:  2023-10-04T17:50:44Z
    Status:                True
    Type:                  RecommendationProvided
  Recommendation:
    Container Recommendations:
      Container Name:  utility-api
      Lower Bound:
        Cpu:     25m
        Memory:  262144k
      Target:
        Cpu:     25m
        Memory:  262144k
      Uncapped Target:
        Cpu:     25m
        Memory:  262144k
      Upper Bound:
        Cpu:     15851m
        Memory:  235427771491
Events:          <none>
```
**Note:** Here, the Lower Bound represents the minimum estimated resources for the container, while the Upper Bound represents
the maximum recommended resource estimation. In other words, the Lower Bound corresponds to the Requests and the Upper Bound
corresponds to the Limits. This is how VPA provides resource recommendations, rather than requiring us to guess the Requests
and Limits for our application.

Now that we understand how to scale our application both horizontally and vertically, what if we don't have enough capacity
in the cluster to scale our pods?

#### 3.Cluster Autoscaler(CA):
**For example,** when a pod requests 1 CPU but the cluster has only 0.5 CPU available, the scheduler marks the pod as 
unschedulable. As we've seen in advanced scheduling, in this case, we have multiple options:

1. **Free up resources:** Delete unwanted workloads in the cluster.
2. **Add a node manually:** Manually add new nodes.
3. **Let Kubernetes handle it:** Allow Kubernetes to automatically add nodes when necessary, using an automated approach.

This is where the **Cluster Autoscaler (CA)** comes into play. The CA checks for any unschedulable pods every 10 seconds.
Once one or more unschedulable pods are detected, the CA runs an algorithm to decide how many nodes are needed to deploy
all pending pods and what type of nodes should be created. The same process applies for downscaling; every 10 seconds, 
the CA decides to remove a node only when the resource utilization falls below 50%.

When the Cluster Autoscaler (CA) detects an unschedulable pod,it will create a new node,and the scheduler will automatically
schedule this pod onto the new node. **Note** that the CA doesn't look at the available memory or CPU when it triggers 
autoscaling; it focuses on the unschedulable pods.

- If you're using **AWS**, the CA will provision a new EC2 instance.
- If you're using **Azure**, the CA will create a new virtual machine.
- If you're on **GCP**, the CA will create a new compute engine.

![Kubernetes CA Worflow](https://github.com/balusena/kubernetes-for-devops/blob/main/15-Kubernetes%20Autoscaling/ca_workflow.png)

**Note we are using AWS cloud provider example for Cluster Autoscaler (CA):**
### 1.Create a ca.yaml file.
```
ubuntu@ip-172-31-47-64:~ $ nano ca.yaml
apiVersion: cluster-autoscaler.k8s.io/v1
kind: ClusterAutoscaler
metadata:
  name: cluster-autoscaler
  labels:
    k8s.io/cluster-autoscaler/enabled: "true"
    k8s.io/cluster/eks-prod-cluster: "true"
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cluster-autoscaler
  minNodes: 1
  maxNodes: 10
  cloudProvider: aws
```
### 2.Apply the ca.yaml File.
```
ubuntu@ip-172-31-47-64:~ $ kubectl apply -f ca.yaml
clusterautoscaler.cluster-autoscaler.k8s.io/cluster-autoscaler created
```
### 3.Check the status of the Cluster Autoscaler.
```
ubuntu@ip-172-31-47-64:~ $ kubectl get clusterautoscaler
NAME               AGE
cluster-autoscaler 1m
```
### 4.Describe the Cluster Autoscaler.
```
ubuntu@ip-172-31-47-64:~ $ kubectl describe clusterautoscaler cluster-autoscaler
Name:         cluster-autoscaler
Namespace:    default
Labels:       k8s.io/cluster-autoscaler/enabled=true
              k8s.io/cluster/eks-prod-cluster=true
Annotations:  <none>
API Version:  cluster-autoscaler.k8s.io/v1
Kind:         ClusterAutoscaler
Spec:
  Scale Target Ref:
    Api Version:  apps/v1
    Kind:         Deployment
    Name:         cluster-autoscaler
  Min Nodes:      1
  Max Nodes:      10
  Cloud Provider: aws
Events:
  Type    Reason              Age   From              Message
  ----    ------              ----  ----              -------
  Normal  Registered Nodes   1m    cluster-autoscaler  ClusterAutoscaler is watching the cluster for unschedulable pods
```

**To monitor how the Cluster Autoscaler reacts when it schedules new pods, you can follow these steps. This will include 
checking the nodes before and after the autoscaler schedules a pod. Here's how you can do it with the expected outputs:**

# Monitoring Cluster Autoscaler: Nodes Before and After Scheduling a Pod

### 1.Check Nodes Before Scheduling

Before triggering any action that would cause the Cluster Autoscaler to scale the cluster, check the current nodes in the cluster.
```
ubuntu@ip-172-31-47-64:~ $ kubectl get nodes
NAME                                STATUS   ROLES    AGE   VERSION
ip-172-31-47-64.ec2.internal          Ready    <role>   10m   v1.21.0
ip-172-31-47-65.ec2.internal          Ready    <role>   10m   v1.21.0
```
**Note:** This output shows that there are currently two nodes in the cluster.

### 2.Trigger Autoscaling by Scheduling a New Pod
Now, deploy a pod that requires more resources than the current nodes can provide. This will trigger the Cluster Autoscaler
to add new nodes.
```
ubuntu@ip-172-31-47-64:~ $ nano resource-intensive-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-intensive-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: resource-intensive
  template:
    metadata:
      labels:
        app: resource-intensive
    spec:
      containers:
      - name: stress-container
        image: vish/stress
        resources:
          requests:
            cpu: "2"        # Request 2 CPUs, which might be more than available on current nodes
            memory: "2Gi"
        args:
          - "-cpus"
          - "2"
```
### 3.Apply the Deployment.
```
ubuntu@ip-172-31-47-64:~ $ kubectl apply -f resource-intensive-deployment.yaml
deployment.apps/resource-intensive-deployment created
```
### 4.To get the list of pods in the cluster.
```
ubuntu@ip-172-31-47-64:~ $ kubectl get pods
NAME                                           READY   STATUS      RESTARTS   AGE
resource-intensive-deployment-5c5fbbcf5b-l9g76   0/1     Pending     0          2m
```
**Note:** The pod will be in the Pending state if there aren't enough resources on the current nodes.

### 5.Check the Cluster Autoscaler logs to see if it detected the unschedulable pod and started adding nodes.
```
ubuntu@ip-172-31-47-64:~ $ kubectl logs -f deployment/cluster-autoscaler -n kube-system
I0827 12:00:00.000000       1 scale_up.go:145] Scale-up: finding node group to scale up
I0827 12:00:00.000000       1 scale_up.go:185] Scale-up: scaling up node group <node-group> by 1
```
**Note:** This indicates that the Cluster Autoscaler has decided to scale up the cluster by adding a new node.

### 6.After the Cluster Autoscaler adds a new node, check the nodes in the cluster again.
```
ubuntu@ip-172-31-47-64:~ $ kubectl get nodes
NAME                                STATUS   ROLES    AGE   VERSION
ip-172-31-47-64.ec2.internal          Ready    <role>   10m   v1.21.0
ip-172-31-47-65.ec2.internal          Ready    <role>   10m   v1.21.0
ip-172-31-47-66.ec2.internal          Ready    <role>   1m    v1.21.0
```
### 7.Finally, check that the previously unschedulable pod is now running.
```
ubuntu@ip-172-31-47-64:~ $ kubectl get pods
NAME                                           READY   STATUS      RESTARTS   AGE
resource-intensive-deployment-5c5fbbcf5b-l9g76   1/1     Running     0         5m
```
**Note:** This shows that the pod is now running, confirming that the Cluster Autoscaler successfully scaled the cluster
and the scheduler placed the pod on the new node.

#### Summary:
- **Set Up an EKS Cluster:**
Create an EKS (Elastic Kubernetes Service) cluster on AWS using the AWS Management Console, CLI, or an Infrastructure as
Code tool like Terraform.

- **Install the Cluster Autoscaler on EKS:**
Deploy the Cluster Autoscaler in your EKS cluster by applying a cluster-autoscaler.yaml configuration file, ensuring it's
configured to work with your specific AWS environment.

- **Update IAM Policy to Allow EKS Cluster Autoscaler to Modify Nodes:**
Attach the necessary IAM (Identity and Access Management) policy to the EKS worker node role. This policy must grant 
permissions to the Cluster Autoscaler to modify node groups, such as scaling nodes up or down.

- **Verify the Setup of the Cluster Autoscaler on EKS:**
Check the logs of the Cluster Autoscaler pod to ensure it's running correctly. Additionally, test the scaling behavior 
by deploying resource-intensive workloads and observing whether the cluster scales up and down as expected.









 


