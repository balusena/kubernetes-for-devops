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


# 6: kubernetes Services

## 1.Types of Services
1.**ClusterIP**
2.**Multiport**
3.**NodePort**
4.**LoadBalancer**

## 2.Benefits

## 3.How Kubernetes Services Works.
1.**Use case-1**
  - **Kubernetes Services Workflow**
  - **Other advantages of Kubernetes Services**
    - **Load balancing** 
    - **Service discovery** 
    - **Zero downtime deployments** 

2.**Use case-2:**
  - **The different Service types in Kubernetes are**
    - **ClusterIP Service**
    - **Multi-Port Service**
    - **NodePort Service**
    - **LoadBalancer Service**
  
    - **To get the apiVersion of services in kubernetes**
    - **Create a service using file nginx-service.yaml**

3.**ClusterIP-Service**

## 4.Understanding Services
- **cluster-ip-service.yaml**
   - **Now apply the changes to the services into kubernetes**
   - **Now list down all the services**
      - **Verifying External Access**
   - **This service is not accessible from outside of the cluster lets verify**
      - **Verifying Internal Access:**
   - **This service can be accessible from any Pods with in the cluster, verify this by getting into any of the Pods in cluster**
      - **To get the list of all Pods Running in the cluster**
      - **Accessing ClusterIP Service using IP address of the service**
         - **Using kubectl exec to enter one of the pods and trying to access the service internally**
         - **Accessing ClusterIP Service using name of the service**
         - **Accessing Services Using Port-Forwarding**
   - **Port-Forwarding a Service**
   - **Verifying in a Web Browser**
      - **Now, go to your web browser and check the following URL**
      - **Services Offers Load balancing**
   - **Testing Load Balancing with Kubernetes Services**
      - **To get the list of all Pods Running in the cluster**
      - **To get into the pod with sh**
      - **Automating Load Testing with Shell Script**
      - **Shell Script for Load Testing**
         - **You can use the following shell script to automate the load testing**
      - **Verifying Load Balancing by Monitoring Pod Logs**
         - **Monitoring Pod Logs**
         - **Logs of Pod 1**
         - **Logs of Pod 2** 
         - **Running the Shell Script**
            - **Access the Pod**
            - **Run the Shell Script**
            - **Script Output**
            - **Pod Logs and Load Balancing Verification**
               - **Pod 1 Logs**
               - **Pod 2 Logs**
   - **Pods Associated with Services**
      - **To see what pods are associated with the services**
      - **We can also check which pods are associated with a service by describing the service**
         - **Checking Pods and Endpoints**
            - **Option 1:Scaling Using kubectl scale**              
               - **To set the number of replicas to 3**
            - **Option 2:Editing the Deployment YAML**        
               - **Edit the deployment YAML file directly to change the number of replicas**

4. **2.Multi-Port Service**
    - **multi-port-service.yaml**
    - **With this configuration**

5. **3.NodePort Service**
    - **Important Points**   
       - **NodePort Range**
       - **Automatic Port Assignment**
       - **Port Handling**
    - **nginx-service.yaml**
      - **In this configuration**
      - **Apply the changes of the service in the kubernetes cluster**
      - **List down all the services in the cluster**
      - **To get the IP address in the minikube of my local-cluster**

6. **4.LoadBalancer Service**
    - **nginx-service.yaml**
    - **In this configuration**
       - **type**  
       - **port**
       - **targetPort**
       - **nodePort**

    - **Apply the changes of the service in the kubernetes cluster**
    - **List down all the services in the cluster**
    - **Accessing the LoadBalancer Service**
    
## 5.Summary


# 7:  kubernetes Ingress

## 1.What is Kubernetes Ingress?
- **Note:The AWS, Azure, GCP cloud ingress controller implementation differs a little**
   - **AWS** 
   - **Azure** 
   - **GCP** 
   
## 2.How Does Kubernetes Ingress work?
- **Kubernetes Ingress Resource** 
- **2.Kubernetes Ingress Controller** 

## 3.Ingress & Ingress Controller Architecture

## 4.Benefits of Using Kubernetes Ingress
- **Routing flexibility**
- **Load balancing** 
- **TLS termination** 
- **Namespace isolation** 

- **Create a new cluster in minikube with name ingress-cluster**
- **We are using this nginx-deployment file in new cluster for our deployment**
- **We are using below nginx-service file to create service in the cluster**
- **To get the information about all services running in the cluster**
- **To get the information about all pods,services,deploments,replicasets**

## 5.Setting Up Kubernetes Ingress and Ingress controller
- **Create a new cluster in minikube with name ingress-cluster**
- **Setup the Ingress Controller**
- **To deploy nginx-ingress-controller in minikube**
- **To verify whether nginx-ingress-controller is enabled or not**
- **ingress-controller is a pod and it gets exposed through a service**
   - **Ingress Rules**
- **To get the apiVersion of ingress in kubernetes**
- **Create the Ingress Resource YAML File**
- **Ingress-Rule**
- **Path Type**
   - **ImplementationSpecific** 
   - **Exact**
   - **Prefix**
- **Apply the changes to the ingress-cluster to implement the ingress-rule**
- **We can verify that, it was created with apply nginx-ingress.yaml file**
- **Now go to the browser and check whether we can able to access the nginx-demo.com**
- **To we can get the minikube ip by using this**
- **Now we should map the minikube ip address to our nginx-demo.com in our machine,this can be done by editing below**
- **Now go to the browser and check whether we can able to access the nginx-demo.com**
- **Now extend this ingress to access our to-do ui and api applications**
- **Apply the changes in the ingress-cluster**

## 6.Now we will access these two applications using ingress
- **We will be accessing this application in two ways of routing**
   - **Path-Based Routing/Mapping:**
   - **Host-Based Routing/Mapping:**

- **Path-Based Routing/Mapping**
   - **Create the todo-ingress-path-based.yaml file**
   - **Explanation**
   - **Apply the Ingress Configuration**
   - **We can verify that it was created with**
   - **Map the Minikube IP Address to todo.com**
   - **Access the Applications in the Browser**

- **Host-Based Routing/Mapping**
   - Create the file todo-ingress-host-based.yaml**
   - **Explanation**
   - **Apply the Ingress Configuration**
   - **We can verify that it was created with**
   - **Map the Minikube IP Address to Our Hosts**
   - **Access the Applications in the Browser**

- **Default Backend**
   - **To illustrate, let's describe the Ingress named nginx-ingress**
   - **Example Configuration for Default Backend**
   - **Create a file named default-backend.yaml**
   - **Apply the configuration**

## 7.Securing Application with HTTPS using TLS Certificates in Ingress
- **Step 1:Generate a Self-Signed Certificate**
- **Step 2:Create a Kubernetes Secret**
- **Step 3:Update Ingress Resource with TLS Secret**
- **step 4:Apply the Changes**
   - **Apply the updated Ingress resource to the cluster**
   - **Go tho browser and access your nginx-demo.com with https**








