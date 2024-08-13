# kubernetes StatefulSets
Kubernetes StatefulSets are designed for managing stateful applications, ensuring**ordered deployment**,
**scaling**, and **consistent updates**. Unlike Deployments, which are stateless, StatefulSets provide:

- **Stable, unique network identities**: Each pod gets a consistent DNS endpoint and identifier (e.g., `pod-0`, `pod-1`), crucial for maintaining the order and identity required by stateful applications.

- **Persistent storage**: Each pod is associated with its own persistent storage, which remains intact even if the pod is rescheduled or restarted. This ensures data durability and consistency, making StatefulSets ideal for applications like databases and distributed systems.

- **Sequential pod creation and scaling**: Pods are created, updated, or deleted in a strict order, ensuring that the system's overall state is maintained correctly.

- **Ideal for stateful applications**: StatefulSets are perfect for systems that require stable identities and persistent storage, such as Cassandra, Zookeeper, Kafka, and other distributed databases.

- **Controlled rolling updates**: During updates, StatefulSets manage pod disruptions in a controlled manner, ensuring application stability throughout the process.

StatefulSets are essential when applications require stable storage and networking, strict ordering, and 
consistent updates, providing the necessary infrastructure for reliable and predictable operations in 
stateful environments.

![Kubernetes StatefulSets](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/statefulsets.png)

StatefulSet is a controller in Kubernetes that allows users to manage pods the same as the deployments. It
is mainly designed to use for stateful apps. In most cases, users ignore how their pods are scheduled. But
many times, due to some requirements, users make sure that the pods are deployed in order with persistent 
storage volume and a unique stable network identifier across restarts and rescheduled. In these cases, 
StatefulSets can make your job easy.

# StatefulSet Workflow 
Whenever StatefulSet creates a pod, it assigns an ordinal value and a stable network ID to the pod. Users
can also create a VolumeClaimTemplate in the manifest file, and it will further create a persistent volume
for each pod. When the StatefulSet deploys pods, they will get deployed in order from 0 to the last pod. 
The next pod will only be created once the previous pod is ready and in running state. 

![Kubernetes StatefulSets Workflow](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/statefulsets_flowchart.png)

# When to Use StatefulSet?
Kubernetes StatefulSets are particularly useful in the following scenarios:

- **Cassandra Cluster**: When deploying a Cassandra cluster where each node needs to maintain consistent access to its own data, StatefulSets are the ideal choice. They ensure that each node retains its state across restarts or rescheduling.

- **Web Applications with Replica Communication**: If a web application requires communication with its replicas using predefined network identifiers, StatefulSets should be used. They provide stable network identities, which are essential for such communication.

- **Redis with Persistent Storage**: When using Redis, if a pod needs persistent access to a specific volume, even after being redeployed or restarted, StatefulSets ensure that the same volume is consistently attached to the pod. This guarantees data persistence and availability.

StatefulSets are the go-to solution whenever you need stable identities, persistent storage, and ordered deployment for stateful applications in Kubernetes.

# Use Cases of StatefulSet
Let’s discuss some of the use cases of StatefulSet.

![Kubernetes StatefulSets Usecase](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/statefulsets_usecase.png)

## Key Benefits of StatefulSet

- **1.Ordered Scaling and Deployment**:
    - Ensures that dependent containers are created in a specific order during deployments and scaling.
    - This ordered approach is crucial for applications relying on multiple containers, maintaining the correct sequence for dependencies.

- **2.Automated Rolling Updates**:
    - StatefulSets strictly follow automated rolling updates, where dependent applications and microservices are updated in a controlled order.
    - Users have the flexibility to define the update order according to their preferences.

- **3.Persistent Storage Association**:
    - Users can define which pods correspond to specific persistent storage, ensuring that each pod maintains access to the correct volume even after rescheduling or restarts.
    - This capability supports resilient application deployment by maintaining data consistency.

- **4.Unique Network Identifiers**:
    - StatefulSets allow the use of unique network identifiers, ensuring persistent network connectivity for each pod.
    - This eliminates concerns about IP changes during rescheduling, as each pod retains its stable network identity, ensuring seamless communication.

# Limitations of Kubernetes StatefulSet

While StatefulSets offer numerous advantages, they also come with some limitations. Here are the key limitations:

