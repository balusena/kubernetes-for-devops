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

![Kubernetes DaemonSets](https://github.com/balusena/kubernetes-for-devops/blob/main/17-Kubernetes%20DaemonSets/daemonsets_intro.png)




