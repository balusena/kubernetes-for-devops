# Kubernetes Advanced Scheduling
Kubernetes advanced scheduling optimizes pod placement and resource management. Node Name and Node Selector assign pods 
to specific nodes or those with certain labels. Node Affinity provides flexible rules for node selection based on labels.
Pod Affinity/Anti-Affinity dictates how pods should be scheduled in relation to other pods. Taints and Tolerations control
pod placement on nodes with specific taints. Together, these features enhance resource efficiency, balance workloads, and
meet precise deployment needs.

![Kubernetes Advanced Scheduling](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/advanced_scheduling.png)

We know that the Kubernetes scheduler is responsible for scheduling pods onto nodes. But how does the Kubernetes scheduler
pick the most suitable nodes for a pod, and is it possible to customize this behavior? In this discussion, we will explore
how the Kubernetes scheduler works and how we can customize this behavior using:

- **NodeName**
- **NodeSelector**
- **Affinity**
  - **Node Affinity**
  - **Pod Affinity**
- **Taints and Tolerations**

## How Kubernetes Scheduler Works
In a Kubernetes cluster with many nodes, when a pod is created, the Kubernetes scheduler first filters the nodes to 
identify which ones are capable of running the pod. It then evaluates these feasible nodes by running a set of scoring 
functions, rating each node from 1 to 10. Finally, the scheduler selects the node with the highest score and assigns 
it to the pod.

![Kubernetes Scheduler Workflow 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_1.png)

The Kubernetes scheduler takes into account the pod's resource requirements and any customizations defined by us when 
scheduling the pod. Most of the time, we don't need to write custom scheduling logic, as the default scheduler effectively
handles this automatically.

![Kubernetes Scheduler Workflow 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_2.png)

For example, running our pods across multiple nodes ensures that if one node goes down, the pods on other nodes can 
still handle requests. 

![Kubernetes Scheduler Workflow 3](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_3.png)

![Kubernetes Scheduler Workflow 4](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_4.png)

Kubernetes handles rescheduling of deleted pods onto different nodes when using Deployments or StatefulSets. 

![Kubernetes Scheduler Workflow 5](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/kubernetes_scheduler_workflow_5.png) 

However,there may be cases where we want to deploy our pods onto specific nodes,such as nodes with SSDs attached or nodes
within the same availability zone for frequently communicating pods.

To achieve this,we need to instruct the Kubernetes scheduler.There are several methods to guide Kubernetes in scheduling
pods onto specific nodes.

### 1.Create a minikube cluster with 4 nodes.
```
ubuntu@balasenapathi:~$ minikube start --nodes 4
😄  minikube v1.31.1 on Ubuntu 20.04
✨  Automatically selected the docker driver. Other choices: virtualbox, ssh
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node minikube-m02 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
🔎  Verifying Kubernetes components...

👍  Starting worker node minikube-m03 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2,192.168.49.3
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3
🔎  Verifying Kubernetes components...

👍  Starting worker node minikube-m04 in cluster minikube
🚜  Pulling base image ...
⌛  Another minikube instance is downloading dependencies... 
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2,192.168.49.3,192.168.49.4
❗  This container is having trouble accessing https://registry.k8s.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3,192.168.49.4
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
### 2.To verify the nodes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
minikube       Ready    control-plane   17m     v1.27.3
minikube-m02   Ready    <none>          13m     v1.27.3
minikube-m03   Ready    <none>          9m33s   v1.27.3
minikube-m04   Ready    <none>          5m10s   v1.27.3
```

### Different methods to guide Kubernetes in scheduling pods onto specific nodes.

### 1.nodeName:
nodeName in Kubernetes specifies a particular node for pod scheduling. By setting nodeName in a pod specification, you 
direct Kubernetes to deploy the pod onto that exact node. While this ensures placement on a specific node, it is less 
flexible compared to using nodeSelector for broader scheduling control.

We have four nodes ready in our cluster, named `1.minikube`, `2.minikube-m02`, `3.minikube-m03`, and `4.minikube-m04`. 
Each node in the cluster has a unique name, and the simplest way to instruct Kubernetes to schedule a pod onto a specific
node is by using the node name.

![Kubernetes Node Name 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodename_1.png) 

![Kubernetes Node Name 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodename_2.png) 

### 3.Now create a simple deployment file todo-ui-deployment.yaml.
```
ubuntu@balasenapathi:~$ nano todo-ui-deployment.yaml
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
      nodeName: minikube-m02
      containers:
        - name: todo-ui
          image: balasenapathi/todo-ui:1.0.2
          ports:
            - containerPort: 80
          env:
            - name: "REACT_APP_BACKEND_SERVER_URL"
              value: "http://todo.com/api"
```
**Note:** Now let's set the `nodeName` in the deployment to `minikube-m02`, so that our pod is scheduled onto the specific
node `minikube-m02`. Please note that the scheduling information is specified at the pod level, not at the container level.
By setting this, we instruct Kubernetes to deploy all replicas of our deployment onto the second node in the cluster, i.e.,
`minikube-m02`.