- **1.Persistent Storage Handling**:
    - Scaling up, scaling down, or deleting pods does not affect the underlying persistent storage. To ensure data safety, provisioned volumes will remain within Kubernetes, which could lead to unused storage if not managed properly.

- **2.Storage Provisioning Requirement**:
    - Storage for StatefulSets must always be provisioned using a PersistentVolume Provisioner, either based on a storage class or pre-provisioned. This adds complexity compared to stateless deployments, where storage is not a concern.

- **3.Manual Headless Service Creation**:
    - Users are required to manually create a headless service to provide network identity for StatefulSets. This is an additional step not required for other types of deployments like Deployments or DaemonSets.

- **4.Pod Termination on StatefulSet Deletion**:
    - StatefulSet does not guarantee the termination of existing pods when the StatefulSet is deleted. To avoid orphaned pods, it is recommended to scale the StatefulSet down to zero before deletion.

- **5.Risks with Rolling Updates**:
    - Performing rolling updates with the default pod management policy can lead to issues if a pod is faulty. In such cases, the update process may stall or cause disruptions in the application, requiring careful management and possibly manual intervention.

**Note:**
In Kubernetes, we deployed MongoDB with a single replica using Deployments. However, to achieve high 
availability and enhance overall performance and reliability, we should deploy multiple replicas, as 
MongoDB is a stateful application. Regular Deployment resources cannot be used to deploy multiple replicas
for such stateful applications.

### Stateful Applications vs Stateless Applications

### Stateful Applications
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Aspect**                | **Details**                                                                                                                       |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Data Persistence**      | Requires persistent storage. Data is saved and remains accessible even after pod rescheduling or restarts.                        |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Network Identity**      | Each pod has a unique and stable network identifier, ensuring consistent communication with other components.                     |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Deployment and Scaling**| Pods are created, updated, and deleted in a specific order, crucial for applications needing strict sequence (e.g., databases).   |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Storage**               | Each pod is associated with its own persistent volume, ensuring data continuity across the pod's lifecycle.                       |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Pod Management**        | Pods have a stable, unique identity and their lifecycle, including persistent storage, is managed.                                |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Headless Service**      | Requires a headless service to provide stable network identities and enable direct communication between pods.                    |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Rolling Updates**       | Updates are controlled and managed in a specific order to maintain stability, with pods updated sequentially.                     |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Use Cases**             | Ideal for stateful applications like databases (e.g., MongoDB, Cassandra) where data and state must be preserved.                 |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
```
**Usecase:**
Let's say we have a simple Spring Boot application that handles authentication. When a client sends a login
request, we set an authenticated flag to true in memory. If the same client sends a subsequent request, we
simply read this flag to determine if the user is already logged in or not. Essentially, we are storing the
state of the current request, and the next request depends on the state of the previous one. This type of
application is called a stateful application because we are storing the state, in this case, the 
authentication state.

![Kubernetes Stateful Application](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/stateful_app.png)


### Stateless Applications
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Aspect**                | **Details**                                                                                                                       |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Data Persistence**      | Does not require persistent storage. Data is temporary and lost when the pod is terminated.                                       |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Network Identity**      | Pods have dynamic network identities; they can be replaced or rescheduled without affecting functionality.                        |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Deployment and Scaling**| Pods can be created, updated, or deleted in any order, making scaling flexible and straightforward.                               |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Storage**               | May use shared storage or none; data persistence is not required after pod termination.                                           |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Pod Management**        | Pods are managed independently; any pod can be replaced at any time without concern for state or identity.                        |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Headless Service**      | No need for a headless service; communication is managed through standard Kubernetes services.                                    |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Rolling Updates**       | Updates can be performed flexibly; pods are updated independently and can be replaced in any order.                               |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Use Cases**             | Suitable for stateless applications like web servers or batch processing jobs where each instance operates independently.         |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
```
**Usecase:**
Let's say we have multiple instances of the same application with a load balancer in front.

- **Scenario**: The first request goes to Instance1, where we set the authenticated flagship group. If the second request goes to Instance2, it might not give the correct result because the flag was not set on Instance2.

- **Best Practice**: To address this issue, it is recommended to move the state to a database. Here’s how it works:
  - When a login request is sent for the first time, a token is generated and saved in the database.
  - The Spring Boot application expects this token for subsequent requests and validates it against the database.

