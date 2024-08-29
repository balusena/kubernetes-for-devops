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

### Common Use Cases for DaemonSets

DaemonSets are commonly used in Kubernetes to ensure that certain types of agents are deployed across all nodes in the 
cluster. This guarantees that essential tasks are performed consistently on every node. Here are some common use cases:

1. **Logging Agents:**
   - **Example:** Fluentd, Logstash
   - **Purpose:** Collect logs from all nodes and centralize them in a single location for analysis.

2. **Monitoring Agents:**
   - **Example:** Prometheus, Datadog
   - **Purpose:** Collect metrics from all nodes and centralize them for monitoring and analysis.

3. **Network Agents:**
   - **Example:** Cilium, Weave Net
   - **Purpose:** Enforce network policies and manage networking across all nodes.

By deploying these agents as DaemonSets, you ensure that each node in the Kubernetes cluster has the necessary tools for
logging, monitoring, and networking, adapting dynamically as nodes are added or removed.

![Kubernetes DaemonSets Usecases](https://github.com/balusena/kubernetes-for-devops/blob/main/17-Kubernetes%20DaemonSets/daemonsets_usecases.png)

### Practical Example: Running Prometheus Node Exporter with DaemonSet

To understand DaemonSets practically, let's run the Prometheus Node Exporter on every node in our Minikube cluster. The 
Prometheus Node Exporter is a widely used daemon for collecting hardware and operating system metrics from Linux nodes 
in a Kubernetes cluster.

### 1.Now create a simple daemonset.
```
ubuntu@balasenapathi:~$ nano daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      # nodeSelector:
      #   kubernetes.io/os: linux
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        args:
          - --path.procfs=/host/proc
          - --path.sysfs=/host/sys
        ports:
        - name: metrics
          containerPort: 9100
        volumeMounts:
        - name: procfs
          mountPath: /host/proc
          readOnly: true
        - name: sysfs
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: procfs
        hostPath:
          path: /proc
      - name: sysfs
        hostPath:
          path: /sys
```
**Note:** apiVersion: apps/v1 this is the apiVersion for daemonset we can get this with:

### 2.To get the api-version of daemonset in our cluster.
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep daemonset 
daemonsets                         ds              apps/v1                                true         DaemonSet
```
**Note:** We can see, this is `apiVersion: apps/v1`, the short name for DaemonSet is `ds`, and the kind is `DaemonSet`.

Under the `spec` section, we specify `matchLabels` with `app: node-exporter`, which the DaemonSet uses to determine if 
the pod is already running on a node. The pod labels provided here are `app: node-exporter`, which the DaemonSet applies
to the pod.

Under the `template` spec, you can use `nodeSelector` or `nodeAffinity` to instruct the DaemonSet to run the pods on 
specific nodes. This is useful if you want to monitor only certain nodes.

In the `containers` section, similar to deployments, we define the container details. We use the image `prom/node-exporter:latest`
for the Prometheus Node Exporter, and in the `args` section, we specify the arguments `"path.procfs=/host/proc"` and 
`"path.sysfs=/host/sys"`, with the container port set to `9100`.

In the `volumeMounts` section, we mount two volumes named `procfs` and `sysfs`. These are virtual file systems in Linux 
that provide system information dynamically generated by the kernel. Both are read-only. To gather system-level metrics,
we need to mount these folders, which are present on every node, using the `hostPath` volume specification.

### 2.Apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f daemonset.yaml
daemonset.apps/node-exporter created
```
### 3.We can verify that using.
```
ubuntu@balasenapathi:~$ kubectl get ds
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
node-exporter   1         1         1       1            1           <none>          22m
```
**Note:** Our expectation is that this daemonset should create a pod.

### 4.Now list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
node-exporter-bcm7c            1/1     Running   0             24m  ====> one pod is running
```
**Note:** We can see here ====> one pod is running this is because we have only one node in the kubernetes cluster.

### 5.We can verify that using.
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   5d20h   v1.27.3
```
**Note:** We have one node ===> minikube in our cluster.

### 6.Now try to add another node to our cluster.
```
ubuntu@balasenapathi:~$ minikube node add
😄  Adding node m02 to cluster minikube
❗  Cluster was created without any CNI, adding a node to it might cause broken networking.
👍  Starting worker node minikube-m02 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
🔎  Verifying Kubernetes components...
🏄  Successfully added m02 to minikube!
```
### 7.To get the list of nodes in our cluster.
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
minikube       Ready    control-plane   5d20h   v1.27.3
minikube-m02   Ready    <none>          113s    v1.27.3
```
**Note:** We have two nodes minikube, minikube-m02 in our cluster.

- According to the definition of a DaemonSet, a new pod should automatically be created on any newly added node.

### 8.We can verify that by using.
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS        AGE     IP            NODE           NOMINATED NODE   READINESS GATES
node-exporter-9wrcw            1/1     Running   4 (3m54s ago)   4m47s   10.244.1.2    minikube-m02   <none>           <none>
node-exporter-bcm7c            1/1     Running   0               35m     10.244.0.61   minikube       <none>           <none>
```
**Note:** We can see, we have two pods: the first pod is running on the first node, `minikube` ("node-exporter-bcm7c"),
and the second pod is running on the second node, `minikube-m02` ("node-exporter-9wrcw"). Similarly, whenever a new node 
is added to the cluster, a new pod will be created. Conversely, if we delete any node, the pod running on that node will
be garbage collected by the DaemonSet.

### 9.Now lets try to delete the second node i.e, minikube-m02.
```
ubuntu@balasenapathi:~$ kubectl delete node minikube-m02
node "minikube-m02" deleted
```
### 10.Now try to list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS       AGE
node-exporter-bcm7c            1/1     Running   0              43m
```
**Note:** Now we have only one pod running because we have only one node in the cluster.

### 11.Now that we have monitoring pods running, let's check if they are providing metrics by port-forwarding these pods.
```
ubuntu@balasenapathi:~$ kubectl port-forward node-exporter-bcm7c 9100:9100
Forwarding from 127.0.0.1:9100 -> 9100
Forwarding from [::1]:9100 -> 9100
Handling connection for 9100
Handling connection for 9100
Handling connection for 9100
```
### 12.Now lets go to the browser:
```
[http://localhost:9100/metrics] ===> give this to get the metrics from prometheus node-exporter.
```
```
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 8
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.20.6"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 892168
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 892168
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.445624e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 697
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 7.140592e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 892168
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 1.572864e+06
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 2.29376e+06
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 8541
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 1.572864e+06
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 3.866624e+06
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 0
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 9238
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 1200
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 15600
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 34880
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 48960
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 4.194304e+06
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 783600
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 327680
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 327680
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 1.362868e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 5
# HELP node_boot_time_seconds Node boot time, in unixtime.
# TYPE node_boot_time_seconds gauge
node_boot_time_seconds 1.69693957e+09
# HELP node_context_switches_total Total number of context switches.
# TYPE node_context_switches_total counter
node_context_switches_total 3.7152889e+07
# HELP node_cooling_device_cur_state Current throttle state of the cooling device
# TYPE node_cooling_device_cur_state gauge
node_cooling_device_cur_state{name="0",type="Processor"} 0
node_cooling_device_cur_state{name="1",type="Processor"} 0
# HELP node_cooling_device_max_state Maximum throttle state of the cooling device
# TYPE node_cooling_device_max_state gauge
node_cooling_device_max_state{name="0",type="Processor"} 7
node_cooling_device_max_state{name="1",type="Processor"} 7
# HELP node_cpu_guest_seconds_total Seconds the CPUs spent in guests (VMs) for each mode.
# TYPE node_cpu_guest_seconds_total counter
node_cpu_guest_seconds_total{cpu="0",mode="nice"} 0
node_cpu_guest_seconds_total{cpu="0",mode="user"} 0
node_cpu_guest_seconds_total{cpu="1",mode="nice"} 0
node_cpu_guest_seconds_total{cpu="1",mode="user"} 0
# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 5731.8
node_cpu_seconds_total{cpu="0",mode="iowait"} 701.06
node_cpu_seconds_total{cpu="0",mode="irq"} 0
node_cpu_seconds_total{cpu="0",mode="nice"} 16.16
node_cpu_seconds_total{cpu="0",mode="softirq"} 71.46
node_cpu_seconds_total{cpu="0",mode="steal"} 0
node_cpu_seconds_total{cpu="0",mode="system"} 685.54
node_cpu_seconds_total{cpu="0",mode="user"} 699.7
node_cpu_seconds_total{cpu="1",mode="idle"} 5695.62
node_cpu_seconds_total{cpu="1",mode="iowait"} 715.64
node_cpu_seconds_total{cpu="1",mode="irq"} 0
node_cpu_seconds_total{cpu="1",mode="nice"} 24.78
node_cpu_seconds_total{cpu="1",mode="softirq"} 66.98
node_cpu_seconds_total{cpu="1",mode="steal"} 0
node_cpu_seconds_total{cpu="1",mode="system"} 684.96
node_cpu_seconds_total{cpu="1",mode="user"} 708.84
# HELP node_disk_discard_time_seconds_total This is the total number of seconds spent by all discards.
# TYPE node_disk_discard_time_seconds_total counter
node_disk_discard_time_seconds_total{device="sda"} 0
node_disk_discard_time_seconds_total{device="sr0"} 0
# HELP node_disk_discarded_sectors_total The total number of sectors discarded successfully.
# TYPE node_disk_discarded_sectors_total counter
node_disk_discarded_sectors_total{device="sda"} 0
node_disk_discarded_sectors_total{device="sr0"} 0
# HELP node_disk_discards_completed_total The total number of discards completed successfully.
# TYPE node_disk_discards_completed_total counter
node_disk_discards_completed_total{device="sda"} 0
node_disk_discards_completed_total{device="sr0"} 0
# HELP node_disk_discards_merged_total The total number of discards merged.
# TYPE node_disk_discards_merged_total counter
node_disk_discards_merged_total{device="sda"} 0
node_disk_discards_merged_total{device="sr0"} 0
# HELP node_disk_flush_requests_time_seconds_total This is the total number of seconds spent by all flush requests.
# TYPE node_disk_flush_requests_time_seconds_total counter
node_disk_flush_requests_time_seconds_total{device="sda"} 0
node_disk_flush_requests_time_seconds_total{device="sr0"} 0
# HELP node_disk_flush_requests_total The total number of flush requests completed successfully
# TYPE node_disk_flush_requests_total counter
node_disk_flush_requests_total{device="sda"} 0
node_disk_flush_requests_total{device="sr0"} 0
# HELP node_disk_info Info of /sys/block/<block_device>.
# TYPE node_disk_info gauge
node_disk_info{device="sda",major="8",minor="0",model="",path="",revision="",serial="",wwn=""} 1
node_disk_info{device="sr0",major="11",minor="0",model="",path="",revision="",serial="",wwn=""} 1
# HELP node_disk_io_now The number of I/Os currently in progress.
# TYPE node_disk_io_now gauge
node_disk_io_now{device="sda"} 1
node_disk_io_now{device="sr0"} 0
# HELP node_disk_io_time_seconds_total Total seconds spent doing I/Os.
# TYPE node_disk_io_time_seconds_total counter
node_disk_io_time_seconds_total{device="sda"} 1507.2640000000001
node_disk_io_time_seconds_total{device="sr0"} 9.392
# HELP node_disk_io_time_weighted_seconds_total The weighted # of seconds spent doing I/Os.
# TYPE node_disk_io_time_weighted_seconds_total counter
node_disk_io_time_weighted_seconds_total{device="sda"} 8035.887
node_disk_io_time_weighted_seconds_total{device="sr0"} 11.476
# HELP node_disk_read_bytes_total The total number of bytes read successfully.
# TYPE node_disk_read_bytes_total counter
node_disk_read_bytes_total{device="sda"} 7.91841792e+09
node_disk_read_bytes_total{device="sr0"} 1.104384e+06
# HELP node_disk_read_time_seconds_total The total number of seconds spent by all reads.
# TYPE node_disk_read_time_seconds_total counter
node_disk_read_time_seconds_total{device="sda"} 6705.756
node_disk_read_time_seconds_total{device="sr0"} 11.476
# HELP node_disk_reads_completed_total The total number of reads completed successfully.
# TYPE node_disk_reads_completed_total counter
node_disk_reads_completed_total{device="sda"} 192789
node_disk_reads_completed_total{device="sr0"} 56
# HELP node_disk_reads_merged_total The total number of reads merged.
# TYPE node_disk_reads_merged_total counter
node_disk_reads_merged_total{device="sda"} 74521
node_disk_reads_merged_total{device="sr0"} 0
# HELP node_disk_write_time_seconds_total This is the total number of seconds spent by all writes.
# TYPE node_disk_write_time_seconds_total counter
node_disk_write_time_seconds_total{device="sda"} 1330.13
node_disk_write_time_seconds_total{device="sr0"} 0
# HELP node_disk_writes_completed_total The total number of writes completed successfully.
# TYPE node_disk_writes_completed_total counter
node_disk_writes_completed_total{device="sda"} 108942
node_disk_writes_completed_total{device="sr0"} 0
# HELP node_disk_writes_merged_total The number of writes merged.
# TYPE node_disk_writes_merged_total counter
node_disk_writes_merged_total{device="sda"} 568339
node_disk_writes_merged_total{device="sr0"} 0
# HELP node_disk_written_bytes_total The total number of bytes written successfully.
# TYPE node_disk_written_bytes_total counter
node_disk_written_bytes_total{device="sda"} 5.30954752e+09
node_disk_written_bytes_total{device="sr0"} 0
# HELP node_dmi_info A metric with a constant '1' value labeled by bios_date, bios_release, bios_vendor, bios_version, board_asset_tag, 
board_name, board_serial, board_vendor, board_version, chassis_asset_tag, chassis_serial, chassis_vendor, chassis_version, 
product_family, product_name, product_serial, product_sku, product_uuid, product_version, system_vendor if provided by DMI.
# TYPE node_dmi_info gauge
node_dmi_info{bios_date="07/02/2015",bios_release="4.6",bios_vendor="Phoenix Technologies 
LTD",bios_version="6.00",board_asset_tag="",board_name="440BX Desktop Reference Platform",board_vendor="Intel 
Corporation",board_version="None",chassis_asset_tag="No Asset Tag",chassis_vendor="No Enclosure",chassis_version="N/
A",product_family="",product_name="kind",product_sku="",product_uuid="ad938390-4f57-4144-997c-6f80ed41efb5",product_version="None",system_vendor="VMware, 
Inc."} 1
# HELP node_entropy_available_bits Bits of available entropy.
# TYPE node_entropy_available_bits gauge
node_entropy_available_bits 256
# HELP node_entropy_pool_size_bits Bits of entropy pool.
# TYPE node_entropy_pool_size_bits gauge
node_entropy_pool_size_bits 256
# HELP node_exporter_build_info A metric with a constant '1' value labeled by version, revision, branch, goversion from which 
node_exporter was built, and the goos and goarch for the build.
# TYPE node_exporter_build_info gauge
node_exporter_build_info{branch="HEAD",goarch="amd64",goos="linux",goversion="go1.20.6",revision="4a1b77600c1873a8233f3ffb55afcedbb63b8d84",tags="netgo 
osusergo static_build",version="1.6.1"} 1
# HELP node_filefd_allocated File descriptor statistics: allocated.
# TYPE node_filefd_allocated gauge
node_filefd_allocated 12128
# HELP node_filefd_maximum File descriptor statistics: maximum.
# TYPE node_filefd_maximum gauge
node_filefd_maximum 9.223372036854776e+18
# HELP node_filesystem_avail_bytes Filesystem space available to non-root users in bytes.
# TYPE node_filesystem_avail_bytes gauge
node_filesystem_avail_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 3.2603992064e+10
node_filesystem_avail_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 3.2603992064e+10
node_filesystem_avail_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 3.2603992064e+10
node_filesystem_avail_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 3.2603992064e+10
node_filesystem_avail_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 3.2603992064e+10
# HELP node_filesystem_device_error Whether an error occurred while getting statistics for the given device.
# TYPE node_filesystem_device_error gauge
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/data"} 1
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 0
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 0
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 0
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/tmp/hostpath-provisioner"} 1
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/tmp/hostpath_pv"} 1
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/usr/lib/modules"} 1
node_filesystem_device_error{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 0
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/run"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/run/lock"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 0
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/0b2c2311-e6bf-44e9-8193-faed28133cfd/
volumes/kubernetes.io~projected/kube-api-access-59s6p"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/2bff57c5-b7a0-40e9-b6d2-c7c2c61fe86e/
volumes/kubernetes.io~projected/kube-api-access-gkh8w"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/771d4ea5-9572-44ca-a845-d722cf7ce5ce/
volumes/kubernetes.io~projected/kube-api-access-pf9qn"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/79bf07a3-c40a-4322-86cd-6f18192c84f8/
volumes/kubernetes.io~projected/kube-api-access-bz89x"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/866397fa-efd0-4efe-b9bd-e9165416e161/
volumes/kubernetes.io~projected/kube-api-access-jr52k"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/8cb3ed04-6043-4b76-a53d-9b837010568e/
volumes/kubernetes.io~projected/kube-api-access-2jv66"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/b36f3e30-3b07-411d-9c11-30510947368a/
volumes/kubernetes.io~projected/kube-api-access-fhgrb"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/b878f9cb-2b0a-4f10-8e24-1a48752fe103/
volumes/kubernetes.io~projected/kube-api-access-wb5h5"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/c4e28a54-f89f-4228-a98d-bdfa8bdcbb95/
volumes/kubernetes.io~projected/kube-api-access-5v52s"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/c4e28a54-f89f-4228-a98d-bdfa8bdcbb95/
volumes/kubernetes.io~secret/tls-certs"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/d43914dd-af6b-4c78-8952-b897283441a5/
volumes/kubernetes.io~projected/kube-api-access-46mqv"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/d55f78c0-cc64-45ea-b8c0-511c1a912740/
volumes/kubernetes.io~projected/kube-api-access-zgc8t"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/d9b29a00-052e-454c-b7b0-a3177d0ff724/
volumes/kubernetes.io~projected/kube-api-access-mj27m"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/e8225141-1770-4067-9ec9-89fe46d3b395/
volumes/kubernetes.io~projected/kube-api-access-pgxlf"} 1
node_filesystem_device_error{device="tmpfs",fstype="tmpfs",mountpoint="/var/lib/kubelet/pods/f9ef1f07-d041-478e-9784-3c932a58e09f/
volumes/kubernetes.io~projected/kube-api-access-z6gxp"} 1
# HELP node_filesystem_files Filesystem total file nodes.
# TYPE node_filesystem_files gauge
node_filesystem_files{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 6.520832e+06
node_filesystem_files{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 6.520832e+06
node_filesystem_files{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 6.520832e+06
node_filesystem_files{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 6.520832e+06
node_filesystem_files{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 6.520832e+06
# HELP node_filesystem_files_free Filesystem total free file nodes.
# TYPE node_filesystem_files_free gauge
node_filesystem_files_free{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 5.980704e+06
node_filesystem_files_free{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 5.980704e+06
node_filesystem_files_free{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 5.980704e+06
node_filesystem_files_free{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 5.980704e+06
node_filesystem_files_free{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 5.980704e+06
# HELP node_filesystem_free_bytes Filesystem free space in bytes.
# TYPE node_filesystem_free_bytes gauge
node_filesystem_free_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 3.7962477568e+10
node_filesystem_free_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 3.7962477568e+10
node_filesystem_free_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 3.7962477568e+10
node_filesystem_free_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 3.7962477568e+10
node_filesystem_free_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 3.7962477568e+10
# HELP node_filesystem_readonly Filesystem read-only status.
# TYPE node_filesystem_readonly gauge
node_filesystem_readonly{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 0
node_filesystem_readonly{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 0
node_filesystem_readonly{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 0
node_filesystem_readonly{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 0
node_filesystem_readonly{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 0
# HELP node_filesystem_size_bytes Filesystem size in bytes.
# TYPE node_filesystem_size_bytes gauge
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hostname"} 1.04557666304e+11
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/hosts"} 1.04557666304e+11
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/etc/resolv.conf"} 1.04557666304e+11
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/var"} 1.04557666304e+11
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/tmp"} 1.04557666304e+11
# HELP node_forks_total Total number of forks.
# TYPE node_forks_total counter
node_forks_total 196523
# HELP node_hwmon_chip_names Annotation metric for human-readable chip names
# TYPE node_hwmon_chip_names gauge
node_hwmon_chip_names{chip="platform_coretemp_0",chip_name="coretemp"} 1
node_hwmon_chip_names{chip="platform_coretemp_1",chip_name="coretemp"} 1
node_hwmon_chip_names{chip="power_supply_acad",chip_name="acad"} 1
# HELP node_hwmon_sensor_label Label for given chip and sensor
# TYPE node_hwmon_sensor_label gauge
node_hwmon_sensor_label{chip="platform_coretemp_0",label="Core 0",sensor="temp2"} 1
node_hwmon_sensor_label{chip="platform_coretemp_0",label="Package id 0",sensor="temp1"} 1
node_hwmon_sensor_label{chip="platform_coretemp_1",label="Core 0",sensor="temp2"} 1
node_hwmon_sensor_label{chip="platform_coretemp_1",label="Package id 1",sensor="temp1"} 1
# HELP node_hwmon_temp_celsius Hardware monitor for temperature (input)
# TYPE node_hwmon_temp_celsius gauge
node_hwmon_temp_celsius{chip="platform_coretemp_0",sensor="temp1"} 100
node_hwmon_temp_celsius{chip="platform_coretemp_0",sensor="temp2"} 100
node_hwmon_temp_celsius{chip="platform_coretemp_1",sensor="temp1"} 100
node_hwmon_temp_celsius{chip="platform_coretemp_1",sensor="temp2"} 100
# HELP node_hwmon_temp_crit_alarm_celsius Hardware monitor for temperature (crit_alarm)
# TYPE node_hwmon_temp_crit_alarm_celsius gauge
node_hwmon_temp_crit_alarm_celsius{chip="platform_coretemp_0",sensor="temp1"} 0
node_hwmon_temp_crit_alarm_celsius{chip="platform_coretemp_0",sensor="temp2"} 0
node_hwmon_temp_crit_alarm_celsius{chip="platform_coretemp_1",sensor="temp1"} 0
node_hwmon_temp_crit_alarm_celsius{chip="platform_coretemp_1",sensor="temp2"} 0
# HELP node_hwmon_temp_crit_celsius Hardware monitor for temperature (crit)
# TYPE node_hwmon_temp_crit_celsius gauge
node_hwmon_temp_crit_celsius{chip="platform_coretemp_0",sensor="temp1"} 100
node_hwmon_temp_crit_celsius{chip="platform_coretemp_0",sensor="temp2"} 100
node_hwmon_temp_crit_celsius{chip="platform_coretemp_1",sensor="temp1"} 100
node_hwmon_temp_crit_celsius{chip="platform_coretemp_1",sensor="temp2"} 100
# HELP node_hwmon_temp_max_celsius Hardware monitor for temperature (max)
# TYPE node_hwmon_temp_max_celsius gauge
node_hwmon_temp_max_celsius{chip="platform_coretemp_0",sensor="temp1"} 100
node_hwmon_temp_max_celsius{chip="platform_coretemp_0",sensor="temp2"} 100
node_hwmon_temp_max_celsius{chip="platform_coretemp_1",sensor="temp1"} 100
node_hwmon_temp_max_celsius{chip="platform_coretemp_1",sensor="temp2"} 100
# HELP node_intr_total Total number of interrupts serviced.
# TYPE node_intr_total counter
node_intr_total 1.6295328e+07
# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 2.67
# HELP node_load15 15m load average.
# TYPE node_load15 gauge
node_load15 2.29
# HELP node_load5 5m load average.
# TYPE node_load5 gauge
node_load5 2.28
# HELP node_memory_Active_anon_bytes Memory information field Active_anon_bytes.
# TYPE node_memory_Active_anon_bytes gauge
node_memory_Active_anon_bytes 6.31402496e+08
# HELP node_memory_Active_bytes Memory information field Active_bytes.
# TYPE node_memory_Active_bytes gauge
node_memory_Active_bytes 1.193529344e+09
# HELP node_memory_Active_file_bytes Memory information field Active_file_bytes.
# TYPE node_memory_Active_file_bytes gauge
node_memory_Active_file_bytes 5.62126848e+08
# HELP node_memory_AnonHugePages_bytes Memory information field AnonHugePages_bytes.
# TYPE node_memory_AnonHugePages_bytes gauge
node_memory_AnonHugePages_bytes 4.194304e+06
# HELP node_memory_AnonPages_bytes Memory information field AnonPages_bytes.
# TYPE node_memory_AnonPages_bytes gauge
node_memory_AnonPages_bytes 1.691287552e+09
# HELP node_memory_Bounce_bytes Memory information field Bounce_bytes.
# TYPE node_memory_Bounce_bytes gauge
node_memory_Bounce_bytes 0
# HELP node_memory_Buffers_bytes Memory information field Buffers_bytes.
# TYPE node_memory_Buffers_bytes gauge
node_memory_Buffers_bytes 3.3325056e+07
# HELP node_memory_Cached_bytes Memory information field Cached_bytes.
# TYPE node_memory_Cached_bytes gauge
node_memory_Cached_bytes 1.223811072e+09
# HELP node_memory_CommitLimit_bytes Memory information field CommitLimit_bytes.
# TYPE node_memory_CommitLimit_bytes gauge
node_memory_CommitLimit_bytes 4.179861504e+09
# HELP node_memory_Committed_AS_bytes Memory information field Committed_AS_bytes.
# TYPE node_memory_Committed_AS_bytes gauge
node_memory_Committed_AS_bytes 1.4546169856e+10
# HELP node_memory_DirectMap2M_bytes Memory information field DirectMap2M_bytes.
# TYPE node_memory_DirectMap2M_bytes gauge
node_memory_DirectMap2M_bytes 3.808428032e+09
# HELP node_memory_DirectMap4k_bytes Memory information field DirectMap4k_bytes.
# TYPE node_memory_DirectMap4k_bytes gauge
node_memory_DirectMap4k_bytes 4.86342656e+08
# HELP node_memory_Dirty_bytes Memory information field Dirty_bytes.
# TYPE node_memory_Dirty_bytes gauge
node_memory_Dirty_bytes 897024
# HELP node_memory_FileHugePages_bytes Memory information field FileHugePages_bytes.
# TYPE node_memory_FileHugePages_bytes gauge
node_memory_FileHugePages_bytes 0
# HELP node_memory_FilePmdMapped_bytes Memory information field FilePmdMapped_bytes.
# TYPE node_memory_FilePmdMapped_bytes gauge
node_memory_FilePmdMapped_bytes 0
# HELP node_memory_HardwareCorrupted_bytes Memory information field HardwareCorrupted_bytes.
# TYPE node_memory_HardwareCorrupted_bytes gauge
node_memory_HardwareCorrupted_bytes 0
# HELP node_memory_HugePages_Free Memory information field HugePages_Free.
# TYPE node_memory_HugePages_Free gauge
node_memory_HugePages_Free 0
# HELP node_memory_HugePages_Rsvd Memory information field HugePages_Rsvd.
# TYPE node_memory_HugePages_Rsvd gauge
node_memory_HugePages_Rsvd 0
# HELP node_memory_HugePages_Surp Memory information field HugePages_Surp.
# TYPE node_memory_HugePages_Surp gauge
node_memory_HugePages_Surp 0
# HELP node_memory_HugePages_Total Memory information field HugePages_Total.
# TYPE node_memory_HugePages_Total gauge
node_memory_HugePages_Total 0
# HELP node_memory_Hugepagesize_bytes Memory information field Hugepagesize_bytes.
# TYPE node_memory_Hugepagesize_bytes gauge
node_memory_Hugepagesize_bytes 2.097152e+06
# HELP node_memory_Hugetlb_bytes Memory information field Hugetlb_bytes.
# TYPE node_memory_Hugetlb_bytes gauge
node_memory_Hugetlb_bytes 0
# HELP node_memory_Inactive_anon_bytes Memory information field Inactive_anon_bytes.
# TYPE node_memory_Inactive_anon_bytes gauge
node_memory_Inactive_anon_bytes 1.183629312e+09
# HELP node_memory_Inactive_bytes Memory information field Inactive_bytes.
# TYPE node_memory_Inactive_bytes gauge
node_memory_Inactive_bytes 1.816154112e+09
# HELP node_memory_Inactive_file_bytes Memory information field Inactive_file_bytes.
# TYPE node_memory_Inactive_file_bytes gauge
node_memory_Inactive_file_bytes 6.325248e+08
# HELP node_memory_KReclaimable_bytes Memory information field KReclaimable_bytes.
# TYPE node_memory_KReclaimable_bytes gauge
node_memory_KReclaimable_bytes 1.43900672e+08
# HELP node_memory_KernelStack_bytes Memory information field KernelStack_bytes.
# TYPE node_memory_KernelStack_bytes gauge
node_memory_KernelStack_bytes 2.8565504e+07
# HELP node_memory_Mapped_bytes Memory information field Mapped_bytes.
# TYPE node_memory_Mapped_bytes gauge
node_memory_Mapped_bytes 7.50444544e+08
# HELP node_memory_MemAvailable_bytes Memory information field MemAvailable_bytes.
# TYPE node_memory_MemAvailable_bytes gauge
node_memory_MemAvailable_bytes 1.188900864e+09
# HELP node_memory_MemFree_bytes Memory information field MemFree_bytes.
# TYPE node_memory_MemFree_bytes gauge
node_memory_MemFree_bytes 1.31670016e+08
# HELP node_memory_MemTotal_bytes Memory information field MemTotal_bytes.
# TYPE node_memory_MemTotal_bytes gauge
node_memory_MemTotal_bytes 4.064763904e+09
# HELP node_memory_Mlocked_bytes Memory information field Mlocked_bytes.
# TYPE node_memory_Mlocked_bytes gauge
node_memory_Mlocked_bytes 16384
# HELP node_memory_NFS_Unstable_bytes Memory information field NFS_Unstable_bytes.
# TYPE node_memory_NFS_Unstable_bytes gauge
node_memory_NFS_Unstable_bytes 0
# HELP node_memory_PageTables_bytes Memory information field PageTables_bytes.
# TYPE node_memory_PageTables_bytes gauge
node_memory_PageTables_bytes 4.3917312e+07
# HELP node_memory_Percpu_bytes Memory information field Percpu_bytes.
# TYPE node_memory_Percpu_bytes gauge
node_memory_Percpu_bytes 2.10239488e+08
# HELP node_memory_SReclaimable_bytes Memory information field SReclaimable_bytes.
# TYPE node_memory_SReclaimable_bytes gauge
node_memory_SReclaimable_bytes 1.43900672e+08
# HELP node_memory_SUnreclaim_bytes Memory information field SUnreclaim_bytes.
# TYPE node_memory_SUnreclaim_bytes gauge
node_memory_SUnreclaim_bytes 2.13557248e+08
# HELP node_memory_ShmemHugePages_bytes Memory information field ShmemHugePages_bytes.
# TYPE node_memory_ShmemHugePages_bytes gauge
node_memory_ShmemHugePages_bytes 0
# HELP node_memory_ShmemPmdMapped_bytes Memory information field ShmemPmdMapped_bytes.
# TYPE node_memory_ShmemPmdMapped_bytes gauge
node_memory_ShmemPmdMapped_bytes 0
# HELP node_memory_Shmem_bytes Memory information field Shmem_bytes.
# TYPE node_memory_Shmem_bytes gauge
node_memory_Shmem_bytes 6.234112e+07
# HELP node_memory_Slab_bytes Memory information field Slab_bytes.
# TYPE node_memory_Slab_bytes gauge
node_memory_Slab_bytes 3.5745792e+08
# HELP node_memory_SwapCached_bytes Memory information field SwapCached_bytes.
# TYPE node_memory_SwapCached_bytes gauge
node_memory_SwapCached_bytes 6.490112e+07
# HELP node_memory_SwapFree_bytes Memory information field SwapFree_bytes.
# TYPE node_memory_SwapFree_bytes gauge
node_memory_SwapFree_bytes 2.9681664e+08
# HELP node_memory_SwapTotal_bytes Memory information field SwapTotal_bytes.
# TYPE node_memory_SwapTotal_bytes gauge
node_memory_SwapTotal_bytes 2.147479552e+09
# HELP node_memory_Unevictable_bytes Memory information field Unevictable_bytes.
# TYPE node_memory_Unevictable_bytes gauge
node_memory_Unevictable_bytes 16384
# HELP node_memory_VmallocChunk_bytes Memory information field VmallocChunk_bytes.
# TYPE node_memory_VmallocChunk_bytes gauge
node_memory_VmallocChunk_bytes 0
# HELP node_memory_VmallocTotal_bytes Memory information field VmallocTotal_bytes.
# TYPE node_memory_VmallocTotal_bytes gauge
node_memory_VmallocTotal_bytes 3.5184372087808e+13
# HELP node_memory_VmallocUsed_bytes Memory information field VmallocUsed_bytes.
# TYPE node_memory_VmallocUsed_bytes gauge
node_memory_VmallocUsed_bytes 8.9915392e+07
# HELP node_memory_WritebackTmp_bytes Memory information field WritebackTmp_bytes.
# TYPE node_memory_WritebackTmp_bytes gauge
node_memory_WritebackTmp_bytes 0
# HELP node_memory_Writeback_bytes Memory information field Writeback_bytes.
# TYPE node_memory_Writeback_bytes gauge
node_memory_Writeback_bytes 0
# HELP node_netstat_Icmp6_InErrors Statistic Icmp6InErrors.
# TYPE node_netstat_Icmp6_InErrors untyped
node_netstat_Icmp6_InErrors 0
# HELP node_netstat_Icmp6_InMsgs Statistic Icmp6InMsgs.
# TYPE node_netstat_Icmp6_InMsgs untyped
node_netstat_Icmp6_InMsgs 0
# HELP node_netstat_Icmp6_OutMsgs Statistic Icmp6OutMsgs.
# TYPE node_netstat_Icmp6_OutMsgs untyped
node_netstat_Icmp6_OutMsgs 0
# HELP node_netstat_Icmp_InErrors Statistic IcmpInErrors.
# TYPE node_netstat_Icmp_InErrors untyped
node_netstat_Icmp_InErrors 0
# HELP node_netstat_Icmp_InMsgs Statistic IcmpInMsgs.
# TYPE node_netstat_Icmp_InMsgs untyped
node_netstat_Icmp_InMsgs 0
# HELP node_netstat_Icmp_OutMsgs Statistic IcmpOutMsgs.
# TYPE node_netstat_Icmp_OutMsgs untyped
node_netstat_Icmp_OutMsgs 0
# HELP node_netstat_Ip6_InOctets Statistic Ip6InOctets.
# TYPE node_netstat_Ip6_InOctets untyped
node_netstat_Ip6_InOctets 0
# HELP node_netstat_Ip6_OutOctets Statistic Ip6OutOctets.
# TYPE node_netstat_Ip6_OutOctets untyped
node_netstat_Ip6_OutOctets 0
# HELP node_netstat_IpExt_InOctets Statistic IpExtInOctets.
# TYPE node_netstat_IpExt_InOctets untyped
node_netstat_IpExt_InOctets 1112
# HELP node_netstat_IpExt_OutOctets Statistic IpExtOutOctets.
# TYPE node_netstat_IpExt_OutOctets untyped
node_netstat_IpExt_OutOctets 1112
# HELP node_netstat_Ip_Forwarding Statistic IpForwarding.
# TYPE node_netstat_Ip_Forwarding untyped
node_netstat_Ip_Forwarding 1
# HELP node_netstat_TcpExt_ListenDrops Statistic TcpExtListenDrops.
# TYPE node_netstat_TcpExt_ListenDrops untyped
node_netstat_TcpExt_ListenDrops 0
# HELP node_netstat_TcpExt_ListenOverflows Statistic TcpExtListenOverflows.
# TYPE node_netstat_TcpExt_ListenOverflows untyped
node_netstat_TcpExt_ListenOverflows 0
# HELP node_netstat_TcpExt_SyncookiesFailed Statistic TcpExtSyncookiesFailed.
# TYPE node_netstat_TcpExt_SyncookiesFailed untyped
node_netstat_TcpExt_SyncookiesFailed 0
# HELP node_netstat_TcpExt_SyncookiesRecv Statistic TcpExtSyncookiesRecv.
# TYPE node_netstat_TcpExt_SyncookiesRecv untyped
node_netstat_TcpExt_SyncookiesRecv 0
# HELP node_netstat_TcpExt_SyncookiesSent Statistic TcpExtSyncookiesSent.
# TYPE node_netstat_TcpExt_SyncookiesSent untyped
node_netstat_TcpExt_SyncookiesSent 0
# HELP node_netstat_TcpExt_TCPSynRetrans Statistic TcpExtTCPSynRetrans.
# TYPE node_netstat_TcpExt_TCPSynRetrans untyped
node_netstat_TcpExt_TCPSynRetrans 0
# HELP node_netstat_TcpExt_TCPTimeouts Statistic TcpExtTCPTimeouts.
# TYPE node_netstat_TcpExt_TCPTimeouts untyped
node_netstat_TcpExt_TCPTimeouts 0
# HELP node_netstat_Tcp_ActiveOpens Statistic TcpActiveOpens.
# TYPE node_netstat_Tcp_ActiveOpens untyped
node_netstat_Tcp_ActiveOpens 2
# HELP node_netstat_Tcp_CurrEstab Statistic TcpCurrEstab.
# TYPE node_netstat_Tcp_CurrEstab untyped
node_netstat_Tcp_CurrEstab 4
# HELP node_netstat_Tcp_InErrs Statistic TcpInErrs.
# TYPE node_netstat_Tcp_InErrs untyped
node_netstat_Tcp_InErrs 0
# HELP node_netstat_Tcp_InSegs Statistic TcpInSegs.
# TYPE node_netstat_Tcp_InSegs untyped
node_netstat_Tcp_InSegs 8
# HELP node_netstat_Tcp_OutRsts Statistic TcpOutRsts.
# TYPE node_netstat_Tcp_OutRsts untyped
node_netstat_Tcp_OutRsts 0
# HELP node_netstat_Tcp_OutSegs Statistic TcpOutSegs.
# TYPE node_netstat_Tcp_OutSegs untyped
node_netstat_Tcp_OutSegs 8
# HELP node_netstat_Tcp_PassiveOpens Statistic TcpPassiveOpens.
# TYPE node_netstat_Tcp_PassiveOpens untyped
node_netstat_Tcp_PassiveOpens 2
# HELP node_netstat_Tcp_RetransSegs Statistic TcpRetransSegs.
# TYPE node_netstat_Tcp_RetransSegs untyped
node_netstat_Tcp_RetransSegs 0
# HELP node_netstat_Udp6_InDatagrams Statistic Udp6InDatagrams.
# TYPE node_netstat_Udp6_InDatagrams untyped
node_netstat_Udp6_InDatagrams 0
# HELP node_netstat_Udp6_InErrors Statistic Udp6InErrors.
# TYPE node_netstat_Udp6_InErrors untyped
node_netstat_Udp6_InErrors 0
# HELP node_netstat_Udp6_NoPorts Statistic Udp6NoPorts.
# TYPE node_netstat_Udp6_NoPorts untyped
node_netstat_Udp6_NoPorts 0
# HELP node_netstat_Udp6_OutDatagrams Statistic Udp6OutDatagrams.
# TYPE node_netstat_Udp6_OutDatagrams untyped
node_netstat_Udp6_OutDatagrams 0
# HELP node_netstat_Udp6_RcvbufErrors Statistic Udp6RcvbufErrors.
# TYPE node_netstat_Udp6_RcvbufErrors untyped
node_netstat_Udp6_RcvbufErrors 0
# HELP node_netstat_Udp6_SndbufErrors Statistic Udp6SndbufErrors.
# TYPE node_netstat_Udp6_SndbufErrors untyped
node_netstat_Udp6_SndbufErrors 0
# HELP node_netstat_UdpLite6_InErrors Statistic UdpLite6InErrors.
# TYPE node_netstat_UdpLite6_InErrors untyped
node_netstat_UdpLite6_InErrors 0
# HELP node_netstat_UdpLite_InErrors Statistic UdpLiteInErrors.
# TYPE node_netstat_UdpLite_InErrors untyped
node_netstat_UdpLite_InErrors 0
# HELP node_netstat_Udp_InDatagrams Statistic UdpInDatagrams.
# TYPE node_netstat_Udp_InDatagrams untyped
node_netstat_Udp_InDatagrams 0
# HELP node_netstat_Udp_InErrors Statistic UdpInErrors.
# TYPE node_netstat_Udp_InErrors untyped
node_netstat_Udp_InErrors 0
# HELP node_netstat_Udp_NoPorts Statistic UdpNoPorts.
# TYPE node_netstat_Udp_NoPorts untyped
node_netstat_Udp_NoPorts 0
# HELP node_netstat_Udp_OutDatagrams Statistic UdpOutDatagrams.
# TYPE node_netstat_Udp_OutDatagrams untyped
node_netstat_Udp_OutDatagrams 0
# HELP node_netstat_Udp_RcvbufErrors Statistic UdpRcvbufErrors.
# TYPE node_netstat_Udp_RcvbufErrors untyped
node_netstat_Udp_RcvbufErrors 0
# HELP node_netstat_Udp_SndbufErrors Statistic UdpSndbufErrors.
# TYPE node_netstat_Udp_SndbufErrors untyped
node_netstat_Udp_SndbufErrors 0
# HELP node_network_address_assign_type Network device property: address_assign_type
# TYPE node_network_address_assign_type gauge
node_network_address_assign_type{device="bridge"} 3
node_network_address_assign_type{device="docker0"} 3
node_network_address_assign_type{device="eth0"} 3
node_network_address_assign_type{device="lo"} 0
node_network_address_assign_type{device="veth13db11b5"} 1
node_network_address_assign_type{device="veth2ed5a6d2"} 1
node_network_address_assign_type{device="veth33aff603"} 1
node_network_address_assign_type{device="veth58f0f152"} 1
node_network_address_assign_type{device="veth6a5b2816"} 1
node_network_address_assign_type{device="veth6eba0277"} 1
node_network_address_assign_type{device="vetha4218b25"} 1
node_network_address_assign_type{device="vethaeec7e91"} 1
node_network_address_assign_type{device="vethb556750f"} 1
node_network_address_assign_type{device="vethb9bb9063"} 1
node_network_address_assign_type{device="vethcc58f76c"} 1
# HELP node_network_carrier Network device property: carrier
# TYPE node_network_carrier gauge
node_network_carrier{device="bridge"} 1
node_network_carrier{device="docker0"} 0
node_network_carrier{device="eth0"} 1
node_network_carrier{device="lo"} 1
node_network_carrier{device="veth13db11b5"} 1
node_network_carrier{device="veth2ed5a6d2"} 1
node_network_carrier{device="veth33aff603"} 1
node_network_carrier{device="veth58f0f152"} 1
node_network_carrier{device="veth6a5b2816"} 1
node_network_carrier{device="veth6eba0277"} 1
node_network_carrier{device="vetha4218b25"} 1
node_network_carrier{device="vethaeec7e91"} 1
node_network_carrier{device="vethb556750f"} 1
node_network_carrier{device="vethb9bb9063"} 1
node_network_carrier{device="vethcc58f76c"} 1
# HELP node_network_carrier_changes_total Network device property: carrier_changes_total
# TYPE node_network_carrier_changes_total counter
node_network_carrier_changes_total{device="bridge"} 4
node_network_carrier_changes_total{device="docker0"} 1
node_network_carrier_changes_total{device="eth0"} 2
node_network_carrier_changes_total{device="lo"} 0
node_network_carrier_changes_total{device="veth13db11b5"} 2
node_network_carrier_changes_total{device="veth2ed5a6d2"} 2
node_network_carrier_changes_total{device="veth33aff603"} 2
node_network_carrier_changes_total{device="veth58f0f152"} 2
node_network_carrier_changes_total{device="veth6a5b2816"} 2
node_network_carrier_changes_total{device="veth6eba0277"} 2
node_network_carrier_changes_total{device="vetha4218b25"} 2
node_network_carrier_changes_total{device="vethaeec7e91"} 2
node_network_carrier_changes_total{device="vethb556750f"} 2
node_network_carrier_changes_total{device="vethb9bb9063"} 2
node_network_carrier_changes_total{device="vethcc58f76c"} 2
# HELP node_network_carrier_down_changes_total Network device property: carrier_down_changes_total
# TYPE node_network_carrier_down_changes_total counter
node_network_carrier_down_changes_total{device="bridge"} 2
node_network_carrier_down_changes_total{device="docker0"} 1
node_network_carrier_down_changes_total{device="eth0"} 1
node_network_carrier_down_changes_total{device="lo"} 0
node_network_carrier_down_changes_total{device="veth13db11b5"} 1
node_network_carrier_down_changes_total{device="veth2ed5a6d2"} 1
node_network_carrier_down_changes_total{device="veth33aff603"} 1
node_network_carrier_down_changes_total{device="veth58f0f152"} 1
node_network_carrier_down_changes_total{device="veth6a5b2816"} 1
node_network_carrier_down_changes_total{device="veth6eba0277"} 1
node_network_carrier_down_changes_total{device="vetha4218b25"} 1
node_network_carrier_down_changes_total{device="vethaeec7e91"} 1
node_network_carrier_down_changes_total{device="vethb556750f"} 1
node_network_carrier_down_changes_total{device="vethb9bb9063"} 1
node_network_carrier_down_changes_total{device="vethcc58f76c"} 1
# HELP node_network_carrier_up_changes_total Network device property: carrier_up_changes_total
# TYPE node_network_carrier_up_changes_total counter
node_network_carrier_up_changes_total{device="bridge"} 2
node_network_carrier_up_changes_total{device="docker0"} 0
node_network_carrier_up_changes_total{device="eth0"} 1
node_network_carrier_up_changes_total{device="lo"} 0
node_network_carrier_up_changes_total{device="veth13db11b5"} 1
node_network_carrier_up_changes_total{device="veth2ed5a6d2"} 1
node_network_carrier_up_changes_total{device="veth33aff603"} 1
node_network_carrier_up_changes_total{device="veth58f0f152"} 1
node_network_carrier_up_changes_total{device="veth6a5b2816"} 1
node_network_carrier_up_changes_total{device="veth6eba0277"} 1
node_network_carrier_up_changes_total{device="vetha4218b25"} 1
node_network_carrier_up_changes_total{device="vethaeec7e91"} 1
node_network_carrier_up_changes_total{device="vethb556750f"} 1
node_network_carrier_up_changes_total{device="vethb9bb9063"} 1
node_network_carrier_up_changes_total{device="vethcc58f76c"} 1
# HELP node_network_device_id Network device property: device_id
# TYPE node_network_device_id gauge
node_network_device_id{device="bridge"} 0
node_network_device_id{device="docker0"} 0
node_network_device_id{device="eth0"} 0
node_network_device_id{device="lo"} 0
node_network_device_id{device="veth13db11b5"} 0
node_network_device_id{device="veth2ed5a6d2"} 0
node_network_device_id{device="veth33aff603"} 0
node_network_device_id{device="veth58f0f152"} 0
node_network_device_id{device="veth6a5b2816"} 0
node_network_device_id{device="veth6eba0277"} 0
node_network_device_id{device="vetha4218b25"} 0
node_network_device_id{device="vethaeec7e91"} 0
node_network_device_id{device="vethb556750f"} 0
node_network_device_id{device="vethb9bb9063"} 0
node_network_device_id{device="vethcc58f76c"} 0
# HELP node_network_dormant Network device property: dormant
# TYPE node_network_dormant gauge
node_network_dormant{device="bridge"} 0
node_network_dormant{device="docker0"} 0
node_network_dormant{device="eth0"} 0
node_network_dormant{device="lo"} 0
node_network_dormant{device="veth13db11b5"} 0
node_network_dormant{device="veth2ed5a6d2"} 0
node_network_dormant{device="veth33aff603"} 0
node_network_dormant{device="veth58f0f152"} 0
node_network_dormant{device="veth6a5b2816"} 0
node_network_dormant{device="veth6eba0277"} 0
node_network_dormant{device="vetha4218b25"} 0
node_network_dormant{device="vethaeec7e91"} 0
node_network_dormant{device="vethb556750f"} 0
node_network_dormant{device="vethb9bb9063"} 0
node_network_dormant{device="vethcc58f76c"} 0
# HELP node_network_flags Network device property: flags
# TYPE node_network_flags gauge
node_network_flags{device="bridge"} 4099
node_network_flags{device="docker0"} 4099
node_network_flags{device="eth0"} 4099
node_network_flags{device="lo"} 9
node_network_flags{device="veth13db11b5"} 4867
node_network_flags{device="veth2ed5a6d2"} 4867
node_network_flags{device="veth33aff603"} 4867
node_network_flags{device="veth58f0f152"} 4867
node_network_flags{device="veth6a5b2816"} 4867
node_network_flags{device="veth6eba0277"} 4867
node_network_flags{device="vetha4218b25"} 4867
node_network_flags{device="vethaeec7e91"} 4867
node_network_flags{device="vethb556750f"} 4867
node_network_flags{device="vethb9bb9063"} 4867
node_network_flags{device="vethcc58f76c"} 4867
# HELP node_network_iface_id Network device property: iface_id
# TYPE node_network_iface_id gauge
node_network_iface_id{device="bridge"} 3
node_network_iface_id{device="docker0"} 2
node_network_iface_id{device="eth0"} 5
node_network_iface_id{device="lo"} 1
node_network_iface_id{device="veth13db11b5"} 8
node_network_iface_id{device="veth2ed5a6d2"} 7
node_network_iface_id{device="veth33aff603"} 10
node_network_iface_id{device="veth58f0f152"} 14
node_network_iface_id{device="veth6a5b2816"} 12
node_network_iface_id{device="veth6eba0277"} 4
node_network_iface_id{device="vetha4218b25"} 11
node_network_iface_id{device="vethaeec7e91"} 16
node_network_iface_id{device="vethb556750f"} 6
node_network_iface_id{device="vethb9bb9063"} 13
node_network_iface_id{device="vethcc58f76c"} 9
# HELP node_network_iface_link Network device property: iface_link
# TYPE node_network_iface_link gauge
node_network_iface_link{device="bridge"} 3
node_network_iface_link{device="docker0"} 2
node_network_iface_link{device="eth0"} 6
node_network_iface_link{device="lo"} 1
node_network_iface_link{device="veth13db11b5"} 2
node_network_iface_link{device="veth2ed5a6d2"} 2
node_network_iface_link{device="veth33aff603"} 2
node_network_iface_link{device="veth58f0f152"} 2
node_network_iface_link{device="veth6a5b2816"} 2
node_network_iface_link{device="veth6eba0277"} 2
node_network_iface_link{device="vetha4218b25"} 2
node_network_iface_link{device="vethaeec7e91"} 2
node_network_iface_link{device="vethb556750f"} 2
node_network_iface_link{device="vethb9bb9063"} 2
node_network_iface_link{device="vethcc58f76c"} 2
# HELP node_network_iface_link_mode Network device property: iface_link_mode
# TYPE node_network_iface_link_mode gauge
node_network_iface_link_mode{device="bridge"} 0
node_network_iface_link_mode{device="docker0"} 0
node_network_iface_link_mode{device="eth0"} 0
node_network_iface_link_mode{device="lo"} 0
node_network_iface_link_mode{device="veth13db11b5"} 0
node_network_iface_link_mode{device="veth2ed5a6d2"} 0
node_network_iface_link_mode{device="veth33aff603"} 0
node_network_iface_link_mode{device="veth58f0f152"} 0
node_network_iface_link_mode{device="veth6a5b2816"} 0
node_network_iface_link_mode{device="veth6eba0277"} 0
node_network_iface_link_mode{device="vetha4218b25"} 0
node_network_iface_link_mode{device="vethaeec7e91"} 0
node_network_iface_link_mode{device="vethb556750f"} 0
node_network_iface_link_mode{device="vethb9bb9063"} 0
node_network_iface_link_mode{device="vethcc58f76c"} 0
# HELP node_network_info Non-numeric data from /sys/class/net/<iface>, value is always 1.
# TYPE node_network_info gauge
node_network_info{address="00:00:00:00:00:00",adminstate="up",broadcast="00:00:00:00:00:00",device="lo",duplex="",ifalias="",operstate="unknown"} 
1
node_network_info{address="02:42:40:33:0d:d8",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="docker0",duplex="unknown",ifalias="",operstate="down"} 
1
node_network_info{address="02:42:c0:a8:31:02",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="eth0",duplex="full",ifalias="",operstate="up"} 
1
node_network_info{address="36:be:
79:68:83:ab",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="vethb556750f",duplex="full",ifalias="",operstate="up"} 1
node_network_info{address="3e:4c:
16:99:59:da",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="vethaeec7e91",duplex="full",ifalias="",operstate="up"} 1
node_network_info{address="3e:c0:69:78:fd:eb",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="vethb9bb9063",duplex="full",ifalias="",operstate="up"} 
1
node_network_info{address="46:56:98:93:7e:
63",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="veth33aff603",duplex="full",ifalias="",operstate="up"} 1
node_network_info{address="5a:a9:6c:
88:a5:ce",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="vetha4218b25",duplex="full",ifalias="",operstate="up"} 1
node_network_info{address="5a:c0:e9:ef:
2c:ce",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="veth13db11b5",duplex="full",ifalias="",operstate="up"} 1
node_network_info{address="72:54:11:75:c3:20",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="bridge",duplex="unknown",ifalias="",operstate="up"} 
1
node_network_info{address="82:11:36:20:48:78",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="veth6eba0277",duplex="full",ifalias="",operstate="up"} 
1
node_network_info{address="96:63:a3:88:42:2f",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="vethcc58f76c",duplex="full",ifalias="",operstate="up"} 
1
node_network_info{address="d2:f7:90:e7:99:f9",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="veth6a5b2816",duplex="full",ifalias="",operstate="up"} 
1
node_network_info{address="da:57:7f:
0e:b9:a4",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="veth58f0f152",duplex="full",ifalias="",operstate="up"} 1
node_network_info{address="fe:a0:32:f7:2c:
94",adminstate="up",broadcast="ff:ff:ff:ff:ff:ff",device="veth2ed5a6d2",duplex="full",ifalias="",operstate="up"} 1
# HELP node_network_mtu_bytes Network device property: mtu_bytes
# TYPE node_network_mtu_bytes gauge
node_network_mtu_bytes{device="bridge"} 1500
node_network_mtu_bytes{device="docker0"} 1500
node_network_mtu_bytes{device="eth0"} 1500
node_network_mtu_bytes{device="lo"} 65536
node_network_mtu_bytes{device="veth13db11b5"} 1500
node_network_mtu_bytes{device="veth2ed5a6d2"} 1500
node_network_mtu_bytes{device="veth33aff603"} 1500
node_network_mtu_bytes{device="veth58f0f152"} 1500
node_network_mtu_bytes{device="veth6a5b2816"} 1500
node_network_mtu_bytes{device="veth6eba0277"} 1500
node_network_mtu_bytes{device="vetha4218b25"} 1500
node_network_mtu_bytes{device="vethaeec7e91"} 1500
node_network_mtu_bytes{device="vethb556750f"} 1500
node_network_mtu_bytes{device="vethb9bb9063"} 1500
node_network_mtu_bytes{device="vethcc58f76c"} 1500
# HELP node_network_name_assign_type Network device property: name_assign_type
# TYPE node_network_name_assign_type gauge
node_network_name_assign_type{device="bridge"} 3
node_network_name_assign_type{device="docker0"} 3
node_network_name_assign_type{device="eth0"} 4
node_network_name_assign_type{device="lo"} 2
node_network_name_assign_type{device="veth13db11b5"} 3
node_network_name_assign_type{device="veth2ed5a6d2"} 3
node_network_name_assign_type{device="veth33aff603"} 3
node_network_name_assign_type{device="veth58f0f152"} 3
node_network_name_assign_type{device="veth6a5b2816"} 3
node_network_name_assign_type{device="veth6eba0277"} 3
node_network_name_assign_type{device="vetha4218b25"} 3
node_network_name_assign_type{device="vethaeec7e91"} 3
node_network_name_assign_type{device="vethb556750f"} 3
node_network_name_assign_type{device="vethb9bb9063"} 3
node_network_name_assign_type{device="vethcc58f76c"} 3
# HELP node_network_net_dev_group Network device property: net_dev_group
# TYPE node_network_net_dev_group gauge
node_network_net_dev_group{device="bridge"} 0
node_network_net_dev_group{device="docker0"} 0
node_network_net_dev_group{device="eth0"} 0
node_network_net_dev_group{device="lo"} 0
node_network_net_dev_group{device="veth13db11b5"} 0
node_network_net_dev_group{device="veth2ed5a6d2"} 0
node_network_net_dev_group{device="veth33aff603"} 0
node_network_net_dev_group{device="veth58f0f152"} 0
node_network_net_dev_group{device="veth6a5b2816"} 0
node_network_net_dev_group{device="veth6eba0277"} 0
node_network_net_dev_group{device="vetha4218b25"} 0
node_network_net_dev_group{device="vethaeec7e91"} 0
node_network_net_dev_group{device="vethb556750f"} 0
node_network_net_dev_group{device="vethb9bb9063"} 0
node_network_net_dev_group{device="vethcc58f76c"} 0
# HELP node_network_protocol_type Network device property: protocol_type
# TYPE node_network_protocol_type gauge
node_network_protocol_type{device="bridge"} 1
node_network_protocol_type{device="docker0"} 1
node_network_protocol_type{device="eth0"} 1
node_network_protocol_type{device="lo"} 772
node_network_protocol_type{device="veth13db11b5"} 1
node_network_protocol_type{device="veth2ed5a6d2"} 1
node_network_protocol_type{device="veth33aff603"} 1
node_network_protocol_type{device="veth58f0f152"} 1
node_network_protocol_type{device="veth6a5b2816"} 1
node_network_protocol_type{device="veth6eba0277"} 1
node_network_protocol_type{device="vetha4218b25"} 1
node_network_protocol_type{device="vethaeec7e91"} 1
node_network_protocol_type{device="vethb556750f"} 1
node_network_protocol_type{device="vethb9bb9063"} 1
node_network_protocol_type{device="vethcc58f76c"} 1
# HELP node_network_receive_bytes_total Network device statistic receive_bytes.
# TYPE node_network_receive_bytes_total counter
node_network_receive_bytes_total{device="eth0"} 0
node_network_receive_bytes_total{device="lo"} 1112
# HELP node_network_receive_compressed_total Network device statistic receive_compressed.
# TYPE node_network_receive_compressed_total counter
node_network_receive_compressed_total{device="eth0"} 0
node_network_receive_compressed_total{device="lo"} 0
# HELP node_network_receive_drop_total Network device statistic receive_drop.
# TYPE node_network_receive_drop_total counter
node_network_receive_drop_total{device="eth0"} 0
node_network_receive_drop_total{device="lo"} 0
# HELP node_network_receive_errs_total Network device statistic receive_errs.
# TYPE node_network_receive_errs_total counter
node_network_receive_errs_total{device="eth0"} 0
node_network_receive_errs_total{device="lo"} 0
# HELP node_network_receive_fifo_total Network device statistic receive_fifo.
# TYPE node_network_receive_fifo_total counter
node_network_receive_fifo_total{device="eth0"} 0
node_network_receive_fifo_total{device="lo"} 0
# HELP node_network_receive_frame_total Network device statistic receive_frame.
# TYPE node_network_receive_frame_total counter
node_network_receive_frame_total{device="eth0"} 0
node_network_receive_frame_total{device="lo"} 0
# HELP node_network_receive_multicast_total Network device statistic receive_multicast.
# TYPE node_network_receive_multicast_total counter
node_network_receive_multicast_total{device="eth0"} 0
node_network_receive_multicast_total{device="lo"} 0
# HELP node_network_receive_nohandler_total Network device statistic receive_nohandler.
# TYPE node_network_receive_nohandler_total counter
node_network_receive_nohandler_total{device="eth0"} 0
node_network_receive_nohandler_total{device="lo"} 0
# HELP node_network_receive_packets_total Network device statistic receive_packets.
# TYPE node_network_receive_packets_total counter
node_network_receive_packets_total{device="eth0"} 0
node_network_receive_packets_total{device="lo"} 8
# HELP node_network_speed_bytes Network device property: speed_bytes
# TYPE node_network_speed_bytes gauge
node_network_speed_bytes{device="bridge"} 1.25e+09
node_network_speed_bytes{device="docker0"} -125000
node_network_speed_bytes{device="eth0"} 1.25e+09
node_network_speed_bytes{device="veth13db11b5"} 1.25e+09
node_network_speed_bytes{device="veth2ed5a6d2"} 1.25e+09
node_network_speed_bytes{device="veth33aff603"} 1.25e+09
node_network_speed_bytes{device="veth58f0f152"} 1.25e+09
node_network_speed_bytes{device="veth6a5b2816"} 1.25e+09
node_network_speed_bytes{device="veth6eba0277"} 1.25e+09
node_network_speed_bytes{device="vetha4218b25"} 1.25e+09
node_network_speed_bytes{device="vethaeec7e91"} 1.25e+09
node_network_speed_bytes{device="vethb556750f"} 1.25e+09
node_network_speed_bytes{device="vethb9bb9063"} 1.25e+09
node_network_speed_bytes{device="vethcc58f76c"} 1.25e+09
# HELP node_network_transmit_bytes_total Network device statistic transmit_bytes.
# TYPE node_network_transmit_bytes_total counter
node_network_transmit_bytes_total{device="eth0"} 42
node_network_transmit_bytes_total{device="lo"} 1112
# HELP node_network_transmit_carrier_total Network device statistic transmit_carrier.
# TYPE node_network_transmit_carrier_total counter
node_network_transmit_carrier_total{device="eth0"} 0
node_network_transmit_carrier_total{device="lo"} 0
# HELP node_network_transmit_colls_total Network device statistic transmit_colls.
# TYPE node_network_transmit_colls_total counter
node_network_transmit_colls_total{device="eth0"} 0
node_network_transmit_colls_total{device="lo"} 0
# HELP node_network_transmit_compressed_total Network device statistic transmit_compressed.
# TYPE node_network_transmit_compressed_total counter
node_network_transmit_compressed_total{device="eth0"} 0
node_network_transmit_compressed_total{device="lo"} 0
# HELP node_network_transmit_drop_total Network device statistic transmit_drop.
# TYPE node_network_transmit_drop_total counter
node_network_transmit_drop_total{device="eth0"} 0
node_network_transmit_drop_total{device="lo"} 0
# HELP node_network_transmit_errs_total Network device statistic transmit_errs.
# TYPE node_network_transmit_errs_total counter
node_network_transmit_errs_total{device="eth0"} 0
node_network_transmit_errs_total{device="lo"} 0
# HELP node_network_transmit_fifo_total Network device statistic transmit_fifo.
# TYPE node_network_transmit_fifo_total counter
node_network_transmit_fifo_total{device="eth0"} 0
node_network_transmit_fifo_total{device="lo"} 0
# HELP node_network_transmit_packets_total Network device statistic transmit_packets.
# TYPE node_network_transmit_packets_total counter
node_network_transmit_packets_total{device="eth0"} 1
node_network_transmit_packets_total{device="lo"} 8
# HELP node_network_transmit_queue_length Network device property: transmit_queue_length
# TYPE node_network_transmit_queue_length gauge
node_network_transmit_queue_length{device="bridge"} 1000
node_network_transmit_queue_length{device="docker0"} 0
node_network_transmit_queue_length{device="eth0"} 0
node_network_transmit_queue_length{device="lo"} 1000
node_network_transmit_queue_length{device="veth13db11b5"} 0
node_network_transmit_queue_length{device="veth2ed5a6d2"} 0
node_network_transmit_queue_length{device="veth33aff603"} 0
node_network_transmit_queue_length{device="veth58f0f152"} 0
node_network_transmit_queue_length{device="veth6a5b2816"} 0
node_network_transmit_queue_length{device="veth6eba0277"} 0
node_network_transmit_queue_length{device="vetha4218b25"} 0
node_network_transmit_queue_length{device="vethaeec7e91"} 0
node_network_transmit_queue_length{device="vethb556750f"} 0
node_network_transmit_queue_length{device="vethb9bb9063"} 0
node_network_transmit_queue_length{device="vethcc58f76c"} 0
# HELP node_network_up Value is 1 if operstate is 'up', 0 otherwise.
# TYPE node_network_up gauge
node_network_up{device="bridge"} 1
node_network_up{device="docker0"} 0
node_network_up{device="eth0"} 1
node_network_up{device="lo"} 0
node_network_up{device="veth13db11b5"} 1
node_network_up{device="veth2ed5a6d2"} 1
node_network_up{device="veth33aff603"} 1
node_network_up{device="veth58f0f152"} 1
node_network_up{device="veth6a5b2816"} 1
node_network_up{device="veth6eba0277"} 1
node_network_up{device="vetha4218b25"} 1
node_network_up{device="vethaeec7e91"} 1
node_network_up{device="vethb556750f"} 1
node_network_up{device="vethb9bb9063"} 1
node_network_up{device="vethcc58f76c"} 1
# HELP node_nf_conntrack_entries Number of currently allocated flow entries for connection tracking.
# TYPE node_nf_conntrack_entries gauge
node_nf_conntrack_entries 0
# HELP node_nf_conntrack_entries_limit Maximum size of connection tracking table.
# TYPE node_nf_conntrack_entries_limit gauge
node_nf_conntrack_entries_limit 65536
# HELP node_power_supply_info info of /sys/class/power_supply/<power_supply>.
# TYPE node_power_supply_info gauge
node_power_supply_info{power_supply="ACAD",type="Mains"} 1
# HELP node_power_supply_online online value of /sys/class/power_supply/<power_supply>.
# TYPE node_power_supply_online gauge
node_power_supply_online{power_supply="ACAD"} 1
# HELP node_pressure_cpu_waiting_seconds_total Total time in seconds that processes have waited for CPU time
# TYPE node_pressure_cpu_waiting_seconds_total counter
node_pressure_cpu_waiting_seconds_total 1016.570075
# HELP node_pressure_io_stalled_seconds_total Total time in seconds no process could make progress due to IO congestion
# TYPE node_pressure_io_stalled_seconds_total counter
node_pressure_io_stalled_seconds_total 750.175329
# HELP node_pressure_io_waiting_seconds_total Total time in seconds that processes have waited due to IO congestion
# TYPE node_pressure_io_waiting_seconds_total counter
node_pressure_io_waiting_seconds_total 1099.2220379999999
# HELP node_pressure_memory_stalled_seconds_total Total time in seconds no process could make progress due to memory congestion
# TYPE node_pressure_memory_stalled_seconds_total counter
node_pressure_memory_stalled_seconds_total 291.470702
# HELP node_pressure_memory_waiting_seconds_total Total time in seconds that processes have waited for memory
# TYPE node_pressure_memory_waiting_seconds_total counter
node_pressure_memory_waiting_seconds_total 443.418529
# HELP node_procs_blocked Number of processes blocked waiting for I/O to complete.
# TYPE node_procs_blocked gauge
node_procs_blocked 1
# HELP node_procs_running Number of processes in runnable state.
# TYPE node_procs_running gauge
node_procs_running 2
# HELP node_schedstat_running_seconds_total Number of seconds CPU spent running a process.
# TYPE node_schedstat_running_seconds_total counter
node_schedstat_running_seconds_total{cpu="0"} 1759.035843968
node_schedstat_running_seconds_total{cpu="1"} 1772.862933092
# HELP node_schedstat_timeslices_total Number of timeslices executed by CPU.
# TYPE node_schedstat_timeslices_total counter
node_schedstat_timeslices_total{cpu="0"} 1.4373439e+07
node_schedstat_timeslices_total{cpu="1"} 1.4279248e+07
# HELP node_schedstat_waiting_seconds_total Number of seconds spent by processing waiting for this CPU.
# TYPE node_schedstat_waiting_seconds_total counter
node_schedstat_waiting_seconds_total{cpu="0"} 2378.781336969
node_schedstat_waiting_seconds_total{cpu="1"} 2352.944740242
# HELP node_scrape_collector_duration_seconds node_exporter: Duration of a collector scrape.
# TYPE node_scrape_collector_duration_seconds gauge
node_scrape_collector_duration_seconds{collector="arp"} 0.000106934
node_scrape_collector_duration_seconds{collector="bcache"} 0.000639365
node_scrape_collector_duration_seconds{collector="bonding"} 2.8723e-05
node_scrape_collector_duration_seconds{collector="btrfs"} 0.001109069
node_scrape_collector_duration_seconds{collector="conntrack"} 0.000279912
node_scrape_collector_duration_seconds{collector="cpu"} 0.000947208
node_scrape_collector_duration_seconds{collector="cpufreq"} 0.000129016
node_scrape_collector_duration_seconds{collector="diskstats"} 0.002849358
node_scrape_collector_duration_seconds{collector="dmi"} 3.4389e-05
node_scrape_collector_duration_seconds{collector="edac"} 7.9091e-05
node_scrape_collector_duration_seconds{collector="entropy"} 0.000178878
node_scrape_collector_duration_seconds{collector="fibrechannel"} 2.1342e-05
node_scrape_collector_duration_seconds{collector="filefd"} 7.3261e-05
node_scrape_collector_duration_seconds{collector="filesystem"} 0.0924408
node_scrape_collector_duration_seconds{collector="hwmon"} 0.007125394
node_scrape_collector_duration_seconds{collector="infiniband"} 0.12767282
node_scrape_collector_duration_seconds{collector="ipvs"} 3.9041e-05
node_scrape_collector_duration_seconds{collector="loadavg"} 0.00010188
node_scrape_collector_duration_seconds{collector="mdadm"} 6.1902e-05
node_scrape_collector_duration_seconds{collector="meminfo"} 0.000309533
node_scrape_collector_duration_seconds{collector="netclass"} 0.061453924
node_scrape_collector_duration_seconds{collector="netdev"} 0.003959721
node_scrape_collector_duration_seconds{collector="netstat"} 0.002252647
node_scrape_collector_duration_seconds{collector="nfs"} 9.4514e-05
node_scrape_collector_duration_seconds{collector="nfsd"} 2.3991e-05
node_scrape_collector_duration_seconds{collector="nvme"} 4.7784e-05
node_scrape_collector_duration_seconds{collector="os"} 0.0320769
node_scrape_collector_duration_seconds{collector="powersupplyclass"} 0.000744484
node_scrape_collector_duration_seconds{collector="pressure"} 0.05909214
node_scrape_collector_duration_seconds{collector="rapl"} 0.000104886
node_scrape_collector_duration_seconds{collector="schedstat"} 7.702e-05
node_scrape_collector_duration_seconds{collector="selinux"} 0.001135488
node_scrape_collector_duration_seconds{collector="sockstat"} 0.001334309
node_scrape_collector_duration_seconds{collector="softnet"} 7.1027e-05
node_scrape_collector_duration_seconds{collector="stat"} 0.001563961
node_scrape_collector_duration_seconds{collector="tapestats"} 3.2463e-05
node_scrape_collector_duration_seconds{collector="textfile"} 0.001363715
node_scrape_collector_duration_seconds{collector="thermal_zone"} 0.000291149
node_scrape_collector_duration_seconds{collector="time"} 0.019964045
node_scrape_collector_duration_seconds{collector="timex"} 0.140087536
node_scrape_collector_duration_seconds{collector="udp_queues"} 0.00025805
node_scrape_collector_duration_seconds{collector="uname"} 4.2186e-05
node_scrape_collector_duration_seconds{collector="vmstat"} 0.000434587
node_scrape_collector_duration_seconds{collector="xfs"} 2.7853e-05
node_scrape_collector_duration_seconds{collector="zfs"} 1.8613e-05
# HELP node_scrape_collector_success node_exporter: Whether a collector succeeded.
# TYPE node_scrape_collector_success gauge
node_scrape_collector_success{collector="arp"} 1
node_scrape_collector_success{collector="bcache"} 1
node_scrape_collector_success{collector="bonding"} 0
node_scrape_collector_success{collector="btrfs"} 1
node_scrape_collector_success{collector="conntrack"} 0
node_scrape_collector_success{collector="cpu"} 1
node_scrape_collector_success{collector="cpufreq"} 1
node_scrape_collector_success{collector="diskstats"} 1
node_scrape_collector_success{collector="dmi"} 1
node_scrape_collector_success{collector="edac"} 1
node_scrape_collector_success{collector="entropy"} 1
node_scrape_collector_success{collector="fibrechannel"} 0
node_scrape_collector_success{collector="filefd"} 1
node_scrape_collector_success{collector="filesystem"} 1
node_scrape_collector_success{collector="hwmon"} 1
node_scrape_collector_success{collector="infiniband"} 0
node_scrape_collector_success{collector="ipvs"} 0
node_scrape_collector_success{collector="loadavg"} 1
node_scrape_collector_success{collector="mdadm"} 1
node_scrape_collector_success{collector="meminfo"} 1
node_scrape_collector_success{collector="netclass"} 1
node_scrape_collector_success{collector="netdev"} 1
node_scrape_collector_success{collector="netstat"} 1
node_scrape_collector_success{collector="nfs"} 0
node_scrape_collector_success{collector="nfsd"} 0
node_scrape_collector_success{collector="nvme"} 0
node_scrape_collector_success{collector="os"} 0
node_scrape_collector_success{collector="powersupplyclass"} 1
node_scrape_collector_success{collector="pressure"} 1
node_scrape_collector_success{collector="rapl"} 1
node_scrape_collector_success{collector="schedstat"} 1
node_scrape_collector_success{collector="selinux"} 1
node_scrape_collector_success{collector="sockstat"} 1
node_scrape_collector_success{collector="softnet"} 1
node_scrape_collector_success{collector="stat"} 1
node_scrape_collector_success{collector="tapestats"} 0
node_scrape_collector_success{collector="textfile"} 1
node_scrape_collector_success{collector="thermal_zone"} 1
node_scrape_collector_success{collector="time"} 1
node_scrape_collector_success{collector="timex"} 1
node_scrape_collector_success{collector="udp_queues"} 1
node_scrape_collector_success{collector="uname"} 1
node_scrape_collector_success{collector="vmstat"} 1
node_scrape_collector_success{collector="xfs"} 1
node_scrape_collector_success{collector="zfs"} 0
# HELP node_selinux_enabled SELinux is enabled, 1 is true, 0 is false
# TYPE node_selinux_enabled gauge
node_selinux_enabled 0
# HELP node_sockstat_FRAG6_inuse Number of FRAG6 sockets in state inuse.
# TYPE node_sockstat_FRAG6_inuse gauge
node_sockstat_FRAG6_inuse 0
# HELP node_sockstat_FRAG6_memory Number of FRAG6 sockets in state memory.
# TYPE node_sockstat_FRAG6_memory gauge
node_sockstat_FRAG6_memory 0
# HELP node_sockstat_FRAG_inuse Number of FRAG sockets in state inuse.
# TYPE node_sockstat_FRAG_inuse gauge
node_sockstat_FRAG_inuse 0
# HELP node_sockstat_FRAG_memory Number of FRAG sockets in state memory.
# TYPE node_sockstat_FRAG_memory gauge
node_sockstat_FRAG_memory 0
# HELP node_sockstat_RAW6_inuse Number of RAW6 sockets in state inuse.
# TYPE node_sockstat_RAW6_inuse gauge
node_sockstat_RAW6_inuse 0
# HELP node_sockstat_RAW_inuse Number of RAW sockets in state inuse.
# TYPE node_sockstat_RAW_inuse gauge
node_sockstat_RAW_inuse 0
# HELP node_sockstat_TCP6_inuse Number of TCP6 sockets in state inuse.
# TYPE node_sockstat_TCP6_inuse gauge
node_sockstat_TCP6_inuse 3
# HELP node_sockstat_TCP_alloc Number of TCP sockets in state alloc.
# TYPE node_sockstat_TCP_alloc gauge
node_sockstat_TCP_alloc 245
# HELP node_sockstat_TCP_inuse Number of TCP sockets in state inuse.
# TYPE node_sockstat_TCP_inuse gauge
node_sockstat_TCP_inuse 2
# HELP node_sockstat_TCP_mem Number of TCP sockets in state mem.
# TYPE node_sockstat_TCP_mem gauge
node_sockstat_TCP_mem 15
# HELP node_sockstat_TCP_mem_bytes Number of TCP sockets in state mem_bytes.
# TYPE node_sockstat_TCP_mem_bytes gauge
node_sockstat_TCP_mem_bytes 61440
# HELP node_sockstat_TCP_orphan Number of TCP sockets in state orphan.
# TYPE node_sockstat_TCP_orphan gauge
node_sockstat_TCP_orphan 0
# HELP node_sockstat_TCP_tw Number of TCP sockets in state tw.
# TYPE node_sockstat_TCP_tw gauge
node_sockstat_TCP_tw 0
# HELP node_sockstat_UDP6_inuse Number of UDP6 sockets in state inuse.
# TYPE node_sockstat_UDP6_inuse gauge
node_sockstat_UDP6_inuse 0
# HELP node_sockstat_UDPLITE6_inuse Number of UDPLITE6 sockets in state inuse.
# TYPE node_sockstat_UDPLITE6_inuse gauge
node_sockstat_UDPLITE6_inuse 0
# HELP node_sockstat_UDPLITE_inuse Number of UDPLITE sockets in state inuse.
# TYPE node_sockstat_UDPLITE_inuse gauge
node_sockstat_UDPLITE_inuse 0
# HELP node_sockstat_UDP_inuse Number of UDP sockets in state inuse.
# TYPE node_sockstat_UDP_inuse gauge
node_sockstat_UDP_inuse 0
# HELP node_sockstat_UDP_mem Number of UDP sockets in state mem.
# TYPE node_sockstat_UDP_mem gauge
node_sockstat_UDP_mem 11
# HELP node_sockstat_UDP_mem_bytes Number of UDP sockets in state mem_bytes.
# TYPE node_sockstat_UDP_mem_bytes gauge
node_sockstat_UDP_mem_bytes 45056
# HELP node_sockstat_sockets_used Number of IPv4 sockets in use.
# TYPE node_sockstat_sockets_used gauge
node_sockstat_sockets_used 10
# HELP node_softnet_backlog_len Softnet backlog status
# TYPE node_softnet_backlog_len gauge
node_softnet_backlog_len{cpu="0"} 0
node_softnet_backlog_len{cpu="1"} 0
# HELP node_softnet_cpu_collision_total Number of collision occur while obtaining device lock while transmitting
# TYPE node_softnet_cpu_collision_total counter
node_softnet_cpu_collision_total{cpu="0"} 0
node_softnet_cpu_collision_total{cpu="1"} 0
# HELP node_softnet_dropped_total Number of dropped packets
# TYPE node_softnet_dropped_total counter
node_softnet_dropped_total{cpu="0"} 0
node_softnet_dropped_total{cpu="1"} 0
# HELP node_softnet_flow_limit_count_total Number of times flow limit has been reached
# TYPE node_softnet_flow_limit_count_total counter
node_softnet_flow_limit_count_total{cpu="0"} 0
node_softnet_flow_limit_count_total{cpu="1"} 0
# HELP node_softnet_processed_total Number of processed packets
# TYPE node_softnet_processed_total counter
node_softnet_processed_total{cpu="0"} 690229
node_softnet_processed_total{cpu="1"} 615010
# HELP node_softnet_received_rps_total Number of times cpu woken up received_rps
# TYPE node_softnet_received_rps_total counter
node_softnet_received_rps_total{cpu="0"} 0
node_softnet_received_rps_total{cpu="1"} 0
# HELP node_softnet_times_squeezed_total Number of times processing packets ran out of quota
# TYPE node_softnet_times_squeezed_total counter
node_softnet_times_squeezed_total{cpu="0"} 5
node_softnet_times_squeezed_total{cpu="1"} 2
# HELP node_textfile_scrape_error 1 if there was an error opening or reading a file, 0 otherwise
# TYPE node_textfile_scrape_error gauge
node_textfile_scrape_error 0
# HELP node_time_clocksource_available_info Available clocksources read from '/sys/devices/system/clocksource'.
# TYPE node_time_clocksource_available_info gauge
node_time_clocksource_available_info{clocksource="acpi_pm",device="0"} 1
node_time_clocksource_available_info{clocksource="hpet",device="0"} 1
node_time_clocksource_available_info{clocksource="tsc",device="0"} 1
# HELP node_time_clocksource_current_info Current clocksource read from '/sys/devices/system/clocksource'.
# TYPE node_time_clocksource_current_info gauge
node_time_clocksource_current_info{clocksource="tsc",device="0"} 1
# HELP node_time_seconds System time in seconds since epoch (1970).
# TYPE node_time_seconds gauge
node_time_seconds 1.6969479165127072e+09
# HELP node_time_zone_offset_seconds System time zone offset in seconds.
# TYPE node_time_zone_offset_seconds gauge
node_time_zone_offset_seconds{time_zone="UTC"} 0
# HELP node_timex_estimated_error_seconds Estimated error in seconds.
# TYPE node_timex_estimated_error_seconds gauge
node_timex_estimated_error_seconds 0
# HELP node_timex_frequency_adjustment_ratio Local clock frequency adjustment.
# TYPE node_timex_frequency_adjustment_ratio gauge
node_timex_frequency_adjustment_ratio 0.9999028048553467
# HELP node_timex_loop_time_constant Phase-locked loop time constant.
# TYPE node_timex_loop_time_constant gauge
node_timex_loop_time_constant 5
# HELP node_timex_maxerror_seconds Maximum error in seconds.
# TYPE node_timex_maxerror_seconds gauge
node_timex_maxerror_seconds 0.1865
# HELP node_timex_offset_seconds Time offset in between local system and reference clock.
# TYPE node_timex_offset_seconds gauge
node_timex_offset_seconds -0.009672327
# HELP node_timex_pps_calibration_total Pulse per second count of calibration intervals.
# TYPE node_timex_pps_calibration_total counter
node_timex_pps_calibration_total 0
# HELP node_timex_pps_error_total Pulse per second count of calibration errors.
# TYPE node_timex_pps_error_total counter
node_timex_pps_error_total 0
# HELP node_timex_pps_frequency_hertz Pulse per second frequency.
# TYPE node_timex_pps_frequency_hertz gauge
node_timex_pps_frequency_hertz 0
# HELP node_timex_pps_jitter_seconds Pulse per second jitter.
# TYPE node_timex_pps_jitter_seconds gauge
node_timex_pps_jitter_seconds 0
# HELP node_timex_pps_jitter_total Pulse per second count of jitter limit exceeded events.
# TYPE node_timex_pps_jitter_total counter
node_timex_pps_jitter_total 0
# HELP node_timex_pps_shift_seconds Pulse per second interval duration.
# TYPE node_timex_pps_shift_seconds gauge
node_timex_pps_shift_seconds 0
# HELP node_timex_pps_stability_exceeded_total Pulse per second count of stability limit exceeded events.
# TYPE node_timex_pps_stability_exceeded_total counter
node_timex_pps_stability_exceeded_total 0
# HELP node_timex_pps_stability_hertz Pulse per second stability, average of recent frequency changes.
# TYPE node_timex_pps_stability_hertz gauge
node_timex_pps_stability_hertz 0
# HELP node_timex_status Value of the status array bits.
# TYPE node_timex_status gauge
node_timex_status 8193
# HELP node_timex_sync_status Is clock synchronized to a reliable server (1 = yes, 0 = no).
# TYPE node_timex_sync_status gauge
node_timex_sync_status 1
# HELP node_timex_tai_offset_seconds International Atomic Time (TAI) offset.
# TYPE node_timex_tai_offset_seconds gauge
node_timex_tai_offset_seconds 0
# HELP node_timex_tick_seconds Seconds between clock ticks.
# TYPE node_timex_tick_seconds gauge
node_timex_tick_seconds 0.01
# HELP node_udp_queues Number of allocated memory in the kernel for UDP datagrams in bytes.
# TYPE node_udp_queues gauge
node_udp_queues{ip="v4",queue="rx"} 0
node_udp_queues{ip="v4",queue="tx"} 0
node_udp_queues{ip="v6",queue="rx"} 0
node_udp_queues{ip="v6",queue="tx"} 0
# HELP node_uname_info Labeled system information as provided by the uname system call.
# TYPE node_uname_info gauge
node_uname_info{domainname="(none)",machine="x86_64",nodename="node-exporter-bcm7c",release="5.15.0-84-
generic",sysname="Linux",version="#93~20.04.1-Ubuntu SMP Wed Sep 6 16:15:40 UTC 2023"} 1
# HELP node_vmstat_oom_kill /proc/vmstat information field oom_kill.
# TYPE node_vmstat_oom_kill untyped
node_vmstat_oom_kill 0
# HELP node_vmstat_pgfault /proc/vmstat information field pgfault.
# TYPE node_vmstat_pgfault untyped
node_vmstat_pgfault 2.7265115e+07
# HELP node_vmstat_pgmajfault /proc/vmstat information field pgmajfault.
# TYPE node_vmstat_pgmajfault untyped
node_vmstat_pgmajfault 81446
# HELP node_vmstat_pgpgin /proc/vmstat information field pgpgin.
# TYPE node_vmstat_pgpgin untyped
node_vmstat_pgpgin 7.874002e+06
# HELP node_vmstat_pgpgout /proc/vmstat information field pgpgout.
# TYPE node_vmstat_pgpgout untyped
node_vmstat_pgpgout 5.185105e+06
# HELP node_vmstat_pswpin /proc/vmstat information field pswpin.
# TYPE node_vmstat_pswpin untyped
node_vmstat_pswpin 75396
# HELP node_vmstat_pswpout /proc/vmstat information field pswpout.
# TYPE node_vmstat_pswpout untyped
node_vmstat_pswpout 513453
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.06
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 10
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 7.360512e+06
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.69694501138e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 7.41941248e+08
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes 1.8446744073709552e+19
# HELP promhttp_metric_handler_errors_total Total number of internal errors encountered by the promhttp metric handler.
# TYPE promhttp_metric_handler_errors_total counter
promhttp_metric_handler_errors_total{cause="encoding"} 0
promhttp_metric_handler_errors_total{cause="gathering"} 0
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 0
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```
**Note:** We can see our prometheus node-exporter reports so:

- `node_memory_MemTotal_bytes 4.064763904e+09` => This is the total memory available on the system.
- `node_memory_MemFree_bytes 1.31670016e+08` => This is the total free memory available on the system.

In this way, all the node-exporter pods created by the DaemonSet pull all the telemetry that Linux provides by default 
from every node. If a new node is added, the DaemonSet runs the same pod on that new node, allowing us to get the metrics
of the new node as well. These metrics can be given to Prometheus to scrape, which can then be visualized in Grafana.

- When a kubernetes cluster is created Kube-Proxy daemonset is created by default.

### 13.To verify the daemonsets in all namespaces.
```
ubuntu@balasenapathi:~$ kubectl get ds -A
NAMESPACE     NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
default       node-exporter   1         1         1       1            1           <none>                   68m
kube-system   kindnet         1         1         1       1            1           <none>                   36m
kube-system   kube-proxy      1         1         1       1            1           kubernetes.io/os=linux   5d21h
```
**Note:** The above is the `kube-proxy` DaemonSet running in the `kube-system` namespace. We can verify if this DaemonSet
is running the pod or not by listing the pods.

### 14.Now list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS        AGE
coredns-5d78c9869d-k9glg                    1/1     Running   4 (138m ago)    5d21h
etcd-minikube                               1/1     Running   4 (138m ago)    5d21h
kindnet-zstbk                               1/1     Running   0               39m
kube-apiserver-minikube                     1/1     Running   4 (138m ago)    5d21h
kube-controller-manager-minikube            1/1     Running   5 (138m ago)    5d21h
kube-proxy-m7mpq                            1/1     Running   4 (138m ago)    5d21h ===> pod is running
kube-scheduler-minikube                     1/1     Running   4 (138m ago)    5d21h
metrics-server-844d8db974-j67w4             1/1     Running   7 (14h ago)     5d21h
storage-provisioner                         1/1     Running   11 (136m ago)   5d21h
vpa-admission-controller-754ccfdf99-2fd6t   1/1     Running   4 (14h ago)     5d20h
vpa-recommender-667f9769fb-lsfkd            1/1     Running   4 (14h ago)     5d20h
vpa-updater-696b8787f9-rt95f                1/1     Running   4 (138m ago)    5d20h
```
**Note:** As we can see, one `kube-proxy-m7mpq` pod is running in the `kube-system` namespace, which is created by the 
`kube-proxy` DaemonSet. The `kube-proxy` is a component of the Kubernetes cluster responsible for implementing the 
Kubernetes service abstraction. `kube-proxy` runs on every node and maintains network rules on each node to allow network
communication to pods and services running on that node.

### 15.When ever we delete a daemonset all pods managed by it are automatically deleted.We can verify that by using.
```
ubuntu@balasenapathi:~$ kubectl delete ds node-exporter
daemonset.apps "node-exporter" deleted
```
### 16.To get the list of pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
No resources found in deafult namespace
```
**Note:** However, if we don't want to delete the pods that are managed by the DaemonSet, we can set `cascade=false` when
deleting the DaemonSet.

### 17.Deleting a DaemonSet Without Deleting Managed Pods (Deprecated Method).
```
ubuntu@balasenapathi:~$ kubectl delete ds node-exporter --cascade=false
warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
daemonset.apps "node-exporter" deleted
```
### 18.Deleting a DaemonSet Without Deleting Managed Pods (Updated Method).
```
ubuntu@balasenapathi:~$ kubectl delete ds node-exporter --cascade=orphan
daemonset.apps "node-exporter" deleted
```
This way, whenever we delete the DaemonSet, the pods managed by the DaemonSet will not be deleted by using `--cascade=orphan`.



