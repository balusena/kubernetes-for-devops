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


# 3: Kubernetes Pods

## 1.Introduction to Kubernetes Pods

## 2.Sidecar Containers and Their Use Cases

## 3.Need for a Pod
   - **Atomic Deployment**
   - **Resource Sharing**
   - **Lifecycle Management**

## 4.Scaling Applications with Pods
   - **Manual Scaling**
   - **Horizontal Pod Autoscaler (HPA)**

## 5. **Pod Communication**
   - **Networking**
   - **Services**
   - **Inter-Pod Communication**

### Pod Overview
   - **Kubernetes Pods: Key Points**
     - **Kubernetes Pod**
     - **Container Management**
     - **Multi-container Pods**
     - **Container Runtime Support**
     - **Deployment Unit**
     - **Network and Port Usage**
     - **Resource Management**
     - **Shared Resources**
     - **Pod Spec**
     - **Replication**
     - **Volume Sharing**
     - **Kubectl**

## 6.Creating Pods with kubectl
   - **Creating a nginx pod using kubectl**
   - **To see list of pods in kubernetes cluster**

## 7.Creating Pods with YAML
   - **Clone this repository and move to example folder**
   - **To get the apiVersion of pods in kubernetes**
   - **Create the yaml file nginx-pod.yaml**
   - **Creating a Pod**
   - **Listing the Pods**
   - **Deleting a Pod**

## 8.Filtering Pods
   - **Filtering Pods by Label**
   - **Filtering Pods with Multiple Labels**
   - **Getting Detailed Pod Information**
   - **Getting Pod Information in YAML Format**

## 9.Listing Pods with Details
   - **To describe the pod with detail information about it**

## 10.Getting Into a Pod
   - **Accessing a Single Container Pod**
   - **Accessing a Specific Container in a Multi-Container Pod**

## 11.Port Forwarding
   - **Introduction**
   - **Accessing the Pod from Within the Node**
   - **Accessing the Service**
   - **Stopping Port Forwarding**

## 12.Checking Pod Logs
   - **Log Analysis**
      - **Initialization Logs**
      - **Access Logs**
      - **Error Logs**

## 13.Deleting Pods
   - **Deleting Pods Using a YAML File**
      - **Checking the remaining pods in the cluster**
   - **Deleting Pods Created with kubectl**
      - **Incorrect Command**
      - **Correct Command**

## 14.Verifying Deletion
   - **To ensure the pod has been deleted, you can run**
   

# 4: Kubernetes Nodes

## 1.Types of Nodes
   - **Master Node**
     - **Purpose** 
     - **Components**
       - **API Server**
       - **Controller Manager**
       - **Scheduler**
       - **etcd**

   - **Worker Node**
     - **Purpose**
     - **Components**
       - **Kubelet**
       - **Kube-Proxy**
       - **Container Runtime**

## 2.Node Lifecycle
   - **Ready**
   - **NotReady**
   - **Unknown**

## 3.Node Management**
   - **Scaling**
   - **Health Monitoring**
   - **Maintenance**

## 4.Node Overview

## 5.Comprehensive List of Commands
   - **To view nodes in a cluster**
   - **To get detailed information about a specific node**
   - **Cordon a Node - To mark the node as unschedulable**
   - **Uncordon a Node - To mark the node as schedulable**
   - **Drain a Node - To safely evict all pods from the node before marking it unschedulable**
   - **Label a Node - To add a label to the node**
   - **Remove a Node Label - To remove a label from the node**
   - **Taint a Node - To add a taint to the node and prevent pods from being scheduled unless they tolerate the taint**
   - **Remove a Node Taint - To remove a taint from the node**
   

# 5: Kubernetes ReplicaSets and Deployments

## 1.ReplicaSets
   - **Replicaset Features**

   - **Creating ReplicaSets with YAML**
     - **Clone this repository and move to example folder**
     - **To get the apiVersion of replicasets in kubernetes**
     - **Create a replicaset using file nginx-replicaset.yaml**
     - **To create the replicaset in kubernetes cluster**
     - **To check whether replicaset is created in kubernetes cluster**
     - **To get the list of pods running in the kubernetes cluster**

## 2.Self-Healing in Kubernetes
   - **To delete a pod from the Kubernetes cluster**

## 3.High Availability in Kubernetes
   - **List of Nodes in the Kubernetes Cluster**
   - **Adding a Node to the Cluster**
   - **List of Nodes After Adding the New Node**
   - **Deleting a Node and Observing Pod Behavior**
   - **Deleting a Node from the Cluster**
   - **List of Pods After Deleting a Node**

## 4.Deployment

## 5.Rolling Update and Rollback in Deployment
   - **Rolling Updates**
   - **Rollback**
   - **Deployment with Replicaset**

- **Rolling Updates**
   - **Deleting All Objects and Services from Kubernetes Cluster**
   - **Confirming Deletion**
   - **To get the apiVersion of deployments in kubernetes**
   - **Create a deployment using file nginx-deployment.yaml**
   - **To create the deployment in kubernetes cluster**
   - **To check whether replicaset is created in kubernetes cluster**
   - **To get the list of all resources in kubernetes cluster**
   - **Verifying Pod Labels in Kubernetes Cluster**
   - **Scaling an Application in Kubernetes** 
   - **Listing All Pods before applying scaling**
   - **Scaling an Application Using `kubectl`**
   - **Listing All Pods after applying scaling using kubectl**
   - **Changing the version from 1.21.3 to 1.21 using kubectl command without using deployment file**
   - **Now try to list down the resources**
   - **Inspecting Pod Details**
   - **Viewing Rollout History of a Deployment**
   - **Now change the image from version 1.21 to version 1.20 using --record**
   - **To get the history of revisions done by rollouts**
   - **Attempting to Create the Deployment**
   - **To check the available namespaces**
   - **Creating the Namespace**
   - **Reapply nginx deployment configuration, after creating the namespace**
   - **Viewing Rollout History of Revisions**

- **Rollback**
   - **To rollback the deployment to previous versions**
   - **To see the rollout deployment status using kubectl rollout status**
   - **To verify it was rolled back to the nginx:latest version, list all the pods and check the pod image**
   - **To get the complete information about a pod**

## 6.Summary





