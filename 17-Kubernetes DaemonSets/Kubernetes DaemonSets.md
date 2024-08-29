# Kubernetes DaemonSets:
A DaemonSet in Kubernetes ensures that a particular pod runs on all (or a subset of) nodes in a cluster.When a DaemonSet
is created,Kubernetes schedules the specified pod on every node,providing a uniform deployment of essential system services
or monitoring tools across the cluster.

**Key Points:**

- **Automatic Scheduling:** DaemonSets automatically add pods to new nodes when they join the cluster and remove them from
  nodes that are terminated.
- **Use Cases:** Common use cases include logging agents, monitoring daemons, and network proxies that need to run on every
  node.
- **Node Selectors:** DaemonSets can be configured with node selectors, taints, and tolerations to control where pods are
  scheduled.
- **Updates:** Updating a DaemonSet updates pods one node at a time to ensure continuous operation.

**Note:** DaemonSets are crucial for maintaining system-wide services and ensuring consistent operation across all nodes.

### DaemonSets for Node-Specific Tasks

We've learned that Deployments and ReplicaSets ensure a specified number of pods are running, potentially distributed 
across different nodes based on affinity. However, in scenarios where tasks must be executed on every node in a cluster,
such as collecting logs or metrics, Deployments or ReplicaSets alone do not guarantee pod placement on every node, 
especially as nodes are dynamically added.

**DaemonSets** address this need by ensuring that a pod runs on every node within the cluster. This is particularly useful
for node-specific tasks where uniform distribution across all nodes is crucial. For example, DaemonSets are ideal for:

- **Collecting Logs:** Deploying log collectors to aggregate logs from every node.
- **Monitoring Metrics:** Running monitoring agents that gather metrics from all nodes.
- **Node Maintenance Tasks:** Executing maintenance or configuration tasks across the cluster.

DaemonSets ensure that as new nodes are added, the specified pod is automatically scheduled on these new nodes, maintaining
the required node-specific functionality across the entire cluster.

![Kubernetes DaemonSets Intro](https://github.com/balusena/kubernetes-for-devops/blob/main/17-Kubernetes%20DaemonSets/daemonsets_intro.png)

### Monitoring Nodes with DaemonSets

In a Kubernetes cluster with multiple nodes, it's essential to monitor node health, such as memory usage and CPU load. 
To achieve this, we need to deploy an agent on each node to collect and store metrics. 

**DaemonSets** are the ideal solution for this requirement. Unlike Deployments or ReplicaSets, which might not automatically
schedule pods on newly added nodes, DaemonSets ensure that a pod runs on every node in the cluster. 

Here's how DaemonSets address the challenge:
- **Automatic Scheduling:** As new nodes are added to the cluster, a new pod is automatically scheduled on the new node.
  Conversely, if a node is removed, the corresponding pod is garbage collected.
- **Consistency:** DaemonSets ensure there is always one pod per node, handling node-specific tasks like log collection 
  or metric gathering consistently across the entire cluster.
- **Difference from Deployments:** While Deployments are used for stateless services and can scale replicas up and down 
  with rolling updates, DaemonSets ensure that a pod runs on all or selected nodes, maintaining uniform coverage for cluster-level tasks.

Using DaemonSets for node monitoring guarantees that the necessary agents are deployed on every node, adapting dynamically
as nodes are added or removed from the cluster.
 
![Kubernetes DaemonSets](https://github.com/balusena/kubernetes-for-devops/blob/main/17-Kubernetes%20DaemonSets/daemonsets.png)