### 4.Now apply the deployment changes in cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-deployment.yaml
deployment.apps/todo-ui created
```
**Note:** As we can see the deployemnt is created.

### 5.Now list down the pods in the cluster and give wide option to display the nodes also.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-ui-5ff8895fdf-h7z2x   1/1     Running   0          2m19s   10.244.1.2   minikube-m02   <none>           <none>
todo-ui-5ff8895fdf-s8s5r   1/1     Running   0          2m19s   10.244.1.3   minikube-m02   <none>           <none>
```
**Note:** As we can see, the two replicas `{todo-ui-5ff8895fdf-h7z2x, todo-ui-5ff8895fdf-s8s5r}` are both running on the
second node, `minikube-m02`. By specifying the `nodeName` in the deployment, Kubernetes schedules both replicas on the 
same node. Without the `nodeName`, these pods would likely be spread across different nodes. However, for high availability,
it's not advisable to schedule all replicas on the same node. To distribute pods across multiple nodes, you should use 
`nodeSelector` instead of `nodeName`.

### 2.nodeSelector:
nodeSelector in Kubernetes allows you to schedule pods onto nodes with specific labels. By defining nodeSelector in a 
pod's configuration, you instruct Kubernetes to match nodes with the given labels, ensuring that pods are placed on 
suitable nodes that meet predefined criteria, enhancing scheduling flexibility.

Just like we add labels to our pods and deployments, we can also add labels to nodes. You can assign any number of labels
to a node, and the same label can be applied to multiple nodes. In your pod or deployment manifest, you use `nodeSelector`
instead of `nodeName`, specifying key-value pairs (e.g., `team:analytics`).

![Kubernetes Node Selector 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodeselector_1.png)  

This instructs the scheduler to select nodes matching the specified label and deploy the pods onto those nodes.

![Kubernetes Node Selector 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodeselector_2.png)  

### 1.Add the label to the first node i.e minikube.
```
ubuntu@balasenapathi:~$ kubectl label node minikube team=analytics
node/minikube labeled
```
**Note:** The label should be specified in the form of key-value pairs, e.g., team=analytics. As shown, we have added 
this label to the first node, minikube.

### 2.Add the label to the third node i.e minikube-m03.
```
ubuntu@balasenapathi:~$ kubectl label node minikube-m03 team=analytics
node/minikube-m03 labeled
```
**Note:** As we can see, we have added the label to the first node, minikube, and the third node, minikube-m03.

### 3.Now verify the node labels in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get nodes --show-labels
NAME           STATUS   ROLES           AGE    VERSION   LABELS
minikube       Ready    control-plane   114m   v1.27.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube,kubernetes.io/os=linux,minikube.k8s.io/
commit=fd3f3801765d093a485d255043149f92ec0a695f,minikube.k8s.io/name=minikube,minikube.k8s.io/primary=true,minikube.k8s.io/
updated_at=2023_10_02T00_56_22_0700,minikube.k8s.io/version=v1.31.1,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-
from-external-load-balancers=,team=analytics
minikube-m02   Ready    <none>          109m   v1.27.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m02,kubernetes.io/os=linux
minikube-m03   Ready    <none>          106m   v1.27.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m03,kubernetes.io/os=linux,team=analytics
minikube-m04   Ready    <none>          101m   v1.27.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m04,kubernetes.io/os=linux
```
**Note: We can see all the labels of nodes.** 

### Labels of first node:minikube.
```
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube,kubernetes.io/os=linux,minikube.k8s.io/
commit=fd3f3801765d093a485d255043149f92ec0a695f,minikube.k8s.io/name=minikube,minikube.k8s.io/primary=true,minikube.k8s.io/
updated_at=2023_10_02T00_56_22_0700,minikube.k8s.io/version=v1.31.1,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-
from-external-load-balancers=,team=analytics
```
### Labels of second node:minikube-m02.
```
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m02,kubernetes.io/os=linux
```
### Labels of third node:minikube:m03.
```
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m03,kubernetes.io/os=linux,team=analytics
```
### Labels of fourth node:minikube-m04.
```
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m04,kubernetes.io/os=linux
```
**Note:** Kubernetes automatically adds default labels to each node, such as `beta.kubernetes.io/arch=amd64`, 
indicating the node architecture is amd64, and `beta.kubernetes.io/os=linux`, specifying the OS is Linux. Additionally,
we added the custom label `team=analytics` to both the first node (`minikube`) and the third node (`minikube-m03`).

### 4.To filter our nodes with specific labels in our cluster.
```
ubuntu@balasenapathi:~$ kubectl get nodes -l team=analytics 
NAME           STATUS   ROLES           AGE    VERSION
minikube       Ready    control-plane   126m   v1.27.3
minikube-m03   Ready    <none>          118m   v1.27.3
```
**Note:** Now we have identified all the nodes with the label team=analytics. Let's use this label in the nodeSelector 
of our deployment file to ensure the pods are scheduled onto these nodes. 

### 5.Now change the deployment file todo-ui-deployment.yaml.
```
ubuntu@balasenapathi:~$ nano todo-ui-deployment.yaml
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
      # nodeName: minikube-m02
      nodeSelector:
        team: analytics
      containers:
        - name: todo-ui
          image: balasenapathi/todo-ui:1.0.2
          ports:
            - containerPort: 80
          env:
            - name: "REACT_APP_BACKEND_SERVER_URL"
              value: "http://todo.com/api"
