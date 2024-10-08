# Kubernetes Introduction and Setup

## Kubernetes
Kubernetes is an open-source container orchestration platform developed by Google and now maintained 
by the Cloud Native Computing Foundation (CNCF). It automates the deployment, scaling, and management 
of containerized applications, which are lightweight, portable, and efficient software units that include 
everything needed to run a piece of software, including the code, runtime, system tools, and libraries.

Kubernetes, often abbreviated as K8s, provides a framework for running distributed systems resiliently. 
It manages clusters of virtual machines and schedules containers to run on those machines based on their 
resource requirements and availability. Key features include automatic bin-packing, self-healing through 
auto-restarting, replicating, and scaling containers, service discovery, load balancing, and automated 
rollouts and rollbacks.

K8s abstracts the underlying hardware of nodes (servers) and provides a uniform interface for managing 
the infrastructure. Developers can define the desired state of their applications using declarative 
configurations, and Kubernetes ensures that the current state matches the desired state.

Its extensibility, robust community support, and compatibility with various cloud platforms 
(public, private, and hybrid clouds) make Kubernetes a preferred choice for organizations aiming 
to modernize their application infrastructure and adopt microservices architectures.

## Need of Kubernetes
We need Kubernetes to automate the deployment, scaling, and management of containerized applications. It 
ensures high availability and resilience through self-healing, optimizes resource utilization, and scales 
applications automatically. Kubernetes provides portability across various environments, supports consistent,
declarative configurations, and includes built-in service discovery and load balancing. Additionally, it 
facilitates seamless rolling updates and rollbacks, ensuring application stability, and integrates with a 
broad ecosystem of tools, enhancing its functionality and adaptability to diverse needs.

## Features of Kubernetes
1. **Automatic Bin Packing**:
   Kubernetes efficiently schedules containers based on their resource requirements and available resources. 
   It considers the CPU and memory needs of containers and places them on the most appropriate nodes to 
   optimize resource utilization and avoid overloading any single node.

2. **Service Discovery**:
   Kubernetes provides service discovery mechanisms, allowing containers to find each other without needing 
   to configure specific IP addresses. Services in Kubernetes get assigned a DNS name, and containers can 
   communicate with each other using these names. This simplifies communication between different components 
   of an application.

3. **Load Balancing**:
   Kubernetes can distribute network traffic across multiple pods, ensuring even load distribution. It 
   provides both internal load balancing to distribute traffic between pods within the cluster and external 
   load balancing to manage inbound traffic from outside the cluster, enhancing application performance and 
   reliability.

4. **Storage Orchestration**:
   Kubernetes allows automatic mounting of various storage systems, such as local storage, cloud-based 
   storage, and network-attached storage. It abstracts the underlying storage details and provides a 
   consistent way to manage storage for applications. Persistent Volumes (PVs) and Persistent Volume Claims
   (PVCs) are used to manage storage in Kubernetes.

5. **Self-Healing**:
   Kubernetes continuously monitors the health of nodes and containers. If a container or node fails, 
   Kubernetes automatically restarts the container, reschedules it on a healthy node, or replaces it, 
   ensuring minimal downtime and high availability of applications.

6. **Automated Rollouts and Rollbacks**:
   Kubernetes facilitates automated rollouts of application updates and configuration changes. It gradually 
   updates the application and monitors its health during the process. If any issue is detected, Kubernetes 
   can automatically rollback the changes to the previous stable state, ensuring application stability and 
   minimizing the impact of faulty updates.

7. **Secret and Configuration Management**:
   Kubernetes securely manages sensitive information such as passwords, tokens, and keys using Secrets. 
   ConfigMaps are used to manage configuration data separately from application code. This approach enhances 
   security and flexibility by allowing sensitive information and configurations to be updated without 
   rebuilding container images.

8. **Batch Execution**:
   Kubernetes supports batch and scheduled jobs, managing their execution and ensuring they complete 
   successfully. CronJobs in Kubernetes allow the scheduling of jobs to run at specific times, similar to 
   cron jobs in Unix-like systems, making it suitable for periodic tasks and batch processing workloads.