By storing the state in the database and not in the Spring Boot application instances, we ensure that no matter how many instances we have, we will get consistent results. This is because the application itself is stateless, while the database is stateful as it stores the state.

- **Conclusion**: This approach ensures that the application remains stateless, and the database manages the state, providing a consistent experience across all instances.

![Kubernetes Stateful Stateless](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/stateful_stateless.png)

### Problems Encountered When Deploying Stateful Applications with Multiple Replicas.

#### **Problem Statement-1:**
Previously we deployed a single replica of mongodb using deployment and we used PV to store its data now to
achieve high availability we need to increase the number of replicas, when we use deployments all replicas
use same PV,but in a distributed database if we use the same PV for all replicas all pods write to the same 
database which leads to data inconsistency,so the solution to this problem is to have a way to use separate
PV for each replica this can be achieved using statefulsets, in statefulsets we define the volumeclaim 
template based on this template separate PVC are used for each pod which creates separate PV.

Previously,we deployed a single replica of MongoDB using a Deployment and used a Persistent Volume (PV) to
store its data. To achieve high availability, we need to increase the number of replicas. However, when 
using Deployments, all replicas share the same PV. In a distributed database setup, if all replicas use 
the same PV, all pods write to the same database, which can lead to data inconsistency.

To address this issue, we need a way to use separate PVs for each replica. This can be achieved using 
StatefulSets. In StatefulSets, we define a VolumeClaimTemplate. Based on this template, separate Persistent
Volume Claims (PVCs) are created for each pod, which in turn creates separate PVs.

By using StatefulSets, each MongoDB replica gets its own PVC and PV, ensuring data consistency and high 
availability.

![Kubernetes Problem Statement 1](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/ps-1.png)

#### **Problem Statement-2:**
In a typical Master-Slave architecture, there are two types of nodes: one Master node and multiple Slave 
nodes. The Master node handles both reads and writes, while the Slaves handle only reads. For efficient 
and reliable replication, the Master should be up and running first. Next, Slave1 should come up, and the
data from the Master should be copied to Slave1. After that, Slave2 should come up, and instead of 
retrieving the data from the Master again, the data should be copied from Slave1. This approach reduces 
the load on the Master. After this initial cloning process, all Slaves will continuously sync data from 
the Master node.

To achieve this initial cloning, the pods should start sequentially, ensuring that each pod copies data 
from the previous replica. However, if we use Deployment resources to deploy this type of application,
all the pods are created in parallel. In contrast, if we deploy the same application using StatefulSets,
the pods are created one by one. For example, if we create three replicas, Pod1 will be created first, 
followed by Pod2, and then Pod3. If the first pod fails for any reason, the second pod will not be created.
Additionally, when we delete the StatefulSet, the last pod is deleted first, i.e., Pod3 in our case.

![Kubernetes Problem Statement 2](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/ps-2.png)

#### **Problem Statement-3:**
As we discussed in Problem Statement 2, in a Master-Slave architecture, all nodes in the cluster must 
communicate with each other for data replication. To facilitate this, a sticky identity is required so 
that each pod can be consistently found within the cluster. A sticky identity means that each pod should 
be accessible via a DNS name that does not change even if the pod restarts. If the DNS name changes when 
the pod restarts, other replicas may not be able to find it. To achieve this, there must be a way to assign
a constant name to all pods, ensuring that even if a pod restarts, it retains the same name. This is why 
it is called a sticky identity—the name "sticks" to the pod.

However, if we use regular Deployments, the pods receive a random name every time they are created, and if
a pod is restarted for any reason, it gets a new name. In contrast, when we deploy the same application 
using a StatefulSet, all pods receive a name that can be easily predicted. The naming convention of the 
pods follows the pattern `statefulsetname-ordinalindex` (e.g., `mongo-0`, `mongo-1`, `mongo-2`, etc.). 
Even if a pod restarts, it retains the same name.

Not only does StatefulSet provide sticky identity, but it also offers sticky storage. Each pod in a 
StatefulSet gets its own Persistent Volume (PV), and when a pod restarts, it reattaches to the same PV. 
StatefulSet manages this process automatically, without any additional configuration.