```

**Note:** When creating labels for nodes, we use = (e.g., team=analytics). However, in the nodeSelector specification of
the deployment file, you should use a colon to define the label as a key-value pair (e.g., team:analytics).

### 6.Now apply the deployment changes in cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-deployment.yaml
deployment.apps/todo-ui configured
```
**Note:** We can see our deployment is changed.

### 7.Now list down the pods in the cluster and give wide option to display the nodes also.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
todo-ui-5b8b7964b7-dckpj   1/1     Running   0          44s   10.244.0.3   minikube       <none>           <none>
todo-ui-5b8b7964b7-dvg9w   1/1     Running   0          44s   10.244.2.3   minikube-m03   <none>           <none>
```
**Note:** As you can see, our pods are scheduled onto the first node, minikube, and the third node, minikube-m03. This 
is because we specified the nodeSelector as team:analytics, and these are the nodes that have the label team=analytics.

nodeSelector is straightforward and useful for placing pods on specific nodes based on labels. However, it lacks 
flexibility for more complex conditions, such as selecting nodes with label values greater than a certain number. 

![Kubernetes Node Selector 3](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodeselector_3.png)  

To address these needs, nodeAffinity was introduced as a more powerful mechanism, offering advanced scheduling 
capabilities and allowing for more nuanced criteria.

### 3.Affinity:
Affinity is an alternative to nodeSelector and allows us to instruct Kubernetes to schedule our pods onto specific 
nodes using advanced operators. It is divided into two main types:
   - **1.Node Affinity**
   - **2.Pod Affinity/Pod Anti Affinity**

#### 1.Node Affinity:
NodeAffinity enables pods to be scheduled on nodes that match specific criteria, such as having particular labels or 
satisfying certain conditions. It allows you to express complex rules, like scheduling pods on nodes with labels that 
fall within a specific range of values or that meet certain requirements.

#### Node Affinity is further divided into two types:
- **1.Required During Scheduling, Ignore During Execution**
- **2.Preffered During Scheduling, Ignore During Execution**

![Kubernetes Node Affinity 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodeaffinity_1.png)  

#### 1.Required During Scheduling, Ignore During Execution: 
This type of affinity specifies rules that must be met during the scheduling of a pod. If the conditions specified are 
not met, the pod will not be scheduled on that node. Once scheduled, Kubernetes does not enforce these rules during the 
pod's execution.
   
**Required During Scheduling** means that whatever labels we specify in the pod definition, the node must have those same 
labels during scheduling. If the node does not have the specified labels, the pod will remain in a pending state. These
labels are mandatory for the pod to be scheduled.

**Ignore During Execution** means that once the pod is already running on a node, it won’t be impacted even if the node’s
labels change or no longer match. Essentially, this means not to disturb the running pods, regardless of any changes to 
the node labels.

### 1.Now create a simple deployment file todo-ui-deployment.yaml.
```
ubuntu@balasenapathi:~$ nano todo-ui-deployment.yaml
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
      # nodeName: minikube-m02
      # nodeSelector:
      #   team: analytics
      affinity:
        nodeAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
             nodeSelectorTerms:
               - matchExpressions:
                   - key: rank
                     operator: Lt
                     values:
                       - "5"
      containers:
        - name: todo-ui
          image: balasenapathi/todo-ui:1.0.2
          ports:
            - containerPort: 80
          env:
            - name: "REACT_APP_BACKEND_SERVER_URL"
              value: "http://todo.com/api"
```
**Note:** We assign node labels using `matchExpressions`, where we specify the key-value pair. 

- **Key:** "rank"
- **Operators:**
  - **Gt:** Greater than
  - **Lt:** Less than
  - **In:** Contains specified values
  - **NotIn:** Does not contain specified values
  - **Exists:** Contains specified labels
  - **DoesNotExist:** Does not contain specified labels

For this case,we'll use **Lt** (Less than) and set the value to "5".In the Affinity section,we are instructing Kubernetes
to choose a node with a label key "rank" where the value is less than 5.

### 2.Now apply the deployment changes in cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-deployment.yaml
deployment.apps/todo-ui configured
```
**Note:** We can see our deployment is changed.

