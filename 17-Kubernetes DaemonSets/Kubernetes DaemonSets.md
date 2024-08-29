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

DaemonSets are crucial for maintaining system-wide services and ensuring consistent operation across all nodes.


