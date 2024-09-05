# kubernetes-for-devops
This repository covers the complete Kubernetes fundamentals along with examples required for a DevOps Engineer.

# 1: Kubernetes Introduction and Setup

## 1.Kubernetes

## 2.Need of Kubernetes

## 3.Features of Kubernetes
1. **Automatic Bin Packing**
2. **Service Discovery**
3. **Load Balancing**
4. **Storage Orchestration**
5. **Self-Healing**
6. **Automated Rollouts and Rollbacks**
7. **Secret and Configuration Management**
8. **Batch Execution**
9. **Auto Scaling**
   - **Horizontal Pod Autoscaler (HPA)**
   - **Vertical Pod Autoscaler (VPA)**
   - **Cluster Autoscaler (CA)**

## 4.Kubernetes Architecture

## 5.Master Node (Control Plane) Components
1. **API Server (`kube-apiserver`)**
2. **Controller Manager (`kube-controller-manager`)**
3. **Scheduler (`kube-scheduler`)**
4. **etcd**
5. **Cloud Controller Manager (`cloud-controller-manager`)**

## 6.Worker Node Components
1. **Kubelet**
2. **Kube-Proxy**
3. **Container Runtime**

## 7.Additional Components and Resources
1. **Pods**
2. **Services**
3. **Deployments**
4. **ConfigMaps**
5. **Secrets**
6. **Namespaces**
7. **Ingress**
8. **Volumes**

## 8.Visual Representation
1. **Master Node**
2. **Worker Nodes**

## 9.Kubernetes Setup
1. **Checking KVM Support on Ubuntu 20.04**
2. **Steps to Enable Virtualization and Check KVM Support**
   - **Enable Virtualization in VMware Workstation**
   - **Update and Upgrade Your System**
   - **Install cpu-checker**
   - **Check KVM Support**
3. **Install Minikube and Kubectl on Ubuntu 20.04**
   - **What You’ll Need?**
4. **Checking CPU Support and Resources on Ubuntu**
   - **To Check CPU for Virtualization (VT-x/AMD-V) Support Using the grep Command**
   - **To Display the Number of Processing Units (CPU Cores) Available on the System**

## 10.Single-node Minikube Kubernetes Cluster
1. **Single-node Minikube Kubernetes Cluster**
2. **Install the Latest Minikube Stable Release on x86-64 Linux Using Binary Download**
   - **Download the Latest Minikube Binary for Linux**
   - **Navigate to the Present Working Directory**
   - **Download the Minikube Binary**
   - **Install the Downloaded Minikube Binary**
   - **Configure Minikube to Use the "none" Driver**
   - **Start the Minikube Cluster**
3. **Install the Latest 'kubectl' Stable Release on x86-64 Linux Using Binary Download**
   - **Download the Latest Kubectl Binary for Linux**
   - **Navigate to the Present Working Directory**
   - **Download the Kubectl Binary**
   - **Install the Downloaded Kubectl Binary**
   - **Verify the Installed Version**
4. **Interact with Your Minikube Cluster and Access Your Cluster with `kubectl`**
5. **Access the Kubernetes Dashboard**
   - **To Get Information About Kubernetes Cluster**
   - **To Get Information About Number of Nodes**
6. **Stop Your Minikube Cluster**

## 11.Multi-node Minikube Kubernetes Cluster
1. **Start Minikube with Multiple Nodes**
2. **Checking the Status of Minikube with Cluster Name `--local-cluster`**
3. **To Get the Nodes in the Cluster**
4. **To View the Running Docker Containers**
5. **To List All the Clusters**
6. **To Switch Between Clusters**
7. **To Add One More New Node to the Cluster**
8. **To Run Kubernetes Dashboard in Minikube**
   - **To Get Information About Kubernetes Cluster**
   - **To Get Information About Number of Nodes**
9. **To Delete the Node from the Local-Cluster**
10. **Stop Your Minikube Cluster**

## 12.Setting Up Kubernetes Environment Variables
1. **Find the Paths**
   - **Check Location for Minikube**
   - **Check Location for kubectl**
2. **Update User-Specific Environment**
   - **Add the Following Lines to the End of the .bashrc File**
   - **Save the File and Exit the Editor. Then, Apply the Changes**
3. **Update System-Wide Environment**
   - **Add the Following Lines to the End of the /etc/profile File**
   - **Save the File and Exit the Editor. Apply the Changes**

## 13.Running Docker in Minikube
1. **How to Use Local Docker Images with Minikube**
   - **Start the Minikube Cluster**
   - **Set Docker Environment: Configure Your Local Docker Client to Use the Docker Daemon Inside Minikube**
   
# 2: Introduction to YAML (YAML Ain't Markup Language)

## 1.Key Features
1. **Human-Readable**
2. **Data Structures**
3. **Indentation-Based**
4. **Support for Comments**
5. **Flexible and Powerful**

## 2.YAML is a Serialization Language
1. **Serialization**
2. **Deserialization**

## 4.Different Serialization Languages
1. **YAML**
2. **JSON**
3. **XML**

## 5.Summary