### 3.Now list down the pods in the cluster and give wide option to display the nodes also.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
todo-ui-5f6567cc9d-6zlbc   0/1     Pending   0          6s    <none>   <none>   <none>           <none>
todo-ui-5f6567cc9d-cds9h   0/1     Pending   0          6s    <none>   <none>   <none>           <none>
```
**Note:** The pod is currently in a pending state. This is because we've specified the label as `rank` in the pod's 
affinity rules, but no node in the cluster has this label. Additionally, since we used the `requiredDuringScheduling` 
rule, these labels must exist on the node during scheduling for the pod to be deployed. Without the matching label, 
the pod cannot be scheduled.

### 4.Now add these labels to the nodes 2 and 3 in the cluster.

#### Add the label to the second node i.e minikube-m02.
```
ubuntu@balasenapathi:~$ kubectl label node minikube-m02 rank=3
node/minikube-m02 labeled
```
#### Add the label to the third node i.e minikube-m03.
```
ubuntu@balasenapathi:~$ kubectl label node minikube-m03 rank=5
node/minikube-m03 labeled
```
**Note: Added labels to the second and third nodes i.e, minikube-m02,  minikube-m03**

### 5.Now list down the pods in the cluster with above changes.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-ui-5f6567cc9d-6zlbc   1/1     Running   0          8m27s   10.244.1.6   minikube-m02   <none>           <none>
todo-ui-5f6567cc9d-cds9h   1/1     Running   0          8m27s   10.244.1.5   minikube-m02   <none>           <none>
```
**Note:** The pods are now running successfully after we added the appropriate label to the nodes. Both pods are scheduled 
on the same node because our `matchExpression` specifies that the `rank` should be less than 5. The third node has a `rank`
of 5, which doesn't meet this criterion. Even if we delete these labels from the nodes, the pods won't be killed because 
we used `IgnoreDuringExecution`, meaning the labels are ignored once the pods are running. Instead of just 
`requiredDuringScheduling`, we can also use `preferredDuringScheduling` or even combine both in `nodeAffinity`.

#### 2.Preferred During Scheduling, Ignore During Execution: 
This type of affinity expresses a preference for certain nodes during pod scheduling. However, it does not strictly 
enforce this preference. If the preferred conditions are not met, the pod can still be scheduled on other nodes. Like
the required type, this does not impact the pod during execution.
   
Let's consider we have four nodes with specific labels, and we're using `preferredDuringScheduling` for node affinity. 
Here, we're setting preferences among the nodes. In this example, we prefer nodes with the label `region: us-east-1`
three times more than those labeled `team: analytics`.

![Kubernetes Node Affinity 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodeaffinity_2.png)  
 
We've set this preference with a weight, so Kubernetes will rank the nodes based on these preferences and choose the node 
with the highest rank.

![Kubernetes Node Affinity 3](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/nodeaffinity_3.png)  

**Note:** This is a soft rule, meaning it's not mandatory. Even if the scheduler doesn't find a node with these 
labels, the pod will still be deployed to another node. This is why Kubernetes ranks the fourth node, even though it 
lacks any of the labels that the pod prefers.

### 1.Now create a simple deployment file todo-ui-deployment.yaml.
```
ubuntu@balasenapathi:~$ nano todo-ui-deployment.yaml
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
      # nodeName: minikube-m02
      # nodeSelector:
      #   team: analytics
      affinity:
        nodeAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
             nodeSelectorTerms:
               - matchExpressions:
                   - key: rank
                     operator: Lt
                     values:
                       - "5"
           preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 40
              preference:
                matchExpressions:
                - key: "team"
                  operator: In
                  values: ["analytics"]
            - weight: 60
              preference:
                matchExpressions:
                - key: "rank"
                  operator: Gt
                  values: ["4"]
      containers:
        - name: todo-ui
          image: balasenapathi/todo-ui:1.0.2
          ports:
            - containerPort: 80
          env:
            - name: "REACT_APP_BACKEND_SERVER_URL"
              value: "http://todo.com/api"
```
**Note:** Here, we're setting our preferences for node scheduling. The first preference is for nodes with a label `rank`
that is greater than 4. The second preference is for nodes labeled `team: analytics`. This setup guides the Kubernetes 
scheduler to prioritize nodes matching the first condition, and if those are not available, it will consider the nodes 
with the second label.

### 2.Now apply the deployment changes in cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-deployment.yaml
deployment.apps/todo-ui configured
```
**Note: We can see our deployment is changed.**

### 3.Now list down the pods in the cluster and give wide option to display the nodes also.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
todo-ui-7f8fc96c5b-g48kt   1/1     Running   0          26s   10.244.1.7   minikube-m02   <none>           <none>
todo-ui-7f8fc96c5b-zcgzm   1/1     Running   0          23s   10.244.1.8   minikube-m02   <none>           <none>
```
**Note:** As we can see, the pods remain on the same node because Kubernetes first filters nodes based on the required
labels (rank < 5), which matches the second node. Then, it applies the preferences, but since there are no nodes meeting
these preferences, it defaults to the second node. If you remove the `nodeAffinity: requiredDuringSchedulingIgnoredDuringExecution:`
section, the scheduler will no longer enforce these strict label requirements, and the pods may be scheduled to other nodes
based on preferences alone.