9. **Auto Scaling**:
   Kubernetes can automatically scale applications up or down based on resource utilization or custom metrics.
   This includes:

      1.**Horizontal Pod Autoscaler (HPA)**: Adjusts the number of pod replicas based on CPU usage, memory 
        usage, or other custom metrics, ensuring the application can handle varying loads.
   
      2.**Vertical Pod Autoscaler (VPA)**: Adjusts the resource limits and requests for containers to optimize 
        resource usage and maintain performance.
   
      3.**Cluster Autoscaler (CA)**: Adjusts the number of nodes in the cluster based on the demands of the 
        workloads.If there are insufficient resources to schedule new pods, the Cluster Autoscaler will add 
        more nodes.Conversely, it will remove underutilized nodes to reduce costs.

These features collectively make Kubernetes a powerful and flexible platform for managing containerized 
applications, providing automation, resilience, and scalability necessary for modern cloud-native 
environments.

## Kubernetes Architecture

![Kubernetes Architecture](https://github.com/balusena/kubernetes-for-devops/blob/main/01-Kubernetes%20Introduction%20and%20Setup/kubernetes_architecture.png)

Kubernetes is an open-source platform designed to manage containerized applications. Its architecture 
is divided into several components and is organized into master (control plane) nodes and worker nodes. 
Below is an overview of these components and their roles:

## Master Node (Control Plane) Components

The master node manages the Kubernetes cluster and includes the following components:

- **API Server (`kube-apiserver`)**
   - **Role:** Serves as the entry point for all API requests.
   - **Function:** Handles CRUD operations for Kubernetes resources and serves as the communication hub between various components.

- **Controller Manager (`kube-controller-manager`)**
   - **Role:** Governs controllers that manage the state of the cluster.
   - **Function:** Ensures the desired state of resources like Deployments, ReplicaSets, and StatefulSets is maintained by performing actions such as scaling pods or replacing failed ones.

- **Scheduler (`kube-scheduler`)**
   - **Role:** Assigns new pods to nodes.
   - **Function:** Determines the most suitable nodes for running pods based on resource availability and constraints.

- **etcd**
   - **Role:** A distributed key-value store for cluster data.
   - **Function:** Stores all cluster state data, including configurations, metadata, and the current state of resources.

- **Cloud Controller Manager (`cloud-controller-manager`)**
   - **Role:** Manages interactions with cloud providers.
   - **Function:** Handles cloud-specific tasks like managing load balancers, volumes, and other resources that are specific to cloud environments.

## Worker Node Components

Worker nodes are responsible for running application workloads. Each worker node includes the following components:

- **Kubelet**
   - **Role:** Manages the state of containers on the node.
   - **Function:** Ensures that containers are running as expected, communicates with the API server, and manages pod lifecycle.

- **Kube-Proxy**
   - **Role:** Manages network communication.
   - **Function:** Maintains network rules on each node and handles service discovery and load balancing for services.

- **Container Runtime**
   - **Role:** Executes containerized applications.
   - **Function:** Handles container creation, execution, and management. Examples include Docker, containerd, and CRI-O.

## Additional Components and Resources

- **Pods**
   - **Role:** The smallest deployable unit in Kubernetes.
   - **Function:** Encapsulates one or more containers, along with storage and networking. Pods are the basic unit of execution.

- **Services**
   - **Role:** Provides a stable network endpoint for accessing pods.
   - **Function:** Abstracts and load-balances access to a set of pods and provides a consistent way to access them.

- **Deployments**
   - **Role:** Manages the deployment and scaling of pods.
   - **Function:** Manages the creation and scaling of pod replicas and facilitates rolling updates and rollbacks.

- **ConfigMaps**
   - **Role:** Manages configuration data.
   - **Function:** Stores configuration data that can be consumed by pods.

- **Secrets**
   - **Role:** Manages sensitive data.
   - **Function:** Stores sensitive information like passwords and API keys securely.

- **Namespaces**
   - **Role:** Provides a mechanism for isolating resources.
   - **Function:** Organizes resources within a cluster into logical groups, often used for separating different environments or teams.

- **Ingress**
   - **Role:** Manages external access to services.
   - **Function:** Provides HTTP and HTTPS routing rules for services and integrates with load balancers or reverse proxies.

- **Volumes**
   - **Role:** Manages persistent storage.
   - **Function:** Provides storage that can be shared and persisted beyond the lifecycle of individual pods.

## Visual Representation

Below is a simplified view of how these components fit together:
```
                            +----------------------+
                            |      Master Node     |
                            |    (Control Plane)   |
                            +----------------------+
                            |  - API Server        |
                            |  - Controller Manager|
                            |  - Scheduler         |
                            |  - etcd              |
                            |  - Cloud Controller  |
                            |    Manager           |
                            +----------------------+
                                       |
                                       |
                +----------------------+------------------+
                |                                         |
   +------------+------------+                +-----------+------------+
   |       Worker Node       |                |      Worker Node       |
   +-------------------------+                +------------------------+
   |  - Kubelet              |                |  - Kubelet             |
   |  - Kube-Proxy           |                |  - Kube-Proxy          |
   |  - Container Runtime    |                |  - Container Runtime   |
   |  - Pods                 |                |  - Pods                |
   |  - Volumes              |                |  - Volumes             |
   +-------------------------+                +------------------------+
                         
```
In this architecture:
- **Master Node** manages the overall cluster and ensures the desired state of the system.
- **Worker Nodes** run the actual applications and manage container execution and networking.

## Kubernetes Setup

Note: I am using Ubuntu, which is installed in VMware.

# Checking KVM Support on Ubuntu 20.04

If you encounter the "kvm-ok: command not found" error when running `sudo kvm-ok`, it likely indicates 
that the `cpu-checker` package, which provides the `kvm-ok` command, is not installed on your system.

## Steps to Enable Virtualization and Check KVM Support

### 1. Enable Virtualization in VMware Workstation
To enable virtualization in your host system, follow these steps:

1. Run VMware Workstation as an administrator.
2. Go to `VM > Settings > Virtual Machine Settings (Hardware) > Processors > Virtualization engine`.
3. Ensure `Virtualize Intel VT-x/EPT or AMD-V/RVI` is checked.

> **Note:** To reflect these changes, please shut down the guest (power off your existing system) and then power on your host machine again.

### 2. Update and Upgrade Your System
Open the terminal on your Ubuntu VM and run the following commands:
```
# 1.Update Your Ubuntu System.

ubuntu@balasenapathi:~$ sudo apt update

# 2.Upgrade Your Ubuntu System.

ubuntu@balasenapathi:~$ sudo apt upgrade
```
### 3. Install cpu-checker
```
# 1.Install the cpu-checker package to get the kvm-ok command.

ubuntu@balasenapathi:~$ sudo apt install cpu-checker
```
### 4.Check KVM Support
```
# 1.Run the following command to check if your system supports KVM.

ubuntu@balasenapathi:~$ sudo kvm-ok
[sudo] password for ubuntu: balasenapathi 
INFO: /dev/kvm exists
KVM acceleration can be used
```
## Install Minikube and Kubectl on Ubuntu 20.04
```
## What You’ll Need?

1. 2 CPUs or more
2. 2GB of free memory
3. 20GB of free disk space
4. Internet connection
5. Container or virtual machine manager, such as:
   - Docker
   - Hyperkit
   - Hyper-V
   - KVM
   - Parallels
   - Podman
   - VirtualBox
   - VMWare
```
### 1.Checking CPU Support and Resources on Ubuntu

```
# 1.Run the following command to check if your system supports KVM.

ubuntu@balasenapathi:~$ sudo kvm-ok
[sudo] password for ubuntu: balasenapathi 
INFO: /dev/kvm exists
KVM acceleration can be used
```

### 2.To Check CPU for Virtualization (VT-x/AMD-V) Support Using the grep Command
```
# 1.Checking CPU for Virtualization (VT-x/AMD-V) Support with grep Command.

ubuntu@balasenapathi:~$ grep -E --color 'vmx|svm' /proc/cpuinfo
flags       : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 syscall nx rdtscp 
lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq vmx ssse3 cx16 
pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx hypervisor lahf_lm epb pti ibrs ibpb stibp tpr_shadow vnmi ept vpid 
tsc_adjust dtherm ida arat pln pts
vmx flags   : vnmi invvpid ept_x_only tsc_offset vtpr mtf ept vpid unrestricted_guest
flags       : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 syscall nx rdtscp 
lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq vmx ssse3 cx16 
pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx hypervisor lahf_lm epb pti ibrs ibpb stibp tpr_shadow vnmi ept vpid 
tsc_adjust dtherm ida arat pln pts
vmx flags   : vnmi invvpid ept_x_only tsc_offset vtpr mtf ept vpid unrestricted_guest
```

### 3.To Display the Number of Processing Units (CPU Cores) Available on the System.
```
# 1.To get Number of CPU Cores available on the system.

ubuntu@balasenapathi:~$ nproc
2
```

## **Single-node Minikube Kubernetes cluster**

### 4.Install the Latest Minikube Stable Release on x86-64 Linux Using Binary Download.

#### 1. Download the Latest Minikube Binary for Linux

This command downloads the latest Minikube binary for Linux (specifically for x86-64 architecture) and 
saves it in the current/present directory.

#### 2.Navigate to the present working directory.
```
ubuntu@balasenapathi:~$ pwd
/home/ubuntu  ====> current/present directory
```
#### 3.Download the Minikube binary:
```
ubuntu@balasenapathi:~$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 82.4M  100 82.4M    0     0  3757k      0  0:00:22  0:00:22 --:--:-- 3689k
```
#### 4.Install the Downloaded Minikube Binary
This command installs the downloaded Minikube binary into the /usr/local/bin directory, making it globally
accessible.
```
ubuntu@balasenapathi:~$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
[sudo] password for ubuntu: balasenapathi
```
#### 5.Configure Minikube to Use the "none" Driver
This command configures Minikube to use the "none" driver for its VM, which means Kubernetes will run 
directly on the host OS without any additional virtualization layer.
```
ubuntu@balasenapathi:~$ sudo minikube config set vm-driver none
❗  These changes will take effect upon a minikube delete and then a minikube start
```
**Note:**
Setting the vm-driver to none allows Minikube to run Kubernetes directly on the host machine without 
using an additional virtualization layer, which can simplify setup and reduce resource overhead. 
Alternatively, you can use drivers like docker, virtualbox, or kvm2 for more isolated and flexible 
environments.

#### 6.Start the minikube cluster.
```
ubuntu@balasenapathi:~$ minikube start
😄  minikube v1.31.1 on Ubuntu 20.04
✨  Automatically selected the virtualbox driver. Other choices: none, ssh
💿  Downloading VM boot image ...
> minikube-v1.31.0-amd64.iso....:  65 B / 65 B [---------] 100.00% ? p/s 0s
> minikube-v1.31.0-amd64.iso:  289.20 MiB / 289.20 MiB  100.00% 3.65 MiB p/
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.27.3 preload ...
> preloaded-images-k8s-v18-v1...:  393.19 MiB / 393.19 MiB  100.00% 3.65 Mi
🔥  Creating virtualbox VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
▪ Generating certificates and keys ...
▪ Booting up control plane ...
▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
E0722 20:46:20.491569    5100 start.go:219] Unable to scale down deployment "coredns" in namespace "kube-system" to 1 replica: non-
retryable failure while rescaling coredns deployment: Operation cannot be fulfilled on deployments.apps "coredns": the object has been
modified; please apply your changes to the latest version and try again
🔎  Verifying Kubernetes components...
▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
### 5.Install the Latest 'kubectl' Stable Release on x86-64 Linux Using Binary Download.

#### 1. Download the Latest Kubectl Binary for Linux
To download the latest release of `kubectl` for Linux (x86-64 architecture), use the following command:

#### 2.Navigate to the present working directory.
```
ubuntu@balasenapathi:~$ pwd
/home/ubuntu  ====> current/present directory
```
#### 3.Download the kubectl binary:
```
ubuntu@balasenapathi:~$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0    633      0 --:--:-- --:--:-- --:--:--   633
100 46.9M  100 46.9M    0     0  3328k      0  0:00:14  0:00:14 --:--:-- 3440k
```
#### 4.Install the Downloaded kubectl Binary
This command installs the downloaded kubectl binary into the /usr/local/bin directory.
```
ubuntu@balasenapathi:~$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
[sudo] password for ubuntu: balasenapathi
```
#### 5.Verify the Installed Version
To ensure the installed version is up-to-date, check the client version:
```
# 1.For a client version.

ubuntu@balasenapathi:~$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:20:54Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
```
```
# 2.For a shorter version of the output:

ubuntu@balasenapathi:~$ kubectl version --short
Client Version: v1.27.4
Kustomize Version: v5.0.1
Server Version: v1.27.3
```
```
# 3.For detailed version information in YAML format:

ubuntu@balasenapathi:~$ kubectl version --output=yaml
clientVersion:
  buildDate: "2023-07-19T12:20:54Z"
  compiler: gc
  gitCommit: fa3d7990104d7c1f16943a67f11b154b71f6a132
  gitTreeState: clean
  gitVersion: v1.27.4
  goVersion: go1.20.6
  major: "1"
  minor: "27"
  platform: linux/amd64
kustomizeVersion: v5.0.1
serverVersion:
  buildDate: "2023-06-14T09:47:40Z"
  compiler: gc
  gitCommit: 25b4e43193bcda6c7328a6d147b1fb73a33f1598
  gitTreeState: clean
  gitVersion: v1.27.3
  goVersion: go1.20.5
  major: "1"
  minor: "27"
  platform: linux/amd64
```
```
# 4.For detailed version information in JSON format:

ubuntu@balasenapathi:~$ kubectl version --output=json
{
  "clientVersion": {
    "major": "1",
    "minor": "27",
    "gitVersion": "v1.27.4",
    "gitCommit": "fa3d7990104d7c1f16943a67f11b154b71f6a132",
    "gitTreeState": "clean",
    "buildDate": "2023-07-19T12:20:54Z",
    "goVersion": "go1.20.6",
    "compiler": "gc",
    "platform": "linux/amd64"
  },
  "kustomizeVersion": "v5.0.1",
  "serverVersion": {
    "major": "1",
    "minor": "27",
    "gitVersion": "v1.27.3",
    "gitCommit": "25b4e43193bcda6c7328a6d147b1fb73a33f1598",
    "gitTreeState": "clean",
    "buildDate": "2023-06-14T09:47:40Z",
    "goVersion": "go1.20.5",
    "compiler": "gc",
    "platform": "linux/amd64"
  }
}
```
### 6.Interact with Your minikube cluster and access your cluster with `kubectl`.

Once `kubectl` is installed, you can interact with your Kubernetes cluster. For example, to list all pods
across all namespaces, run:

```
ubuntu@balasenapathi:~$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-5d78c9869d-lf5lk           1/1     Running   0             19m
kube-system   coredns-5d78c9869d-mwcvs           1/1     Running   0             19m
kube-system   etcd-minikube                      1/1     Running   0             19m
kube-system   kube-apiserver-minikube            1/1     Running   0             19m
kube-system   kube-controller-manager-minikube   1/1     Running   1 (19m ago)   19m
kube-system   kube-proxy-lzknw                   1/1     Running   0             19m
kube-system   kube-scheduler-minikube            1/1     Running   0             19m
kube-system   storage-provisioner                1/1     Running   1 (18m ago)   18m
```
### 7.Access the Kubernetes Dashboard
For additional insight into your cluster's state, Minikube includes the Kubernetes Dashboard. This 
graphical interface helps you get acclimated to your new environment more easily.
```
ubuntu@balasenapathi:~$ minikube dashboard
🔌  Enabling dashboard ...
▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

    minikube addons enable metrics-server

🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:41139/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
Opening in existing browser session.
```
**Note:**
To exit from the Minikube Kubernetes Dashboard, press CTRL+C.

```
# 1.To get information about kubernetes cluster

ubuntu@balasenapathi:~$kubectl cluster-info
Kubernetes control plane is running at http://127.0.0.1:8080
CoreDNS is running at http://127.0.0.1:8080/api/v1/namespaces/kube-system/services/kube-dns:dn
```
```
# 2.To get the information about number of nodes

ubuntu@balasenapathi:~$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   10m     v1.27.3
```

### 8.Stop Your Minikube Cluster
When you are done working with your cluster, you can stop Minikube with:
```
ubuntu@balasenapathi:~$ minikube stop
✋  Stopping node "minikube"  ...
🛑  1 node stopped.
```

## **Multi-node Minikube Kubernetes cluster**

### 1.Start Minikube with Multiple Nodes
```
ubuntu@balasenapathi:~$ minikube start --nodes 2 -p local-cluster --driver=docker
😄  [local-cluster] minikube v1.31.1 on Ubuntu 20.04
✨  Using the docker driver based on user configuration
📌  Using Docker driver with root privileges
👍  Starting control plane node local-cluster in cluster local-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🎉  minikube 1.31.2 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.31.2
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node local-cluster-m02 in cluster local-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.58.2
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.58.2
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "local-cluster" cluster and "default" namespace by default
```
This setup creates a multi-node Kubernetes environment within Docker containers. The control plane and 
worker nodes are managed by Minikube, and kubectl is configured to use the newly created cluster.

### 2.Checking the Status of Minikube with Cluster Name -- local-cluster
```
ubuntu@balasenapathi:~$ minikube status -p local-cluster
local-cluster
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

local-cluster-m02
type: Worker
host: Running
kubelet: Running
```
**Note:**
Here,we are mimicking a multi-node cluster on our local machine using containers.However,in a production
environment,it is not recommended to use Minikube or kind.For production setups,we use individual machines
to create clusters with tools like kubeadm, Rancher Kubernetes Engine, or managed clusters like AWS EKS, 
etc. This local setup is intended for practice on your local laptops.

### 3.To Get the Nodes in the Cluster

```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
local-cluster       Ready    control-plane   98m   v1.27.3
local-cluster-m02   Ready    <none>          93m   v1.27.3
```
**Note:**
Here, we created two nodes, with each node running as a container.

### 4.To view the running Docker containers, use:
```
ubuntu@balasenapathi:~$ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED       STATUS     
PORTS                                                                                                                                  
NAMES
d1440bedba94   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   2 hours ago   Up 2 hours   127.0.0.1:32777->22/tcp, 
127.0.0.1:32776->2376/tcp, 127.0.0.1:32775->5000/tcp, 127.0.0.1:32774->8443/tcp, 127.0.0.1:32773->32443/tcp   local-cluster-m02
8fa42c563600   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   2 hours ago   Up 2 hours   127.0.0.1:32772->22/tcp, 
127.0.0.1:32771->2376/tcp, 127.0.0.1:32770->5000/tcp, 127.0.0.1:32769->8443/tcp, 127.0.0.1:32768->32443/tcp   local-cluster
```
### 5.To List All the Clusters
```
ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER         AUTHINFO        NAMESPACE
*         local-cluster   local-cluster   local-cluster   default
```
**Note:**
The star (*) indicates the active cluster in use, which is local-cluster in this case. You can 
create multiple clusters and switch between them as needed.

### 6.To switch between the cluster we need to use:
```
ubuntu@balasenapathi:~$ kubectl config set-context local-cluster
Context "local-cluster" modified.
```

### 7.To add the one more new node to the cluster:
```
ubuntu@balasenapathi:~$ minikube node add --worker -p local-cluster
😄  Adding node m03 to cluster local-cluster
👍  Starting worker node local-cluster-m03 in cluster local-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
🔎  Verifying Kubernetes components...
🏄  Successfully added m03 to local-cluster!
```

### 8.To run kubernetes dashboard in minikube:
```
ubuntu@balasenapathi:~$ minikube dashboard --url -p local-cluster
🔌  Enabling dashboard ...
▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

	minikube -p local-cluster addons enable metrics-server	


🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:45715/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
**Note:**
To exit from the Minikube Kubernetes Dashboard, press CTRL+C.

```
# 1.To get information about kubernetes cluster

ubuntu@balasenapathi:~$kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:32768
CoreDNS is running at https://127.0.0.1:32768/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
```
# 2.To get the information about number of nodes

ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
local-cluster       Ready    control-plane   98m   v1.27.3
local-cluster-m02   Ready    <none>          93m   v1.27.3
local-cluster-m03   Ready    <none>          92m   v1.27.3
```
### 9.To delete the node from the local-cluster:
```
ubuntu@balasenapathi:~$ minikube node delete local-cluster-m03 -p local-cluster
🔥  Deleting node local-cluster-m03 from cluster local-cluster
✋  Stopping node "local-cluster-m03"  ...
🛑  Powering off "local-cluster-m03" via SSH ...
🔥  Deleting "local-cluster-m03" in docker ...
💀  Node local-cluster-m03 was successfully deleted.
```
### 10.Stop Your Minikube Cluster
When you are done working with your cluster, you can stop Minikube with:
```
ubuntu@balasenapathi:~$ minikube stop
🔥  Stopping node "local-cluster" ...
🔥  Stopping node "local-cluster-m02" ...
```
## Setting Up Kubernetes Environment Variables

To ensure that the `minikube` and `kubectl` commands are available in all terminal sessions, you need to 
add their binary directories to your system's `PATH` environment variable. Follow these steps:

### 1. Find the Paths
Check the locations of the `minikube` and `kubectl` binaries:
```
# 1.Check location for minikube.

ubuntu@balasenapathi:~$ which minikube
/usr/local/bin/minikube
```
```
# 2.Check location for kubectl.

ubuntu@balasenapathi:~$ which kubectl
/usr/local/bin/kubectl
```

### 2.Update User-Specific Environment
Edit the .bashrc file to include the Minikube and Kubectl paths:
```
# 1.Add the following lines to the end of the .bashrc file:

ubuntu@balasenapathi:~$ nano ~/.bashrc
# Minikube, Kubectl path
export PATH=/usr/local/bin:$PATH
```
```
# 2.Save the file and exit the editor. Then, apply the changes:

ubuntu@balasenapathi:~$ source ~/.bashrc
```
### 3.Update System-Wide Environment
If you want to make these changes system-wide, you need to update the /etc/profile file. Open the file 
with root privileges:
```
# 1.Add the following lines to the end of the /etc/profile file.

ubuntu@balasenapathi:~$ sudo nano /etc/profile
# Minikube, Kubectl path
export PATH=/usr/local/bin:$PATH
```
```
# 2.Save the file and exit the editor. Apply the changes:

ubuntu@balasenapathi:~$ . /etc/profile
```
## Running Docker in Minikube

To run Docker inside Minikube, you need to configure your local Docker client to interact with the Docker 
daemon within Minikube. This is achieved using the `eval $(minikube docker-env)` command, which sets up 
the necessary environment variables. This allows you to build, push, and manage Docker images within 
Minikube, seamlessly integrating local development workflows with Kubernetes.

### How to Use Local Docker Images with Minikube

```
# 1.Start the minikube cluster.

ubuntu@balasenapathi:~$ minikube start --kubernetes-version=v1.27.3
😄  minikube v1.31.1 on Ubuntu 20.04
✨  Automatically selected the docker driver. Other choices: virtualbox, ssh
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.27.3 preload ...
    > preloaded-images-k8s-v18-v1...:  393.19 MiB / 393.19 MiB  100.00% 709.31 
    > gcr.io/k8s-minikube/kicbase...:  447.62 MiB / 447.62 MiB  100.00% 672.98 
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
```
# 2.Set Docker Environment, configure your local Docker client to use the Docker daemon inside Minikube.

ubuntu@balasenapathi:~$ eval $(minikube docker-env)
Host added: /home/ubuntu/.ssh/known_hosts ([127.0.0.1]:49157)
```