Furthermore, it’s not just about the pods having a fixed name and storage. As we discussed in the Services
topic, we need a service to communicate with these pods. Typically, services act like load balancers, 
meaning if we call the service, the request goes to a random pod. However, in this scenario, we need to 
communicate with a specific pod—writes should go to the Master node, Slave1 should get data from the 
Master, and Slave2 should get data from Slave1.

To achieve this, Kubernetes provides a special service called a "Headless Service." When we specify 
`clusterIP: None`, the service is considered a "Headless Service." Through this service, each pod gets
its own DNS entry, 

e.g., `mongo-0.mongo.default.svc.cluster.local:27017`.

- mongo-0 —> Pod name
- mongo   —> Service name
- default —> Namespace

Now, when we try to access this DNS, the request is directed specifically to the `mongo-0` pod. Headless
Services are very useful when we do not want to perform load balancing and need to connect directly to a
single pod.

![Kubernetes Problem Statement 3](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/ps-3.png)

# Deployment vs StatefulSets

![Kubernetes Deployment vs StatefulSets](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/deployment_statefulset.png)

### 1.To get the api-resources of statefulset.
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep statefulset
statefulsets            sts          apps/v1            true         StatefulSet
```
**Note:** Before to create statefulset.yaml we need to create sc.yaml.

### 2.create sc.yaml:
```
ubuntu@balasenapathi:~$ nano sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: demo-storage
provisioner: k8s.io/minikube-hostpath
volumeBindingMode: Immediate
reclaimPolicy: Delete
```
### 3.Apply the changes into the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f sc.yaml
secret/mongodb-secret created
```                                                           
### 4.Now create a statefulset.yaml file.
```
ubuntu@balasenapathi:~$ nano statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:4.0.8
          args: ["--dbpath", "/data/db"]
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              value: "admin"
            - name: MONGO_INITDB_ROOT_PASSWORD
              value: "password"
          command:
            - mongod
            - "--bind_ip_all"
            - "--replSet"
            - rs0
          volumeMounts:
            - name: mongo-volume
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: mongo-volume
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: demo-storage
        resources:
          requests:
            storage: 1Gi
```
**Note:** This configuration is similar to the deployment with a few changes.In this setup, the number 
of replicas is set to 3. Instead of specifying a PersistentVolumeClaim (PVC) directly, we are using 
`volumeClaimTemplates`. This means we are providing a template for the PVC that will be created for each 
pod. The volume will be named `mongo-volume` and will be mounted in the container at the path `/data/db`.
The access mode is set to `ReadWriteOnce`, and the storage class we created is `demo-storage`, which will
be used to create the PersistentVolume (PV). We are requesting 1GB of storage. Aside from these changes, 
everything else remains the same as in the deployment.

