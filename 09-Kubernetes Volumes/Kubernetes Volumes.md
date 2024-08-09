# kubernetes Volumes
Containers are considered 'ephemeral,' meaning that when a container is deleted, all data associated with
it will be lost. However, we can persist the data even if the container is deleted by using Docker volumes,
and the same concept applies to Kubernetes.

![Container Pod](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/container_pod.png)

When a pod is deleted, all the data associated with it is also deleted. Now, we will look at how to persist
the data even if the pod is deleted, using Kubernetes Volumes.

**Problem Statement-1**
When we create pods using a Deployment or ReplicaSet, Kubernetes ensures that the specified number of 
replicas is maintained. For example, if we create three (3) replicas of our application and write some 
data to those pods, Kubernetes will automatically create a new pod if any replica or pod goes down, to 
maintain the desired state of three (3) replicas. However, this new pod will start with a clean state, 
and any data previously written to the pod will be lost. This leads to data loss when a pod is deleted 
or restarted.

![Problem Statement 1](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/ps-1.png)