### 4.Now create a simple deployment file todo-ui-deployment.yaml.
```
ubuntu@balasenapathi:~$ nano todo-ui-deployment.yaml

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
      # nodeName: minikube-m02
      # nodeSelector:
      #   team: analytics
      affinity:
        nodeAffinity:
          # requiredDuringSchedulingIgnoredDuringExecution:
          #   nodeSelectorTerms:
          #     - matchExpressions:
          #         - key: rank
          #           operator: Lt
          #           values:
          #             - "5"
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 40
              preference:
                matchExpressions:
                - key: "team"
                  operator: In
                  values: ["analytics"]
            - weight: 60
              preference:
                matchExpressions:
                - key: "rank"
                  operator: Gt
                  values: ["4"]
      containers:
        - name: todo-ui
          image: balasenapathi/todo-ui:1.0.2
          ports:
            - containerPort: 80
          env:
            - name: "REACT_APP_BACKEND_SERVER_URL"
              value: "http://todo.com/api"
```
### 5.Now apply the deployment changes in cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-deployment.yaml
deployment.apps/todo-ui configured
```
**Note: We can see our deployment is changed.**

### 6.Now list down the pods in the cluster and give wide option to display the nodes also:
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
todo-ui-75d9ff8dd4-9vrxf   1/1     Running   0          8s    10.244.2.4   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj   1/1     Running   0          4s    10.244.2.5   minikube-m03   <none>           <none>
```
**Note:** As we can see, our pods got deployed onto the third node, "minikube-m03." This is because we gave higher preference
to nodes with a rank greater than 4, which matches the third node. This demonstrates how node affinity allows for more 
flexible and nuanced scheduling conditions.

**Please note that:**

1. **If we define both `nodeAffinity` and `nodeSelector`, both must be satisfied for the pod to be scheduled onto a node.**

2. **If we define multiple `nodeSelector` terms associated with `nodeAffinity`, the pod can be scheduled onto a node if 
   one of the specified `nodeSelector` terms can be satisfied.**

3. **If we define multiple `matchExpressions` associated with a single `nodeSelector` term, the pod can be scheduled onto
   a node only if all `matchExpressions` are satisfied.**

#### 2.Pod Affinity:
`PodAffinity` controls how pods are placed relative to other pods. It ensures that pods are scheduled on the same node 
or close to each other, which can be useful for optimizing network latency or co-locating related services. Conversely, 
`PodAntiAffinity` ensures that pods are spread across different nodes, improving resilience and fault tolerance.

We learned that while `nodeAffinity` is used to schedule pods onto specific nodes, `podAffinity` is used to co-locate pods. 
For example, if we want two applications that communicate frequently to be deployed in the same region for reduced latency,
we use `podAffinity`.

![Kubernetes Pod Affinity 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/podaffinity_1.png)  

Consider our "todo-ui" and "todo-api" applications. If "todo-ui" is deployed on a node in the "us-east-1" region, we can 
use `podAffinity` to deploy "todo-api" in the same region. We define the label selector and the "topologykey" in `podAffinity`.
Kubernetes scheduler then ensures that the new pod is scheduled onto a node with the same label, so both applications are
co-located.

![Kubernetes Pod Affinity 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/podaffinity_2.png)  

![Kubernetes Pod Affinity 3](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/podaffinity_3.png) 
 
In contrast, there might be cases where we want to spread pods across different nodes or regions to prevent performance 
issues due to co-location. This strategy helps ensure that a failure in one node or region does not affect the entire service.

![Kubernetes Pod Affinity 4](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/podaffinity_4.png)  

### 1.Create a todo-api-deployment in cluster:
```
ubuntu@balasenapathi:~$ nano todo-api-deployment.yaml
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
          image: balasenapathi/todo-api:1.0.2
          ports:
            - containerPort: 8082
          env:
            - name: "spring.data.mongodb.uri"
              value: "mongodb+srv://root:321654@cluster0.p9jq2.mongodb.net/todo?retryWrites=true&w=majority"
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - "todo-ui"
              topologyKey: kubernetes.io/hostname      
```
Note: Here we define podAffinity under the affinity section similar to the nodeAffinity.


#### podAffinity is categorized into two types:

- 1.Soft
- 2.Hard

1. **Soft Affinity**: This type is a preference, not a strict requirement. If the preferred conditions (like co-locating
pods on the same node or region) are met, the Kubernetes scheduler will try to satisfy them. However, if the conditions
cannot be met, the pod will still be scheduled on a different node or region. This is helpful when the desired placement
improves performance but is not critical.