### 5.Now apply the changes into the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo created
```
**1.Ordered Pods:**

### 6.To list the all the pods in the local-cluster:
```
ubuntu@balasenapathi:~$ kubectl get pod -w
NAME      READY   STATUS              RESTARTS   AGE
mongo-0   0/1     ContainerCreating   0          8s
mongo-0   0/1     Running             0          58s
mongo-0   0/1     Running             0          64s
mongo-0   1/1     Running             0          65s
mongo-1   0/1     Pending             0          1s
mongo-1   0/1     Pending             0          1s
mongo-1   0/1     Pending             0          2s
mongo-1   0/1     ContainerCreating   0          3s
mongo-1   0/1     Running             0          15s
mongo-1   0/1     Running             0          23s
mongo-1   1/1     Running             0          24s
mongo-2   0/1     Pending             0          0s
mongo-2   0/1     Pending             0          0s
mongo-2   0/1     Pending             0          1s
mongo-2   0/1     ContainerCreating   0          1s
mongo-2   0/1     Running             0          4s
mongo-2   0/1     Running             0          12s
mongo-2   1/1     Running             0          13s
```
**Note:** The `-w` flag is used to continuously watch the status changes during pod creation. Initially,
the `mongo-0` pod enters the "ContainerCreating" state, followed by it transitioning to "Running." Next,
the `mongo-1` pod is created, then moves to "Running." Finally, the `mongo-2` pod is created and also 
transitions to "Running."

### 7.Now try to scale up the StatefulSet and observe the behavior of the pods in the local cluster:

**Note:** To scale up, you can either update the `replicas` field to `7` in the `statefulset.yaml` file 
or use the `kubectl` command directly:

```
ubuntu@balasenapathi:~$ kubectl scale sts mongo --replicas=7
statefulset.apps/mongo scaled
```
### 8.To list all the pods in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME      READY   STATUS              RESTARTS   AGE
mongo-0   1/1     Running             0          16m
mongo-1   1/1     Running             0          15m
mongo-2   1/1     Running             0          15m
mongo-3   1/1     Running             0          16s
mongo-4   0/1     ContainerCreating   0          3s
mongo-4   0/1     Running             0          4s
mongo-4   0/1     Running             0          13s
mongo-4   1/1     Running             0          14s
mongo-5   0/1     Pending             0          0s
mongo-5   0/1     Pending             0          0s
mongo-5   0/1     Pending             0          1s
mongo-5   0/1     ContainerCreating   0          1s
mongo-5   0/1     Running             0          4s
mongo-5   0/1     Running             0          12s
mongo-5   1/1     Running             0          12s
mongo-6   0/1     Pending             0          0s
mongo-6   0/1     Pending             0          0s
mongo-6   0/1     Pending             0          1s
mongo-6   0/1     ContainerCreating   0          1s
mongo-6   0/1     Running             0          3s
mongo-6   0/1     Running             0          12s
mongo-6   1/1     Running             0          12s
```
**Note:** Initially, we see that three pods are running. Then, the 4th pod is created, followed by the 
5th, 6th, and finally the 7th pod, all running in the local cluster. As discussed, the pod naming 
convention in a StatefulSet is `"mongo-"` followed by an ordinal index value, which increments for each
pod. Thus, the pod names are `"mongo-0", "mongo-1", "mongo-2", "mongo-3", "mongo-4", "mongo-5", "mongo-6"` respectively.

