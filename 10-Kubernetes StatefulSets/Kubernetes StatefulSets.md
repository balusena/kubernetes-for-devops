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

#### **Problem Statement-2:**
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
its own DNS entry, e.g., `mongo-0.mongo.default.svc.cluster.local:27017`.

- mongo-0 —> Pod name
- mongo   —> Service name
- default —> Namespace

Now, when we try to access this DNS, the request is directed specifically to the `mongo-0` pod. Headless
Services are very useful when we do not want to perform load balancing and need to connect directly to a
single pod.