2. **Hard Affinity**: This type is a strict requirement. The pod can only be scheduled on a node if the specified 
conditions are met. If no nodes satisfy these conditions, the pod will remain unscheduled (in a pending state) until a
suitable node becomes available. This is used when specific co-location is crucial for the application's functionality.

Here, we are instructing Kubernetes to select the pods with the "todo-ui" label and use "hostname" as the topologyKey, 
which represents the node name. The expectation is that these pods will be deployed on the same node because the 
topologyKey is set to "hostname," matching the node where the UI application is deployed, as specified by the UI pod label.

### 2.Now apply the deployment changes in cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-api-deployment.yaml
deployment.apps/todo-api created
```

### 3.Now list down the pods in the cluster and give wide option to display the nodes also:
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-api-d7c6465b5-jg6tg   1/1     Running   0          4m10s   10.244.2.7   minikube-m03   <none>           <none>
todo-api-d7c6465b5-q4wfr   1/1     Running   0          4m10s   10.244.2.6   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-9vrxf   1/1     Running   0          175m    10.244.2.4   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj   1/1     Running   0          175m    10.244.2.5   minikube-m03   <none>           <none>
```

**Note:** As we can see, the API pods are also deployed onto the same node where the UI pods are deployed, i.e., 
"minikube-m03." This demonstrates how we can co-locate pods on the same node, in the same availability zone, or within 
the same region.

### podAntiAffinity:
Pod anti-affinity in Kubernetes ensures that specific pods are not scheduled on the same node or within the same topology
as others. This feature helps distribute pods across different nodes, zones, or regions, reducing the risk of resource 
contention and minimizing the impact of node failures. By enforcing separation, pod anti-affinity enhances application 
resilience, ensuring critical workloads remain operational even during node failures, thus supporting high availability
and fault tolerance within the cluster.

### 1.Create a todo-api-deployment in cluster:
```
ubuntu@balasenapathi:~$ nano todo-api-deployment.yaml
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
          image: balasenapathi/todo-api:1.0.2
          ports:
            - containerPort: 8082
          env:
            - name: "spring.data.mongodb.uri"
              value: "mongodb+srv://root:321654@cluster0.p9jq2.mongodb.net/todo?retryWrites=true&w=majority"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - "todo-ui"
              topologyKey: kubernetes.io/hostname      
```
Note: Now, we are specifying that for pods matching the UI pod label, the scheduler should retrieve the value of the 
topologyKey, which is the hostname. With anti-affinity, we are instructing the scheduler not to co-locate these pods. 
The expectation is that the API pods should never be scheduled on the same node as the UI pods, since we defined the 
topologyKey as the hostname and used the UI pod label in the matchExpression.

### 2.Now apply the changes in the cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-api-deployment.yaml
deployment.apps/todo-api configured
```
### 3.Now list down all the pods in the cluster
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-api-67d9f8f977-d6l2m   1/1     Running   0          5m11s   10.244.1.9   minikube-m02   <none>           <none>
todo-api-67d9f8f977-pmzfk   1/1     Running   0          3m10s   10.244.3.3   minikube-m04   <none>           <none>
todo-ui-75d9ff8dd4-9vrxf    1/1     Running   0          3h13m   10.244.2.4   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj    1/1     Running   0          3h13m   10.244.2.5   minikube-m03   <none>           <none>
```
Note: We can see that the API pods are scheduled onto nodes "minikube-m02" and "minikube-m04," but not onto "minikube-m03.
" This is because of the podAntiAffinity rule, which ensures that API pods do not co-locate with the UI pods on "minikube-m03."

### 4.Taints and Tolerations:

Taints and Tolerations in Kubernetes control how pods are scheduled onto nodes. Taints allow nodes to repel pods unless 
those pods have matching Tolerations. A taint on a node prevents pods from being scheduled onto it unless they have a 
corresponding toleration that allows them to be scheduled there. This mechanism helps in managing which pods can run on 
which nodes, ensuring proper workload distribution and resource utilization across the cluster.

We understood that Affinity attracts pods to certain nodes. Conversely, to keep pods away from specific nodes, we use 
Taints and Tolerations.

Imagine Bob uses mosquito repellent to keep mosquitoes away. The repellent is like a taint on a node, preventing
mosquitoes (pods) from landing.

![Kubernetes Taints Toleartion 1](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/taints_toleration_1.png)   

![Kubernetes Taints Toleartion 2](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/taints_toleration_2.png)   

However, if another type of bug is tolerant to the repellent, it can land on Bob's body. Similarly, Taints on nodes repel
pods that don’t have the corresponding Toleration, while those with matching Toleration can still be scheduled on these 
nodes.

![Kubernetes Taints Toleartion 3](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/taints_toleration_3.png)   

#### There are 3 types of Taint effects:
- **1.NoSchedule (Hard):** Pods cannot be scheduled onto the node if they do not tolerate the taint. This is a strict 
  rule, meaning that if a pod cannot tolerate the taint, it will not be scheduled on that node.