### 9.Now lets try to sacledown and see what happens to the pods in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl scale sts mongo --replicas=3
statefulset.apps/mongo scaled
```
### 10.To list all the pods in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME      READY   STATUS        RESTARTS   AGE
mongo-0   1/1     Running       0          26m
mongo-1   1/1     Running       0          25m
mongo-2   1/1     Running       0          24m
mongo-3   1/1     Running       0          9m54s
mongo-4   1/1     Running       0          9m41s
mongo-5   1/1     Running       0          9m27s
mongo-6   0/1     Terminating   0          9m15s
mongo-6   0/1     Terminating   0          9m15s
mongo-6   0/1     Terminating   0          9m15s
mongo-6   0/1     Terminating   0          9m15s
mongo-5   1/1     Terminating   0          9m27s
mongo-5   0/1     Terminating   0          9m28s
mongo-5   0/1     Terminating   0          9m29s
mongo-5   0/1     Terminating   0          9m29s
mongo-5   0/1     Terminating   0          9m29s
mongo-4   1/1     Terminating   0          9m43s
mongo-4   0/1     Terminating   0          9m44s
mongo-4   0/1     Terminating   0          9m44s
mongo-4   0/1     Terminating   0          9m44s
mongo-4   0/1     Terminating   0          9m44s
mongo-3   1/1     Terminating   0          9m57s
mongo-3   0/1     Terminating   0          9m58s
mongo-3   0/1     Terminating   0          9m58s
mongo-3   0/1     Terminating   0          9m58s
mongo-3   0/1     Terminating   0          9m58s
```
**Note:** As we observe the pods being deleted, the first pod to be deleted is the one with the highest 
ordinal index value. In this case, `mongo-6` is deleted first, followed by `mongo-5`, then `mongo-4`,and
finally `mongo-3`. Since we scaled down to `replicas=3`, the remaining three pods (`mongo-0`, `mongo-1`, 
and `mongo-2`) continue to run.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          31m
mongo-1   1/1     Running   0          30m
mongo-2   1/1     Running   0          29m
```
**2.Sticky Identity:**

### 1.Now try to delete the pod and see if the same name is given to the pod in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl delete pod mongo-0
pod "mongo-0" deleted
```
### 2.To get the list of pods in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   0/1     Running   0          3s
mongo-1   1/1     Running   0          32m
mongo-2   1/1     Running   0          32m
```
### 3.To get the list of pods in the local-cluster monitoring with watch (-w) flag.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   0/1     Running   0          8s
mongo-1   1/1     Running   0          32m
mongo-2   1/1     Running   0          32m
mongo-0   0/1     Running   0          11s
mongo-0   1/1     Running   0          12s
```
### 4.To get the list of pods in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          44s
mongo-1   1/1     Running   0          33m
mongo-2   1/1     Running   0          32m
```
**Note:** We can observe that the pod with the same name, `mongo-0`, was recreated 44 seconds ago in the 
local cluster with the same name. This confirms that each pod is assigned a sticky identity.

**3.Seperate PersistentVolumeClaims:**

### 1.Now to confirm each pod is using separate persistentvolumeclaims in the local-cluster:
```
ubuntu@balasenapathi:~$ kubectl get pvc
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mongo-volume-mongo-0   Bound    pvc-98e5c7b3-6f60-451a-af96-52e29229d473   1Gi        RWO            demo-storage   41m
mongo-volume-mongo-1   Bound    pvc-b5d11796-89ac-4eaf-9765-efacc0f8c587   1Gi        RWO            demo-storage   40m
mongo-volume-mongo-2   Bound    pvc-518855f0-cdf3-47a7-a34d-7c56a1f2968e   1Gi        RWO            demo-storage   39m
mongo-volume-mongo-3   Bound    pvc-724cbed1-05ce-4628-8dcf-3130af13f141   1Gi        RWO            demo-storage   24m
mongo-volume-mongo-4   Bound    pvc-808b9ba4-3d01-4eb3-865a-6ec259cb9011   1Gi        RWO            demo-storage   24m
mongo-volume-mongo-5   Bound    pvc-73e4e697-72ae-4155-bbcf-ba0aa670b6e3   1Gi        RWO            demo-storage   24m
mongo-volume-mongo-6   Bound    pvc-02eb6458-5482-4f08-9540-4ce2b9829d84   1Gi        RWO            demo-storage   24m
```
**Note:** There are 7 PersistentVolumeClaims (PVCs) created by the `demo-storage` StorageClass, and the 
status of all these PVCs is "Bound" to separate PersistentVolumes (PVs). You might wonder why there are 
7 PVCs in the local cluster when only 3 pods are running. This is because when we increased the number 
of replicas to 7, 7 PVCs were created. However, PVCs are not deleted when pods are removed. Additionally,
when the `mongo-0` pod is restarted, the same `mongo-0` PVC is used by that pod. This can be confirmed by
describing the pod.

### 2.To describe the mongo-0 pod and get full information.
```
ubuntu@balasenapathi:~$ kubectl describe pod mongo-0 | grep volume
/data/db from mongo-volume (rw)
mongo-volume:
ClaimName:  mongo-volume-mongo-0
Type:      ConfigMap (a volume populated by a ConfigMap)
Type:                    Projected (a volume that contains injected data from multiple sources)
```
**Note:** Here, we can see that the ClaimName: mongo-volume-mongo-0 is used to create the deleted mongo-0
pod again. This demonstrates the concept of sticky storage, where the same PVC is used when the pod is 
recreated.

**4.Headless Service:**

**Note:** As discussed, to communicate with a specific pod, we need a "headless" service.

### 1.Now, let's create a headless service in the local cluster.

```
ubuntu@balasenapathi:~$ nano headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  ports:
    - name: mongo
      port: 27017
      targetPort: 27017
  clusterIP: None
  selector:
    app: mongo
