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

#### 6. Start the minikube cluster.
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