- **2.PreferNoSchedule (Soft):** Pods can be scheduled onto the node if no other nodes are available, even if they do 
  not tolerate the taint. This is a less strict rule, preferring to avoid the node but allowing scheduling if necessary.

- **3.NoExecute (Strict):** This effect not only prevents new pods from being scheduled on the node if they do not 
  tolerate the taint but also evicts running pods that cannot tolerate the newly added taint. This is a very strict 
  rule that enforces taint toleration for both scheduling and running pods.

### 1.So we can add it into the node Now taint a second node minikube-m02 in the cluster.
```
ubuntu@balasenapathi:~$ kubectl taint node minikube-m02 env=production:NoExecute
node/minikube-m02 tainted
```
**Note:** The environment env=production with the taint effect NoExecute (i.e., env=production:NoExecute) has been 
applied to node2 (i.e., minikube-m02). With this taint, we expect that any pods already running on minikube-m02 that do 
not tolerate this taint will be evicted from the node.

### 2.Now verify this by listing down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                        READY   STATUS           RESTARTS    AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-api-67d9f8f977-782rd   0/1     ContainerCreating       0    17s     <none>       minikube-m04   <none>           <none>
todo-api-67d9f8f977-d6l2m   1/1     Terminating             0    86m     10.244.1.9   minikube-m02   <none>           <none>
todo-api-67d9f8f977-pmzfk   1/1     Running                 0    84m     10.244.3.3   minikube-m04   <none>           <none>
todo-ui-75d9ff8dd4-9vrxf    1/1     Running                 0    4h35m   10.244.2.4   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj    1/1     Running                 0    4h35m   10.244.2.5   minikube-m03   <none>           <none>
```
**Note:** As observed, the pod previously running on node2 (i.e., minikube-m02) has been terminated and is now being 
created on a different node, specifically node4 (i.e., minikube-m04). This occurs because the pod cannot tolerate the 
NoExecute taint applied to minikube-m02, resulting in its eviction from the node.

### 3.Now verify this by listing down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-api-67d9f8f977-782rd   1/1     Running   0          39s     10.244.3.4   minikube-m04   <none>           <none>
todo-api-67d9f8f977-pmzfk   1/1     Running   0          85m     10.244.3.3   minikube-m04   <none>           <none>
todo-ui-75d9ff8dd4-9vrxf    1/1     Running   0          4h35m   10.244.2.4   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj    1/1     Running   0          4h35m   10.244.2.5   minikube-m03   <none>           <none>
```

### 4.If we want to look at the taints that are applied on a node we should describe the node i.e, minikube-02:
```
ubuntu@balasenapathi:~$ kubectl describe node minikube-m02
Name:               minikube-m02
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=minikube-m02
                    kubernetes.io/os=linux
                    rank=3
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/cri-dockerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 02 Oct 2023 18:22:12 +0530
Taints:             env=production:NoExecute
Unschedulable:      false
Lease:
  HolderIdentity:  minikube-m02
  AcquireTime:     <unset>
  RenewTime:       Tue, 03 Oct 2023 01:59:12 +0530
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Tue, 03 Oct 2023 01:58:58 +0530   Mon, 02 Oct 2023 21:44:06 +0530   KubeletHasSufficientMemory   kubelet has 
  sufficient memory available
  DiskPressure     False   Tue, 03 Oct 2023 01:58:58 +0530   Mon, 02 Oct 2023 21:44:06 +0530   KubeletHasNoDiskPressure     kubelet has 
  no disk pressure
  PIDPressure      False   Tue, 03 Oct 2023 01:58:58 +0530   Mon, 02 Oct 2023 21:44:06 +0530   KubeletHasSufficientPID      kubelet has 
  sufficient PID available
  Ready            True    Tue, 03 Oct 2023 01:58:58 +0530   Mon, 02 Oct 2023 21:44:06 +0530   KubeletReady                 kubelet is 
  posting ready status
Addresses:
  InternalIP:  192.168.49.3
  Hostname:    minikube-m02
Capacity:
  cpu:                2
  ephemeral-storage:  102107096Ki
  hugepages-2Mi:      0
  memory:             3969504Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  102107096Ki
  hugepages-2Mi:      0
  memory:             3969504Ki
  pods:               110
System Info:
  Machine ID:                 b1595f30b59c403d8a2c6d6b807e8fae
  System UUID:                a24bd237-c239-4b1c-b82b-a3fccc0ef03f
  Boot ID:                    db017686-fa13-4fbd-8064-afde19206136
  Kernel Version:             5.15.0-84-generic
  OS Image:                   Ubuntu 22.04.2 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://24.0.4
  Kubelet Version:            v1.27.3
  Kube-Proxy Version:         v1.27.3
PodCIDR:                      10.244.1.0/24
PodCIDRs:                     10.244.1.0/24
Non-terminated Pods:          (1 in total)
  Namespace                   Name                CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                ------------  ----------  ---------------  -------------  ---
  kube-system                 kube-proxy-wnpw4    0 (0%)        0 (0%)      0 (0%)           0 (0%)         7h37m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests  Limits
  --------           --------  ------
  cpu                0 (0%)    0 (0%)
  memory             0 (0%)    0 (0%)
  ephemeral-storage  0 (0%)    0 (0%)
  hugepages-2Mi      0 (0%)    0 (0%)
Events:              <none>
```
**Note:** In "Taints: env=production:NoExecute" these are the taints applied on to the node2 i.e "minikube-m02"

