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

**StatefulSet Workflow:**
Whenever StatefulSet creates a pod, it assigns an ordinal value and a stable network ID to the pod. Users
can also create a VolumeClaimTemplate in the manifest file, and it will further create a persistent volume
for each pod. When the StatefulSet deploys pods, they will get deployed in order from 0 to the last pod. 
The next pod will only be created once the previous pod is ready and in running state. 

![Kubernetes StatefulSets Workflow](https://github.com/balusena/kubernetes-for-devops/blob/main/10-Kubernetes%20StatefulSets/statefulset_flowchart.png)