```
**Note:** Our mongo pods are running on ports 27017 and the label name is app: mongo

### 2.Apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f headless-service.yaml
service/mongo created
```
### 3.To verify that the headless services are created in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP     110m
mongo        ClusterIP   None         <none>        27017/TCP   100s
```
**Note:** In a headless service, the `clusterIP` is set to `None`. Note that headless services cannot be
accessed directly from outside the cluster.

### 4.Now let us create the MongoDB replica sets in the local cluster.

With 3 pods, we can specify one pod as the Master Node and the others as Slave Nodes.Note that StatefulSets
manage the pods but do not handle replication for us. We need to configure the replica sets manually by 
accessing the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          35m
mongo-1   1/1     Running   0          67m
mongo-2   1/1     Running   0          67m
```
### 5.To get into the mongo pod in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("f51c393c-d744-4888-aecf-3427a479cb9e") }
MongoDB server version: 4.0.8
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
http://docs.mongodb.org/
Questions? Try the support group
http://groups.google.com/group/mongodb-user
Server has startup warnings:
2023-09-25T18:43:24.440+0000 I STORAGE  [initandlisten]
2023-09-25T18:43:24.441+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger
storage engine
2023-09-25T18:43:24.441+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten]
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not
recommended.
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten]
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
>
```
**Note:** Now we are in the mongo-0 pod in local-cluster.

### 6.Initiating the MongoDB Replica Set

According to the official MongoDB documentation, use the following command to initiate the replica set:
```
rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "mongodb0.example.net:27017" },
      { _id: 1, host: "mongodb1.example.net:27017" },
      { _id: 2, host: "mongodb2.example.net:27017" }
   ]
})
```
**5.Our Custom replicaset with DNS:**
```
rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
      { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
      { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
   ]
})
```
**Note:** The name of the replica set is `"rs0"`, as specified in our StatefulSet manifest file. 
The DNS names for our 3 pods are:
- `mongo-0`: `mongodb0.example.net:27017`
- `mongo-1`: `mongodb1.example.net:27017`
- `mongo-2`: `mongodb2.example.net:27017`

### 7.Creating MongoDB Replica Sets in the Local Cluster

**Step 1:** Execute the Command to Initiate the Replica Set
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c6badafa-2063-4000-b789-18c200f371ed") }
MongoDB server version: 4.0.8
Server has startup warnings: 
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] 
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger 
storage engine
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-09-25T21:46:22.123+0000 I CONTROL  [initandlisten] 
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not 
recommended.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] 
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

> rs.initiate( {
...    _id : "rs0",
...    members: [
...       { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
...       { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
...       { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
...    ]
... })
{
	"ok" : 1,
	"operationTime" : Timestamp(1695678772, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695678772, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
rs0:SECONDARY>
```
**Note:** Here, "rs0" is the replica set name specified in the StatefulSet. The hosts are:

- mongo-0.mongo.default.svc.cluster.local:27017
- mongo-1.mongo.default.svc.cluster.local:27017
- mongo-2.mongo.default.svc.cluster.local:27017

Where "mongo-0", "mongo-1", and "mongo-2" are pod names, "mongo" is the service name, and "default" is 
the namespace. We can see that the replica set initiation returned "ok" : 1, indicating success.

**Step 2:** Reconnect to the MongoDB Shell and Verify Replica Set Status

- Exit the current session and reconnect to the mongo-0 pod.
```
> rs.initiate( {
...    _id : "rs0",
...    members: [
...       { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
...       { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
...       { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
...    ]
... })
{
"ok" : 1,
"operationTime" : Timestamp(1695678772, 1),
"$clusterTime" : {
"clusterTime" : Timestamp(1695678772, 1),
"signature" : {
"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
"keyId" : NumberLong(0)
}
}
}
rs0:SECONDARY> exit
bye
```
- Reconnect to the MongoDB shell.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("19288f4c-402c-4d8c-9248-a2c8d85c23d5") }
MongoDB server version: 4.0.8
Server has startup warnings:
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten]
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger
storage engine
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-09-25T21:46:22.123+0000 I CONTROL  [initandlisten]
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not
recommended.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten]
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