### 5.We can delete a taint by using:
```
ubuntu@balasenapathi:~$ kubectl taint node minikube-m02 env=production:NoExecute-
node/minikube-m02 untainted
```
**Note:** By adding "-" at the end of cmd Now the taint on node2 i.e "minikube-m02" untainted meaning the taint is deleted.

### 6.Now add a taint with different effect now NoSchedule on node 2 i.e, "minikube-m02":
```
ubuntu@balasenapathi:~$ kubectl taint node minikube-m02 env=production:NoSchedule
node/minikube-m02 tainted
```
Please note that the taint is applied to the node minikube-m02, and the toleration is added to the pod. Before applying
the toleration, let’s first add the nodeAffinity to ensure that the pod is deployed on the second node, minikube-m02.

### 7.Create a todo-api-deployment in cluster:
```
ubuntu@balasenapathi:~$ nano todo-api-deployment.yaml
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
          image: balasenapathi/todo-api:1.0.2
          ports:
            - containerPort: 8082
          env:
            - name: "spring.data.mongodb.uri"
              value: "mongodb+srv://root:321654@cluster0.p9jq2.mongodb.net/todo?retryWrites=true&w=majority"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: rank
                    operator: Lt
                    values:
                      - "5" 
```
### 8.Now apply the changes in the cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-api-deployment.yaml
deployment.apps/todo-api configured
```
### 9.Now list down all the pods in the cluster
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
todo-api-67d9f8f977-22ps2   1/1     Running   0          2m30s   10.244.3.6   minikube-m04   <none>           <none>
todo-api-67d9f8f977-hn6rq   1/1     Running   0          4m13s   10.244.3.5   minikube-m04   <none>           <none>
todo-api-8675d4bfb7-mn2pg   0/1     Pending   0          5s      <none>       <none>         <none>           <none>
todo-ui-75d9ff8dd4-9vrxf    1/1     Running   0          5h19m   10.244.2.4   minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj    1/1     Running   0          5h19m   10.244.2.5   minikube-m03   <none>           <none>
```
**Note:** As we can see, the pod todo-api-8675d4bfb7-mn2pg is stuck in a pending state, and existing pods remain unaffected.
This is because the NoSchedule taint effect only impacts pod scheduling and does not affect running pods. The pending state
occurs because the scheduler attempts to schedule the pod on the second node (which matches the nodeAffinity criteria) but
the node has a taint that the pod does not tolerate. To resolve this, the pod must have a toleration for the taint applied
to the node.

- Note: So now add the toleration in toleartion section in manifest file.So now this matches the taint that we added:

### 10.Create a todo-api-deployment in cluster:
```
ubuntu@balasenapathi:~$ nano todo-api-deployment.yaml
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
          image: balasenapathi/todo-api:1.0.2
          ports:
            - containerPort: 8082
          env:
            - name: "spring.data.mongodb.uri"
              value: "mongodb+srv://root:321654@cluster0.p9jq2.mongodb.net/todo?retryWrites=true&w=majority"
      tolerations:
        - key: "env"
          operator: "Equal"
          value: "production"
          effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: rank
                    operator: Lt
                    values:
                      - "5"
```

### 11.Now apply the changes in the cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-api-deployment.yaml
deployment.apps/todo-api configured
```
### 12.Now list down all the pods in the cluster
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
todo-api-6bb64f8456-hjnf2   1/1     Running   0          9s      10.244.1.13   minikube-m02   <none>           <none>
todo-api-6bb64f8456-w2lsf   1/1     Running   0          7s      10.244.1.14   minikube-m02   <none>           <none>
todo-ui-75d9ff8dd4-9vrxf    1/1     Running   0          5h31m   10.244.2.4    minikube-m03   <none>           <none>
todo-ui-75d9ff8dd4-j27lj    1/1     Running   0          5h31m   10.244.2.5    minikube-m03   <none>           <none>
```
**Note:** Now we can see that the API pods are running on the second node, "minikube-m02," because the pods have been 
configured to tolerate the applied taint.

Finally, the last taint effect is PreferNoSchedule, which means Kubernetes will try to avoid scheduling pods on nodes 
with this taint but will allow it if no other nodes are available. This is a soft rule.

#### Summary:

![Kubernetes Advanced Scheduling](https://github.com/balusena/kubernetes-for-devops/blob/main/14-Kubernetes%20Advanced%20Scheduling/summary.png)   











