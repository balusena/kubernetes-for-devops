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