rs0:PRIMARY>
```
**Note:** After the initial configuration, the node is now a PRIMARY node.

**Step 3:** Verify Replica Set Status.

Check the status of the replica set to verify the current state of each node.
```
> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2023-09-25T22:05:37.139Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		}
	},
	"lastStableCheckpointTimestamp" : Timestamp(1695679505, 1),
	"members" : [
		{
			"_id" : 0,
			"name" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 1157,
			"optime" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-09-25T22:05:35Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"electionTime" : Timestamp(1695678784, 1),
			"electionDate" : ISODate("2023-09-25T21:53:04Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "mongo-1.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 764,
			"optime" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-09-25T22:05:35Z"),
			"optimeDurableDate" : ISODate("2023-09-25T22:05:35Z"),
			"lastHeartbeat" : ISODate("2023-09-25T22:05:36.658Z"),
			"lastHeartbeatRecv" : ISODate("2023-09-25T22:05:36.687Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceHost" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "mongo-2.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 764,
			"optime" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-09-25T22:05:35Z"),
			"optimeDurableDate" : ISODate("2023-09-25T22:05:35Z"),
			"lastHeartbeat" : ISODate("2023-09-25T22:05:36.660Z"),
			"lastHeartbeatRecv" : ISODate("2023-09-25T22:05:36.692Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceHost" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1695679535, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695679535, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```
**Note:** The rs.status() output shows.

- "stateStr": "PRIMARY" for mongo-0
- "stateStr": "SECONDARY" for mongo-1
- "stateStr": "SECONDARY" for mongo-2

All secondary nodes are synchronized with the primary node after the initial clone.

### 8.Creating and Verifying Data Replication in MongoDB Replica Sets

**Step 1:** Create a Simple Database and Insert Data into the Primary Node

First, log into the primary node and create a simple database with a collection. Insert a document into 
the collection and verify the insertion.

1. **Connect to the MongoDB shell on the primary node:**

    ```bash
    ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
    ```

2. **Switch to the `test` database:**

    ```bash
    rs0:PRIMARY> use test
    ```

3. **Insert a document into the `todos` collection:**

    ```bash
    rs0:PRIMARY> db.todos.insert({"title":"Test"})
    WriteResult({ "nInserted" : 1 })
    ```

4. **Verify the inserted document:**

    ```bash
    rs0:PRIMARY> db.todos.find()
    { "_id" : ObjectId("651206cbf0530d8dc0720209"), "title" : "Test" }
    ```

5. **Exit the MongoDB shell:**

    ```bash
    rs0:PRIMARY> exit
    bye
    ```

**Step 2:** Verify Data Replication in the First Secondary Node

Log into the second node of the pod (mongo-1) and check if the data is replicated. By default, secondary 
nodes do not allow read operations, so you need to enable read operations.

1. **Connect to the MongoDB shell on the secondary node:**

    ```bash
    ubuntu@balasenapathi:~$ kubectl exec -it mongo-1 -- mongo
    ```

2. **Attempt to query the `todos` collection:**

    ```bash
    rs0:SECONDARY> db.todos.find()
    Error: error: {
        "operationTime" : Timestamp(1695680385, 1),
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotMasterNoSlaveOk",
        "$clusterTime" : {
            "clusterTime" : Timestamp(1695680385, 1),
            "signature" : {
                "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                "keyId" : NumberLong(0)
            }
        }
    }
    ```

3. **Enable read operations on the secondary node:**

    ```bash
    rs0:SECONDARY> rs.slaveOk()
    ```

4. **Retry querying the `todos` collection:**

    ```bash
    rs0:SECONDARY> db.todos.find()
    { "_id" : ObjectId("651206cbf0530d8dc0720209"), "title" : "Test" }
    ```

5. **Exit the MongoDB shell:**

    ```bash
    rs0:SECONDARY> exit
    bye
    ```

**Step 3:** Verify Data Replication in the Second Secondary Node

Log into the third node of the pod (mongo-2) and check if the data is replicated.

1. **Connect to the MongoDB shell on the second secondary node:**

    ```bash
    ubuntu@balasenapathi:~$ kubectl exec -it mongo-2 -- mongo
    ```

2. **Attempt to query the `todos` collection:**

    ```bash
    rs0:SECONDARY> db.todos.find()
    Error: error: {
        "operationTime" : Timestamp(1695681005, 1),
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotMasterNoSlaveOk",
        "$clusterTime" : {
            "clusterTime" : Timestamp(1695681005, 1),
            "signature" : {
                "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                "keyId" : NumberLong(0)
            }
        }
    }
    ```

3. **Enable read operations on the secondary node:**

    ```bash
    rs0:SECONDARY> rs.slaveOk()
    ```

4. **Retry querying the `todos` collection:**

    ```bash
    rs0:SECONDARY> db.todos.find()
    { "_id" : ObjectId("651206cbf0530d8dc0720209"), "title" : "Test" }
    ```

5. **Exit the MongoDB shell:**

    ```bash
    rs0:SECONDARY> exit
    bye
    ```

## Summary

The steps above demonstrate how to create and insert data into a MongoDB replica set and verify that data
replication works correctly across primary and secondary nodes. The data inserted into the primary node is
successfully replicated to both secondary nodes, confirming that replication is functioning as expected.

Data replication between nodes is facilitated by the MongoDB replica set and the headless service, which
enables pods in the StatefulSet to communicate and synchronize data.





















