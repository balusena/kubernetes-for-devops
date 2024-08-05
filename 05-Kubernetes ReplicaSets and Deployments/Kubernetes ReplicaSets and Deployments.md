# Kubernetes ReplicaSets and Deployments

## ReplicaSets:
ReplicaSet object is used to maintain a stable set of replicated pods running within a cluster at any given 
time. Its purpose is to maintain the specified number of pods and prevent the user from loosing the control 
of the application in case of any pod failure or inaccessible. Whenever any pod gets failed replicaset 
immediately launches another pod and replacing it with failed pod.

**Replicaset has following features:**

- A pod template is used to create a new pod whenever an existing pod fails and also replica count is also 
  maintained by defining the desired number of replicas that a controller needs to be running.

- A replica set also ensures that additional pod needs to be created or deleted whenever instance with same
  label is created.

- Replcaset allows to have multiple replicas of pod which means that the traffic is sent to different 
  instances which prevents single instance from being overloaded.

- Replcaset ensures it has multiple replicas of application so that it won't fail because of one pod fails.

![Kubernetes ReplicaSet](https://github.com/balusena/kubernetes-for-devops/blob/main/05-Kubernetes%20ReplicaSets%20and%20Deployments/kubernetes_replicaset.png)

### 1.Creating ReplicaSets with YAML.
```
#1.Clone this repository and move to example folder

git clone https://github.com/balusena/kubernetes-for-devops.git
cd  05-Kubernetes ReplicaSets and Deployments/examples
```
### 2.To get the apiVersion of replicasets in kubernetes.
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep replicaset
replicasets            rs           apps/v1            true         ReplicaSet
```
### 3.Create a replicaset using file nginx-replicaset.yaml
```
ubuntu@balasenapathi:~$ nano nginx-replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
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
          image: nginx:latest
          ports:
            - containerPort: 80
```
### 4.To create the replicaset in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-replicaset.yaml
replicaset.apps/nginx-replicaset created
```
### 5.To check whether replicaset is created in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   3         3         3       2m25s

ubuntu@balasenapathi:~$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   3         3         3       2m35s
```
### 6.To get the list of pods running in the kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-h2dqc   1/1     Running   0          4m39s
nginx-replicaset-jsdvj   1/1     Running   0          4m39s
nginx-replicaset-lb77x   1/1     Running   0          4m39s
```
## Self-Healing in Kubernetes

**Note:** Whenever a pod goes down, bringing it back into normal operation is called self-healing.

### 7.To delete a pod from the Kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl delete pod nginx-replicaset-h2dqc
pod "nginx-replicaset-h2dqc" deleted
```
**Note:** We can see that the pod is deleted from the Kubernetes cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-d466v   1/1     Running   0          58s
nginx-replicaset-jsdvj   1/1     Running   0          11m
nginx-replicaset-lb77x   1/1     Running   0          11m
```
**Note:** We can see that the pod is recreated by the ReplicaSet to ensure that the specified number of 
replicas is maintained. This process is called self-healing and is automatically managed by the 
ReplicaSet, which recreates the pod after it has been deleted.

## High Availability in Kubernetes

**Note:** To ensures systems remain operational with minimal downtime, even when some nodes fail.

Adding a Node to the Cluster. First, let's add one more node to the cluster to ensure high availability.

### 8.List of Nodes in the Kubernetes Cluster
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
local-cluster       Ready    control-plane   47h   v1.27.3
local-cluster-m02   Ready    <none>          51m   v1.27.3
```
### 9.Adding a Node to the Cluster
```
ubuntu@balasenapathi:~$ minikube node add --worker -p local-cluster
😄  Adding node m03 to cluster local-cluster
👍  Starting worker node local-cluster-m03 in cluster local-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
🔎  Verifying Kubernetes components...
🏄  Successfully added m03 to local-cluster!
```
### 10.List of Nodes After Adding the New Node
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE    VERSION
local-cluster       Ready    control-plane   47h    v1.27.3
local-cluster-m02   Ready    <none>          61m    v1.27.3
local-cluster-m03   Ready    <none>          110s   v1.27.3
```
### 11.Deleting a Node and Observing Pod Behavior

**Note:** Before deleting a node, let's see on which nodes the pods are currently running.

List of Pods and Their Nodes
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-replicaset-d466v   1/1     Running   0          18m   10.244.0.6   local-cluster       <none>           <none>
nginx-replicaset-jsdvj   1/1     Running   0          28m   10.244.1.2   local-cluster-m02   <none>           <none>
nginx-replicaset-lb77x   1/1     Running   0          28m   10.244.1.3   local-cluster-m02   <
```
### 12.Deleting a Node from the Cluster
```
ubuntu@balasenapathi:~$ minikube node delete local-cluster-m02 -p local-cluster
🔥  Deleting node local-cluster-m02 from cluster local-cluster
✋  Stopping node "local-cluster-m02"  ...
🛑  Powering off "local-cluster-m02" via SSH ...
🔥  Deleting "local-cluster-m02" in docker ...
💀  Node local-cluster-m02 was successfully deleted.
```
### 13.List of Pods After Deleting a Node
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
nginx-replicaset-d466v   1/1     Running   0          23m    10.244.0.6   local-cluster       <none>           <none>
nginx-replicaset-szqkn   1/1     Running   0          109s   10.244.2.3   local-cluster-m03   <none>           <none>
nginx-replicaset-v7pb5   1/1     Running   0          109s   10.244.2.2   local-cluster-m03   
```
**Note:** As we can see, the pods are automatically recreated on the available active running node 
"local-cluster-m03" after deleting the node "local-cluster-m02". This demonstrates how Kubernetes 
ensures high availability by redistributing pods to available nodes when one node is removed.

## Deployment
Deployment is used to maintain the versions of your application running in the pod. It helps to efficiently
scale the number of replica pods and enable the rollout and rollback to an earlier deployment version. 
Deployments runs on multiple replicas of the pod and automatically replaces a pod in case of any failure or
unresponsive. In this way Deployments helps to ensure that one or more instances of your application are 
available to serve user requests.

Deployments use a Pod template, which contains a specification for its Pods. The Pod specification 
determines how each Pod should look like: what applications should run inside its containers, which 
volumes the Pods should mount, its labels, and more. When a Deployment's Pod template is changed, new 
Pods are automatically created one at a time.

![Kubernetes Deployment](https://github.com/balusena/kubernetes-for-devops/blob/main/05-Kubernetes%20ReplicaSets%20and%20Deployments/kubernetes_deployment.png)

                    **Kubernetes ReplicaSet and deployment in a single frame**

![Kubernetes ReplicaSet Deployment](https://github.com/balusena/kubernetes-for-devops/blob/main/05-Kubernetes%20ReplicaSets%20and%20Deployments/kubernetes_replicaset_deployment.png)



