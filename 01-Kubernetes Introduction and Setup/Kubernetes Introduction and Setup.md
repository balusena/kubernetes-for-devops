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







