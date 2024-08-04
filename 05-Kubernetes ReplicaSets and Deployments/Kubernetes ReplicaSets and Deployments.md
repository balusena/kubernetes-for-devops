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
## Self-Healing

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
