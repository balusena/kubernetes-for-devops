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
### Labels of third node:minikube:m03:
```
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m03,kubernetes.io/os=linux,team=analytics
```
### Labels of fourth node:minikube-m04:
```
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
arch=amd64,kubernetes.io/hostname=minikube-m04,kubernetes.io/os=linux
```
**Note:** Kubernetes automatically adds default labels to each node, such as `beta.kubernetes.io/arch=amd64`, 
indicating the node architecture is amd64, and `beta.kubernetes.io/os=linux`, specifying the OS is Linux. Additionally,
we added the custom label `team=analytics` to both the first node (`minikube`) and the third node (`minikube-m03`).

### 4.To filter our nodes with specific labels in our cluster:
```
ubuntu@balasenapathi:~$ kubectl get nodes -l team=analytics 
NAME           STATUS   ROLES           AGE    VERSION
minikube       Ready    control-plane   126m   v1.27.3
minikube-m03   Ready    <none>          118m   v1.27.3
```
**Note:** Now we have identified all the nodes with the label team=analytics. Let's use this label in the nodeSelector 
of our deployment file to ensure the pods are scheduled onto these nodes. 

### 5.Now change the deployment file todo-ui-deployment.yaml:
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

### 6.Now apply the deployment changes in cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f todo-ui-deployment.yaml
deployment.apps/todo-ui configured
```
**Note:** We can see our deployment is changed.

### 7.Now list down the pods in the cluster and give wide option to display the nodes also:
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
todo-ui-5b8b7964b7-dckpj   1/1     Running   0          44s   10.244.0.3   minikube       <none>           <none>
todo-ui-5b8b7964b7-dvg9w   1/1     Running   0          44s   10.244.2.3   minikube-m03   <none>           <none>
```
**Note:** As you can see, our pods are scheduled onto the first node, minikube, and the third node, minikube-m03. This 
is because we specified the nodeSelector as team:analytics, and these are the nodes that have the label team=analytics.













