# Kubernetes For Devops

![Kubernetes](https://github.com/balusena/kubernetes-for-devops/blob/main/kubernetes.png)

The Kubernetes for DevOps repository offers an extensive guide designed to help DevOps professionals master Kubernetes.
It starts with a foundational introduction to Kubernetes and its setup, ensuring a solid understanding of the platform. 
The guide covers YAML, the language used for defining Kubernetes configurations, and progresses through key Kubernetes 
concepts including Pods, Nodes, ReplicaSets, and Deployments.

It then explores Kubernetes Services for exposing applications, Ingress for managing external access, and Namespaces for
organizing resources. The repository also addresses Volumes for persistent storage and StatefulSets for stateful applications.
ConfigMaps and Secrets are covered for managing configuration and sensitive data, while Probes and Resource Management 
focus on maintaining application health and efficient resource utilization.

Advanced topics include Scheduling, Autoscaling, and Role-Based Access Control (RBAC) for managing resources and access.
DaemonSets, Jobs, and CronJobs are discussed for running background tasks and periodic jobs. Overall, the repository provides
a structured approach to mastering Kubernetes, from basic setup to advanced management and scaling techniques.

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
1. **Atomic Deployment**
2. **Resource Sharing**
3. **Lifecycle Management**

## 4.Scaling Applications with Pods
1. **Manual Scaling**
2. **Horizontal Pod Autoscaler (HPA)**

## 5.Pod Communication
1. **Networking**
2. **Services**
3. **Inter-Pod Communication**

### Pod Overview
1. **Kubernetes Pods: Key Points**
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
1. **Creating a nginx pod using kubectl**
2. **To see list of pods in kubernetes cluster**

## 7.Creating Pods with YAML
1. **Clone this repository and move to example folder**
2. **To get the apiVersion of pods in kubernetes**
3. **Create the yaml file nginx-pod.yaml**
4. **Creating a Pod**
5. **Listing the Pods**
6. **Deleting a Pod**

## 8.Filtering Pods
1. **Filtering Pods by Label**
2. **Filtering Pods with Multiple Labels**
3. **Getting Detailed Pod Information**
4. **Getting Pod Information in YAML Format**

## 9.Listing Pods with Details
1. **To describe the pod with detail information about it**

## 10.Getting Into a Pod
1. **Accessing a Single Container Pod**
2. **Accessing a Specific Container in a Multi-Container Pod**

## 11.Port Forwarding
1. **Introduction**
2. **Accessing the Pod from Within the Node**
3. **Accessing the Service**
4. **Stopping Port Forwarding**

## 12.Checking Pod Logs
1. **Log Analysis**
    - **Initialization Logs**
    - **Access Logs**
    - **Error Logs**

## 13.Deleting Pods
1. **Deleting Pods Using a YAML File**
    - **Checking the remaining pods in the cluster**
2. **Deleting Pods Created with kubectl**
    - **Incorrect Command**
    - **Correct Command**

## 14.Verifying Deletion
1. **To ensure the pod has been deleted, you can run**
   

# 4: Kubernetes Nodes

## 1.Types of Nodes
1. **Master Node**
    - **Purpose** 
    - **Components**
       - **API Server**
       - **Controller Manager**
       - **Scheduler**
       - **etcd**

2.**Worker Node**
   - **Purpose**
   - **Components**
      - **Kubelet**
      - **Kube-Proxy**
      - **Container Runtime**

## 2.Node Lifecycle
1. **Ready**
2. **NotReady**
3. **Unknown**

## 3.Node Management**
1. **Scaling**
2. **Health Monitoring**
3. **Maintenance**

## 4.Node Overview

## 5.Comprehensive List of Commands
1. **To view nodes in a cluster**
2. **To get detailed information about a specific node**
3. **Cordon a Node - To mark the node as unschedulable**
4. **Uncordon a Node - To mark the node as schedulable**
5. **Drain a Node - To safely evict all pods from the node before marking it unschedulable**
6. **Label a Node - To add a label to the node**
7. **Remove a Node Label - To remove a label from the node**
8. **Taint a Node - To add a taint to the node and prevent pods from being scheduled unless they tolerate the taint**
9. **Remove a Node Taint - To remove a taint from the node**
   

# 5: Kubernetes ReplicaSets and Deployments

## 1.ReplicaSets
1. **Replicaset Features**

2. **Creating ReplicaSets with YAML**
    - **Clone this repository and move to example folder**
    - **To get the apiVersion of replicasets in kubernetes**
    - **Create a replicaset using file nginx-replicaset.yaml**
    - **To create the replicaset in kubernetes cluster**
    - **To check whether replicaset is created in kubernetes cluster**
    - **To get the list of pods running in the kubernetes cluster**

## 2.Self-Healing in Kubernetes
1. **To delete a pod from the Kubernetes cluster**

## 3.High Availability in Kubernetes
1. **List of Nodes in the Kubernetes Cluster**
2. **Adding a Node to the Cluster**
3. **List of Nodes After Adding the New Node**
4. **Deleting a Node and Observing Pod Behavior**
5. **Deleting a Node from the Cluster**
6. **List of Pods After Deleting a Node**

## 4.Deployment

## 5.Rolling Update and Rollback in Deployment
1. **Rolling Updates**
2. **Rollback**
3. **Deployment with Replicaset**

1. **Rolling Updates**
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

2. **Rollback**
    - **To rollback the deployment to previous versions**
    - **To see the rollout deployment status using kubectl rollout status**
    - **To verify it was rolled back to the nginx:latest version, list all the pods and check the pod image**
    - **To get the complete information about a pod**

## 6.Summary


# 6: kubernetes Services

## 1.Types of Services
1. **ClusterIP**
2. **Multiport**
3. **NodePort**
4. **LoadBalancer**

## 2.Benefits

## 3.How Kubernetes Services Works.
1. **Use case-1**
    - **Kubernetes Services Workflow**
    - **Other advantages of Kubernetes Services**
       - **Load balancing** 
       - **Service discovery** 
       - **Zero downtime deployments** 

2. **Use case-2:**
    - **The different Service types in Kubernetes are**
       - **ClusterIP Service**
       - **Multi-Port Service**
       - **NodePort Service**
       - **LoadBalancer Service**
  
    - **To get the apiVersion of services in kubernetes**
    - **Create a service using file nginx-service.yaml**

3.**ClusterIP-Service**

## 4.Understanding Services
1. **cluster-ip-service.yaml**
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

2. **Multi-Port Service**
    - **multi-port-service.yaml**
    - **With this configuration**

3. **NodePort Service**
    - **Important Points**   
       - **NodePort Range**
       - **Automatic Port Assignment**
       - **Port Handling**
    - **nginx-service.yaml**
       - **In this configuration**
       - **Apply the changes of the service in the kubernetes cluster**
       - **List down all the services in the cluster**
       - **To get the IP address in the minikube of my local-cluster**

4. **LoadBalancer Service**
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


# 7: kubernetes Ingress

## 1.What is Kubernetes Ingress?
1. **Note:The AWS, Azure, GCP cloud ingress controller implementation differs a little**
    - **AWS** 
    - **Azure** 
    - **GCP** 
   
## 2.How Does Kubernetes Ingress work?
1. **Kubernetes Ingress Resource** 
2. **Kubernetes Ingress Controller** 

## 3.Ingress & Ingress Controller Architecture

## 4.Benefits of Using Kubernetes Ingress
1. **Routing flexibility**
2. **Load balancing** 
3. **TLS termination** 
4. **Namespace isolation** 

5. **Create a new cluster in minikube with name ingress-cluster**
6. **We are using this nginx-deployment file in new cluster for our deployment**
7. **We are using below nginx-service file to create service in the cluster**
8. **To get the information about all services running in the cluster**
9. **To get the information about all pods,services,deploments,replicasets**

## 5.Setting Up Kubernetes Ingress and Ingress controller
1. **Create a new cluster in minikube with name ingress-cluster**
2. **Setup the Ingress Controller**
3. **To deploy nginx-ingress-controller in minikube**
4. **To verify whether nginx-ingress-controller is enabled or not**
5. **ingress-controller is a pod and it gets exposed through a service**
    - **Ingress Rules**
6. **To get the apiVersion of ingress in kubernetes**
7. **Create the Ingress Resource YAML File**
8. **Ingress-Rule**
9. **Path Type**
    - **ImplementationSpecific** 
    - **Exact**
    - **Prefix**
10. **Apply the changes to the ingress-cluster to implement the ingress-rule**
11. **We can verify that, it was created with apply nginx-ingress.yaml file**
13. **Now go to the browser and check whether we can able to access the nginx-demo.com**
14. **To we can get the minikube ip by using this**
15. **Now we should map the minikube ip address to our nginx-demo.com in our machine,this can be done by editing below**
16. **Now go to the browser and check whether we can able to access the nginx-demo.com**
17. **Now extend this ingress to access our to-do ui and api applications**
18. **Apply the changes in the ingress-cluster**

## 6.Now we will access these two applications using ingress
1. **We will be accessing this application in two ways of routing**
    - **Path-Based Routing/Mapping:**
    - **Host-Based Routing/Mapping:**

2. **Path-Based Routing/Mapping**
    - **Create the todo-ingress-path-based.yaml file**
    - **Explanation**
    - **Apply the Ingress Configuration**
    - **We can verify that it was created with**
    - **Map the Minikube IP Address to todo.com**
    - **Access the Applications in the Browser**

3. **Host-Based Routing/Mapping**
    - Create the file todo-ingress-host-based.yaml**
    - **Explanation**
    - **Apply the Ingress Configuration**
    - **We can verify that it was created with**
    - **Map the Minikube IP Address to Our Hosts**
    - **Access the Applications in the Browser**

4. **Default Backend**
    - **To illustrate, let's describe the Ingress named nginx-ingress**
    - **Example Configuration for Default Backend**
    - **Create a file named default-backend.yaml**
    - **Apply the configuration**

## 7.Securing Application with HTTPS using TLS Certificates in Ingress
1. **Step 1:Generate a Self-Signed Certificate**
2. **Step 2:Create a Kubernetes Secret**
3. **Step 3:Update Ingress Resource with TLS Secret**
4. **step 4:Apply the Changes**
    - **Apply the updated Ingress resource to the cluster**
    - **Go tho browser and access your nginx-demo.com with https**


# 8: kubernetes Namespaces

## 1.What is Namespace?

## 2.Need for Namespaces
1. **Avoding Conflict**
2. **Restricting Access**
3. **Resource Limits**
4. **Default Namespaces**
    - **When we create a Kubernetes cluster, we get four default namespaces**
       - **Default**
       - **kube-node-lease**
       - **kube-public**
       - **kube-system**

5. **Default Namespace**
6.**kube-node-lease Namespace**
7.**kube-public Namespace**
8.**kube-system Namespace**

9. **Custom Namespaces**

## 3.Organizing Resources

## 4.Namespaces Can Be Created in Two Ways:
1. **Using `kubectl`**
2. **Using a configuration file**

3. **Creating a Namespace Using `kubectl`**
4. **Listing All Namespaces**
5. **Creating Namespaces Using a Configuration File**
6. **Getting the `apiVersion` of Namespaces in Kubernetes**
7. **Creating a Namespace Using a Configuration File**
8. **Applying the Configuration to the Cluster**
9. **Deleting the Existing Namespace**
10. **Reapplying the Configuration**
11. **Creating the `todo` Namespace**
     - **Create a Configuration File named `todo-namespace.yaml`**
12. **Apply the configuration file to create the todo namespace**
13. **Verify the Creation of the todo Namespace**
14. **Current Resources in the Cluster**
     - **Note:The above output consists of**
        - **Pods**
        - **Services**
        - **Deployments**
        - **ReplicaSets**
15. **Creating Resources in Their Respective Namespaces**
16. **Create the `nginx-deployment.yaml` File**
17. **Apply the Changes to the Cluster**
18. **Now get all the resources in the cluster**
19. **To get all the resources from nginx namespace**
20. **If we want to get all the reources from all namespaces**
21. **Apply the configuration file to create the nginx-service in the nginx namespace**
22. **To verify that the nginx-service is created in the nginx namespace**
23. **Now we can listdown the resources without specifying the namespace**
24. **Moving `todo` Resources to the `todo` Namespace**
25. **Create a file named `todo-ui-api.yaml`**
26. **Apply the configuration file to create the todo resources in the todo namespace**
     - **Accessing Services Across Namespaces**
27. **Viewing Resources**
     - **First, list the resources to confirm the setup**
28. **Accessing todo Service from nginx Pod**
     - **Enter the nginx Pod**
29. **Attempt to Access the todo Service**
30. **Accessing the Service Using Fully Qualified Domain Name (FQDN)**
     - **Services in Kubernetes can be accessed using a DNS pattern that includes the service name, namespace, and the .svc.cluster.local suffix**
     - **The DNS name can be simplified by omitting the .svc.cluster.local suffix:**
30. **Notes on Namespace Usage:**
     - **Accessing Services** 
     - **Default Namespace** 
     - **Multiple Teams/Environments** 
     - **Role-Based Access Control**
     - **Enterprise Setup**
     - **Deleting a Namespace** 
   

# 9: kubernetes Volumes

1. **Problem Statement-1**
2. **Problem Statement-2**

## 1.What are kubernetes Volumes?

## 2.Different types of volumes available in kubernetes with examples.

## 1.emptyDir
1. **Description** 
2. **Lifecycle** 
3. **Use Cases** 
    - **Example emptydir.yaml**

## 2.hostPath
1. **Description** 
2. **Lifecycle** 
3. **Use Cases** 
    - **Example hostpath.yaml**

## 3.PersistentVolume (PV)
1. **Description** 
2. **Lifecycle** 
3. **Use Cases** 
    - **Example pv.yaml**

## 4.PersistentVolumeClaim (PVC)
1. **Description** 
2. **Lifecycle** 
3. **Use Cases** 
    - **Example pvc.yaml**

## 5.StorageClass
1. **Description** 
2. **Lifecycle** 
3. **Use Cases** 
    - **Example sc.yaml**

## 6.Deploy mongodb into kubernetes cluster
1. **For better understanding try to deploy mongodb into kubernetes cluster**
    - **Download the mongodb compass "amd64.deb" in your local machine from MongoDB official website**
    - **Create a deployment.yaml for mongodb**
    - **Now apply the changes into the cluster**              
    - **To get the list of all pods**
    - **Now create the service.yaml**
    - **Now apply the changes in the cluster**
    - **To get the list of all services**
    - **Now port-forward this service so that we can access mongodb from our local machine**

## 7.MongoDB Compass Setup and Database Creation
1. **Connecting to MongoDB Compass**   
2. **Create a New Connection**   
3. **Creating a Simple Database**
4. **Verify Inserted Data**    
5. **To get the list of all pods in the cluster**
6. **To get into the pod with bin/bash command**
7. **To see on which pid the mongodb service is running**
8. **To kill the service/process**
9. **To list down all the pods**
10. **Now try to access the data we created earlier in mongodb db.todos**
     - **Note**
     - **Solution**

1. **emptyDir**
    - **Create a Deployment File**
    - **Apply the Changes to the Cluster**
    - **Port-Forward the Service**
    - **Creating a Simple Database in MongoDB**
       - **Go to Databases**
       - **Enter Database Details**
       - **Database Name**: `db`
       - **Collection Name**: `todos`
       - **Add Data**
       - **Verify Inserted Data**
   - **To get the list of all pods in cluster**
   - **To get into the pod with bin/bash command**
   - **To see on which pid the mongodb service is running**
   - **To kill the service/process**
   - **To list down all the pods**
   - **Now try to access the data we created earlier in mongodb db.todos**
   - **Data Storage in emptyDir Volumes on Minikube**
   - **Logging into Minikube Node**
   - **Getting the Pod ID**
   - **To get the pod ID directly**
   - **Navigating to the Volume Data**
   - **To list down the files in that directory "9bb53c02-9363-4a19-b9d3-470e36e6cf7b*"**
   - **Exploring the MongoDB Volume**
   - **Now lets try to delete the pod**
   - **Now list down the emptydir directory again**
   - **Now check this from the mongo compass**
   - **Verifying MongoDB Files in Minikube when pod is running without any RESTARTS**
      - **Listing Files in the `/data` Directory**
         - **To check the contents of the `/data` directory on the Minikube node**
      - **To get into the pod and see the data in the mongodb database**
      - **Access the Pod**
         - **Execute a shell inside the MongoDB pod**
      - **List Files in the /data Directory**
      - **List MongoDB Files in the /data/db Directory**

2. **hostPath**
    - **Create /data directory and give full permissions(rwx) in your host machine**
    - **Create the deployment file**
    - **Now apply the changes into the cluster**
    - **Now get the list of all pods**
    - **Now let us port-forward this service so that we can access mongodb from our machine**
    - **Creating a Simple Database in MongoDB**
       - **Go to Databases**
       - **Enter Database Details**
       - **Add Data**
       - **Verify Inserted Data**
    - **To get the list of pods**
    - **Now delete the pod**
    - **To check the pods list**
    - **Now again do the port-forwarding**
    - **Kubernetes offers three components to persist data across pod restarts and node failures**
       - **Persistent Volumes(PVs)**
       - **Persistent Volume Claims(PVCs)**
       - **Storage Classes(SCs)**

3. **Persistent Volumes(PVs)**
    - **Create two directories in localhost machine i.e local_storage and Minikube**
    - **Local Storage(host machine ubuntu) Directory**
       - **Create a directory called /home/ubuntu/storage in localhost and give permissions**
    - **Minikube cluster Storage Directory**
       - **Create a directory called /home/docker/storage in minikube "local-cluster" and give permissions**
       - **To get the api version of persistent volume**
       - **To create the pv.yaml file**
    - **Types of Persistent Volume Access Modes in Kubernetes**
       - **ReadWriteMany (RWX)**
       - **ReadWriteOnce (RWO)**
       - **ReadOnlyMany (ROX)**
       - **ReadOnlyOnce (RO)**
       - **ReadWriteOncePod (RWOP)**
    - **Now apply the changes to the local-cluster**
    - **To get the list of PersistentVolumes in our cluster**

4. **Persistent Volume Claims(PVCs)**
    - **Now create a PersistentVolumesClaim that need to be used in the pod**
    - **Now apply the changes into the cluster**
    - **List down all the PersistentVolumeClaim in the cluster**
    - **To use this PV and PVC, we need to make some changes in the deployment.yaml file which is required for our pod**
    - **Now apply the changes into the cluster**
    - **To list down all the pods in the cluster**
    - **Port-Forward the Service**
    - **Now go to MongoDB Compass and refresh the database**
    - **Now try to delete the pod in the cluster**
    - **To get the list of all pods**
    - **18.Now delete the pod**
    - **To get the list of all pods**
    - **Again do the port-forwarding as the connection is lost**
    - **To see where the data is mounted in local-cluster with PV,PVC,Deployemnt manifests**

5. **StorageClass(SCs)**
    - **To get the apiresources of storageclass**
    - **Create the StorageClass manifest**
    - **Apply the changes in the cluster**
    - **To get all the StorageClasses in the local-cluster**
    - **Now we have storageclass use it in the pvc.yaml**
    - **To get all the pvs in the local-cluster**
    - **To apply the changes in the local-cluster**
    - **To get the list of all PV's in the local-cluster**


# 10: kubernetes StatefulSets

1. **scaling, and consistent updates**
2. **Stable, unique network identities**
3. **Persistent storage**
4. **Sequential pod creation and scaling**
5. **Ideal for stateful applications**
6. **Controlled rolling updates**

## 1.StatefulSet Workflow 

## 2.When to Use StatefulSet?

## 3.Use Cases of StatefulSet

## 4.Key Benefits of StatefulSet
1. **Ordered Scaling and Deployment**
2. **Automated Rolling Updates**
3. **Persistent Storage Association**
4. **Unique Network Identifiers**

## 5.Limitations of Kubernetes StatefulSet
1. **Persistent Storage Handling**
2. **Storage Provisioning Requirement**
3. **Manual Headless Service Creation**
4. **Pod Termination on StatefulSet Deletion**
5. **Risks with Rolling Updates**

## 6.Stateful Applications vs Stateless Applications

## 7.Problems Encountered When Deploying Stateful Applications with Multiple Replicas.
1. **Problem Statement-1**
2. **Problem Statement-2**
3. **Problem Statement-3**

## 8.Deployment vs StatefulSets
1. **To get the api-resources of statefulset**
2. **create sc.yaml**
3. **Apply the changes into the cluster**                                                        
4. **Now create a statefulset.yaml file**
5. **Now apply the changes into the local-cluster**

1. **Ordered Pods**
    - **To list the all the pods in the local-cluster**
    - **Now try to scale up the StatefulSet and observe the behavior of the pods in the local cluster**
    - **To list all the pods in the local-cluster**
    - **Now lets try to sacledown and see what happens to the pods in the local-cluster**
    - **To list all the pods in the local-cluster**

2. **Sticky Identity**
    - **Now try to delete the pod and see if the same name is given to the pod in the local-cluster**
    - **To get the list of pods in the local-cluster**
    - **To get the list of pods in the local-cluster monitoring with watch (-w) flag**
    - **To get the list of pods in the local-cluster**

3. **Seperate PersistentVolumeClaims**
    - **Now to confirm each pod is using separate persistentvolumeclaims in the local-cluster**
    - **To describe the mongo-0 pod and get full information**

4. **Headless Service**
    - **Now, let's create a headless service in the local cluster**
    - **Apply the changes in the local-cluster**
    - **To verify that the headless services are created in the local-cluster**
    - **Now let us create the MongoDB replica sets in the local cluster**
    - **To get into the mongo pod in the local-cluster**
    - **Initiating the MongoDB Replica Set**
    - **Creating MongoDB Replica Sets in the Local Cluster**
       - **Step 1:Execute the Command to Initiate the Replica Set**
       - **Step 2:Reconnect to the MongoDB Shell and Verify Replica Set Status**
       - **Step 3:Verify Replica Set Status**
    - **Creating and Verifying Data Replication in MongoDB Replica Sets**
       - **Step 1:Create a Simple Database and Insert Data into the Primary Node**
       - **Step 2:Verify Data Replication in the First Secondary Node**
       - **Step 3:Verify Data Replication in the Second Secondary Node**
      
## 9.Summary


# 11: kubernetes ConfigMaps and Secrets.

1. **ConfigMaps** 
2. **Secrets** 

## 1.ConfigMaps

## 2.Creating and Managing ConfigMaps.

## 3.Types of ConfigMaps
1. **Imperative Way:Creating a ConfigMap Without Using a Definition File**
    - **Creating a ConfigMap from Literal Values**
    - **Creating a ConfigMap from a File**
    - **Creating a ConfigMap from a Directory**

2. **Declarative Way:Creating a ConfigMap Using a Definition File**
    - **creating a ConfigMap named app-config with key-value pairs using YAML**
    - **Using ConfigMaps in Pods**
    - **Using ConfigMaps as Environment Variables**
    - **2Using ConfigMaps as Mounted Volumes**
      
## 4.Secrets

## 5.Types of Secrets
1. **Opaque** 
2. **kubernetes.io/service-account-token**
3. **kubernetes.io/dockercfg and kubernetes.io/dockerconfigjson**
4. **kubernetes.io/basic-auth**
5. **kubernetes.io/ssh-auth**
6. **kubernetes.io/tls**

## 6.Secret vs ConfigMap

## 7.Security and Best Practices for ConfigMaps and Secrets in Kubernetes.
1. **ConfigMaps**
    - **Sensitive Data**
    - **Access Control**
    - **Encryption**
    - **Limited Use**
    - **Immutability**

2. **Secrets**
    - **Data Sensitivity**
    - **Encryption at Rest**
    - **API Access Control**
    - **Avoid Direct Access**
    - **Secret Rotation**

## 8.General Practices.
1. **Secure Delivery**
2. **Minimize Exposure**
3. **Namespace Isolation**
4. **Audit and Monitoring**
5. **Least Privilege**
6. **Automation**
7. **Regular Review**
8. **Educate and Train**

## 9.Do Not Hradcode

## 10.Different Ways to Configure Data in YAML
1. **Passing Arguments**
2. **Environment Variables** 
3. **Configuration Files**

1. **To get the api-resources of ConfigMap**
2. **Creating a ConfigMap in local-cluster**
    - **Important Notes**
       - **Size Limitation** 
       - **Immutable ConfigMaps** 
3. **Applying the ConfigMap in local-cluster**
4. **Verify whether the ConfigMap is created in local-cluster**
5. **create the statefulset.yaml**
6. **Now apply the statefulset.yaml changes in the local-cluster**
7. **Now lets try to list the pods that are running in the local-cluster**
8. **Now list down the environmental variables**
9. **Now lets see the configuation file is mounted or not**
10. **Now lets update the configmap and see by Creating a ConfigMap in local-cluster**
11. **Now apply the changes in the local-cluster**
12. **Now get into the pod and see if the values are updated automatically**
13. **Now lets check the environmental variables**
14. **Now lets restart a pod and see whether the environmental variables are updated after pod restart or not**
15. **Now list into the same mongo-0 pod and list down the environmental variables**
16. **To see if the data is stored in the new path i.e /data/db in the container**
17. **Creating a secret file in local-cluster**
19. **To encode a text from a command line**
20. **To refer to this secret data from the pod we need to do some changes in statefulset**
21. **Now lets delete the password from the mongo-congigmap.yaml**
22. **Now apply the changes in the local-cluster**
23. **Now delete the statefulset.yaml and reapply the changes in the local-cluster**
24. **Now get inside the mongo-0 pod in the local-cluster**
25. **Just like we updated the configmap we can update the secret too this can be done by**
26. **Changing a secret file in local-cluster**
27. **Now apply the changes in the local-cluster**
28. **Now delte the mongo-0 pod**
29. **The pods gets automatically created bcz we are using statefulset**
30. **Now get into the mongo-0 pod and see the changes are reflected**

## 11.Choosing between ConfigMap and Secret is very simple
1. **If you want to store non-sensitive, plain configuration data, use a ConfigMap.**
2. **If you want to store any sensitive data, use a Secret.**
3. **If a configuration includes both sensitive and non-sensitive data, you should use Secrets.**


# 12: kubernetes Probes

## 1.Reasons for any application to be unhealthy

## 2.kubernetes Probes and types
1. **Liveness Probe**
2. **Readiness Probe**
3. **Startup Probe**

## 3.Liveness Probe
1. **Purpose** 
2. **Use Case** 

1. **Use the statefulset to test the livenessprobe in local-cluster**

## 4.Probing Mechanisms.
1. **Exec**
2. **HTTP**
3. **TCP**

2. **For our mongo deployment lets make a simple change in statefulset.yaml file and check if our container is healthy or not**

## 5.Probing Customization.
1. **initialDelaySeconds**
2. **periodSeconds**
3. **timeoutSeconds**
4. **failureThreshold**
5. **successThreshold**

3. **Now apply the changes in the local-cluster**
4. **To list all the pods in a cluster**
5. **To describe the pod and get all the information about the pod**
6. **Correct the db1 to db in statefulset.yaml and apply the changes in local-cluster**
7. **To list all the pods**

## 6.Readiness Probe:
1. **Purpose** 
2. **Use Case** 

1. **Use the statefulset to test the livenessprobe in local-cluster**
2. **Now apply the changes in the local-cluster**
3. **To get the list of pods with a watch flag**
4. **We can check this by decribing the mongo-2 pod**
5. **Now let us try to describe the service and see this mongo-2 pod's IP is deleted from the endpoints list or not**
6. **Now try to describe the service mongo**
    - **Now fix this redainessprobe and see that the pod is again added back into the local-cluster**
7. **Now delete the changes in the local-cluster**
8. **Now apply the changes in the local-cluster**
9. **Now to list all the pods in a local-cluster**
10. **Now try to describe the service mongo and see all the three pods are available in local-cluster**

## 7.Startup Probe:
1. **Purpose** 
2. **Use Case** 
3. **We can set the restart policy in the pod's spec, which defines when the pod should be restarted:**
    - **Always:**
    - **OnFailure:**
    - **Never:**
    
1. **Use the statefulset to test the livenessprobe in local-cluster**
2. **Now delete the statefulset.yaml from cluster**
3. **Now apply changes in the local-cluster**          
4. **Now list down all the pod in the local-cluster**
5. **To get the information about the pod**
6. **Now list down all the pod in the local-cluster**
    - **Now simply correct the error in startup probe**
7. **To delete the statefulset in the cluster**
8. **To apply the statefulset in the cluster**
9. **To get the list of pods running in the cluster**

## 8.Probes Workflow:
1. **How Startup, Liveness, and Readiness Probes Work Together:**

## Best Practices:
1. **Ideal Frequency**
2. **Light Weight**
3. **Correct Restart Policy**
4. **Use Only When Needed**
5. **Keep An Eye On Probes Regularly**


# 13: kubernetes Resource Management

## 1. Key Things We Need to Understand:
1. **Requesting CPU and Memory** 
2. **Limiting CPU and Memory** 
3. **Quality of Service (QoS)** 
4. **Setting Minimum, Maximum, and Default Resources for Pods in a Namespace** 
5. **Limiting Resources in a Namespace** 

## 2.Computing Resources
1. **Requests**
2. **Limits**

## 3.Stress Tool

1. **Requests and Limits:**
    - **Note:Now we are going to use the stress tool**

1. **Create a pod in minikube cluster**
2. **To list down the nodes of the minikube cluster**
3. **To describe this minikube node**
4. **Now try to apply this changes into into the minikube cluster**
5. **To get the status of this pod**
6. **Now change the memory in the manifest file to 2GB and see that the pod gets scheduled**
7. **Now try to apply this changes into into the minikube cluster**       
8. **Now delete this pod**
9. **Now try to apply this changes into into the minikube cluster**       
10. **To get the list of pods in cluster**
11. **To get the status of pod in cluster**
12. **To check how many resources a pod is consuming**
13. **To enable metrics server in minikube**
14. **To check how many resources a pod is consuming**
15. **Now lets create a pod in minikube cluster**
16. **First delete the pod**
17. **Now apply the changes to the pod**
18. **Now list down the pod**
19. **Now lets create a pod in minikube cluster**
20. **First delete the pod**
21. **Now apply the changes to the pod**
22. **Now list down the pods and see the status**

## 4.Memory Management

## 5.Quality of Service(QoS):
1. **Best Effort Class** 
2. **Guaranteed Class** 
3. **Burstable Class**

1. **We can describe the pods Quality of service(QoS) using**

## 6.Limit Range
1. **Create a new manifest called limit-range-demo.yaml**
2. **Apply the changes in the cluster**
3. **Now create a pod more than the limit see what happens**
    - **Before applying these changes into the cluster do the below things**

4. **Delete the existing pod from the cluster**
5. **Now apply the changes into the cluster**
6. **Create a new manifest called limit-range-demo.yaml**
7. **Apply the changes in the cluster**
8. **Now create a pod more than the limit see what happens**
9. **Apply the changes in the cluster**
10. **To get the list of pods in the cluster**

## 7.Resource Quota
1. **Now create a new manifesat resource-quota-demo.yaml in cluster**
2. **Now create a pod more than the limit and see what will happens**
3. **Apply the changes in the cluster**
    - **To resolve this error make the changes in resources-demo-pod.yaml**
4. **Now create a pod more than the limit see what happens**
    - **Note:Dont use above manifest in future, The below manifest runs the pods without restarts bcz this satisfies my ResourceQuota:
    - **Note:Instead of memory limit 1Gi make it to 200Mi, that change can create a pod in the cluster.**
5. **To get the complete information of the pod in the cluster**


# 14: Kubernetes Advanced Scheduling

1. **NodeName**
2. **NodeSelector**
3. **Affinity**
    - **Node Affinity**
    - **Pod Affinity**
4. **Taints and Tolerations**

## 1.How Kubernetes Scheduler Works
1. **Create a minikube cluster with 4 nodes**
2. **To verify the nodes in the cluster**

## 2.Different methods to guide Kubernetes in scheduling pods onto specific nodes.

## 1.nodeName
1. **Now create a simple deployment file todo-ui-deployment.yaml**
2. **Now apply the deployment changes in cluster**
3. **Now list down the pods in the cluster and give wide option to display the nodes also**

## 2.nodeSelector
1. **Add the label to the first node i.e minikube**
2. **Add the label to the third node i.e minikube-m03**
3. **Now verify the node labels in the cluster**
    - **Note: We can see all the labels of nodes.** 
    - **Labels of first node:minikube**
    - **Labels of second node:minikube-m02**
    - **Labels of third node:minikube:m03**
    - **Labels of fourth node:minikube-m04**
4. **To filter our nodes with specific labels in our cluster**
5. **Now change the deployment file todo-ui-deployment.yaml**
6. **Now apply the deployment changes in cluster**
7. **Now list down the pods in the cluster and give wide option to display the nodes also**

## 3.Affinity

## 1.Node Affinity:
1. **Node Affinity is further divided into two types**
    - **Required During Scheduling, Ignore During Execution**
    - **Preffered During Scheduling, Ignore During Execution**

2. **Required During Scheduling, Ignore During Execution** 
    - **Required During Scheduling** 
    - **Ignore During Execution** 

1. **Now create a simple deployment file todo-ui-deployment.yaml**
    - **Note:** We assign node labels using `matchExpressions`, where we specify the key-value pair. 
       - **Key:"rank"**
       - **Operators**
          - **Gt** Greater than
          - **Lt** Less than
          - **In** Contains specified values
          - **NotIn** Does not contain specified values
          - **Exists** Contains specified labels
          - **DoesNotExist** Does not contain specified labels
2. **Now apply the deployment changes in cluster**
3. **Now list down the pods in the cluster and give wide option to display the nodes also**
4. **Now add these labels to the nodes 2 and 3 in the cluster**
    - **Add the label to the second node i.e minikube-m02**
    - **Add the label to the third node i.e minikube-m03**
5. **Now list down the pods in the cluster with above changes**

3. **Preferred During Scheduling, Ignore During Execution** 
1. **Now create a simple deployment file todo-ui-deployment.yaml**
2. **Now apply the deployment changes in cluster**
3. **Now list down the pods in the cluster and give wide option to display the nodes also**
4. **Now create a simple deployment file todo-ui-deployment.yaml**
5. **Now apply the deployment changes in cluster**
6. **Now list down the pods in the cluster and give wide option to display the nodes also**

## 2.Pod Affinity
1. **Create a todo-api-deployment in cluster**

1. **podAffinity is categorized into two types**
    - **Soft Affinity**
    - **Hard Affinity**
2. **Now apply the deployment changes in cluster**
3. **Now list down the pods in the cluster and give wide option to display the nodes also**

## 3.podAntiAffinity
1. **Create a todo-api-deployment in cluster**
2. **Now apply the changes in the cluster**
3. **Now list down all the pods in the cluster**

## 4.Taints and Tolerations
1. **There are 3 types of Taint effects**
    - **NoSchedule (Hard)** 
    - **PreferNoSchedule (Soft)** 
    - **NoExecute (Strict)** 

1. **So we can add it into the node Now taint a second node minikube-m02 in the cluster**
2. **Now verify this by listing down the pods**
3. **Now verify this by listing down the pods**
4. **If we want to look at the taints that are applied on a node we should describe the node i.e, minikube-02**
5. **We can delete a taint by using**
6. **Now add a taint with different effect now NoSchedule on node 2 i.e, "minikube-m02"**
7. **Create a todo-api-deployment in cluster**
8. **Now apply the changes in the cluster**
9. **Now list down all the pods in the cluster**
10. **Create a todo-api-deployment in cluster**
11. **Now apply the changes in the cluster**
12. **Now list down all the pods in the cluster**

## 5.Summary


# 15: Kubernetes Autoscaling:

## 1.Types of Autoscalers:
1. **Kubernetes offers 3 types of Autoscalers**
    - **HPA(Horizontal Pod Autoscaler)
    - **VPA(Vertical Pod Autoscaler)
    - **CA(Cluster Autoscaler)

## 2.HPA(Horizontal Pod Autoscaler)

## 3.VPA(Vertical Pod Autoscaler)

## 4.CA(Cluster Autoscaler)

## 5.Scale Up and Scale Down in (HPA, VPA, CA) Autoscaling
1. **Scale Up**
2. **Scale Down**

## 6.Working examples of Autoscalers

## 7.Horizontal Pod Autoscaler (HPA)**
1. **Horizontal Pod Autoscaler (HPA) Workflow**
2. **Usecase**

## 8.Create a simple Spring Boot API with a `/stress` endpoint that generates CPU load when invoked**

1. **Now deploy this API with kubernetes deployment**
2. **Now create a kubernetes service in cluster to access this application**
3. **Now create/define a HPA-Horizontal Pod AutoScaler for the above deployment**
4. **First apply our deployment of spring boot application**
5. **Now apply the service to access this application**
6. **Now apply the HPA in cluster**
7. **Now list down all the pods in the cluster**
8. **Now lets list down the metrics of these pods**
9. **Now list down the pods again**
10. **We can verify that HPA scaled it down by describing the HPA**
11. **Now try to describe this HPA**
12. **Now create a manifest file for traffic-generator.yaml**
13. **Now apply the changes in the cluster**
14. **To get the list of pods running in the cluster**
15. **Now try to get into this pod**
16. **To access the Traffic Generator Pod**
17. **Now lets simulate the load with this command in the pod**
18. **Now lets open the new tab and see the pod matrix**
19. **Now lets open the new tab and see the pod matrix**
20. **Now lets try to list the HPA from the cluster**
21. **We can also verify this by describing HPA**
22. **Now lets list down the metrics of the pod**
23. **Now lets get the HPA**
24. **.Now list down the pods in cluster**
25. **Now lets get the HPA**
26. **We can also verify this by describing HPA**

## 9.Vertical Pod Autoscaler (VPA)
1. **Vertical Pod Autoscaler (VPA) Workflow**
    - **Vertical Pod Autoscaler (VPA) performs three steps**
       - **Reads Resource Metrics** 
       - **Recommends Resources** 
       - **Updates Resources (Optional)** 

1. **Now create a vpa.yaml manifest file**
    - **Note:Here, our target is the same utility-api, and the updateMode is set to Off. The updateMode has three values**
       - Auto 
       - Off
       - Initial
2. **Now apply the changes in the cluster**
3. **Cloning VPA from the Git**
4. **Now get into the autoscaler repository/directory run this command to install VPA**
5. **Now apply the changes in the cluster for VPA**
6. **We can verify that by using**
7. **Now try to describe this VPA**

## 10.Cluster Autoscaler(CA)
1. **Note: We are using AWS cloud provider example for Cluster Autoscaler (CA)**

2. **Steps to Configure and Use Cluster Autoscaler on AWS**
    - **Set Up an EKS Cluster**
    - **Install the Cluster Autoscaler on EKS**
    - **Update IAM Policy to Allow EKS Cluster Autoscaler to Modify Nodes**
    - **Verify the Setup of the Cluster Autoscaler on EKS:**

1. **Create a ca.yaml file**
2. **Apply the ca.yaml File**
3. **Check the status of the Cluster Autoscaler**
4. **Describe the Cluster Autoscaler**

## 11.Monitoring Cluster Autoscaler: Nodes Before and After Scheduling a Pod
1. **Check Nodes Before Scheduling**
2. **Trigger Autoscaling by Scheduling a New Pod**
3. **Apply the Deployment**
4. **To get the list of pods in the cluster**
5. **Check the Cluster Autoscaler logs to see if it detected the unschedulable pod and started adding nodes**
6. **After the Cluster Autoscaler adds a new node, check the nodes in the cluster again**
7. **Finally, check that the previously unschedulable pod is now running**


# 16: Kubernetes Role Based Access Control(RBAC)

## 1.Authentication and Authorization
1. **Authentication**
2. **Authorization**

## 2.Create User
1. **Create a simple nginx pod in minikube**
2. **Apply the changes in the cluster**
                            
## 3.Users and Groups
1. **Creating a user**
2. **Open the config file in nano editor**
3. **First we should generate the users private key with openssl and store output file to the current folder**
4. **Next we must create a certificate signing request for the user "balu" with above private key**
5. **To get the details of present working directory**
6. **Our private key and CertificateSigningRequest(csr) are ready now verify them using**
7. **To sign it we should generate this command**
8. **Next we should add the user "balu" to the cluster**
9. **We should create the context with the same command**
10. **We can also verify that by using**
11. **To switch the context**
12. **Now try to list down the pods in the cluster**
13. **Now switch back to the minikube user**

## 4.Role and RoleBinding
1. **Creating a Role**
    - **Create a simple role manifest file**
2. **To see what kind of verbs we can give for a particular user**
3. **To switch the context**
4. **Now apply this manifest in the cluster**
5. **To verify the roles in the cluster**

## 5.Creating a RoleBinding to Attach Role, Subject, and Grant Permissions to User "balu"
1. **Create a rolebinding in the cluster**
2. **Now apply this rolebinding in the cluster**
3. **Now lets witch back to the user balu-minikube**
4. **Now list down the pods to see whether balu-minikube is having permissions to list down the pods**          
5. **Now lets create a pod in balu-minikube with user balu and see we can create a pod here**
6. **Now lets switch back to the minikube user**
7. **Now create a namespace "ns" and with namespace name "test" in minikube**
8. **Now create a new pod in this namespace**
9. **To verify this list down the pods in test namespace**
10. **Now switch back to balu-minikube of user balu and try to access this pod in the test namespace**

## 6.ClusterRole and ClusterRoleBinding
1. **Create a clusterrole in the cluste**
2. **Create a clusterrolebinding in the cluster**
3. **Now apply the ClusterRole changes in the cluster**
4. **Now switch back to the minikube context**
5. **Now apply the ClusterRole changes in the minikube context**
6. **Now apply the ClusterRoleBinding changes in the minikube context**
7. **Now switch back to the balu-minikube context**
8. **Now try to access or get the pod from test namespace we created**
9. **To get the pods from all the namespaces in the cluster**

## 7.Groups
1. **Create cluster-role-binding in the cluster**
2. **Now apply the cluster-role-binding changes in the cluster**

## 6.Service Accounts
1. **To get the service account in cluster**
2. **Now switch back to the minikube context**
3. **We can also find this namespace in test namespace**
4. **To create a service account in minikube**
5. **To get the service account in cluster**
6. **Now create a simple kubectl-pod and try to get the list of pods from this pod**
7. **Apply the changes in the minikube**
8. **Now get into the kubectl-pod**
9. **Now try to list down the pods**
10. **Now make changes in kubectl-pod**
11. **Now delete the pod in the cluster**
12. **Now apply the changes in the cluster to create the pod**
13. **Now try to get into the pods and try to list down the pods**
14. **To add permissions to this service account "test-sa" for this make changes in role-binding.yaml**
15. **Apply the changes in the cluster**
16. **Now try to list down the pods after updating the role-binding.yaml**
17. **Now come out of pod and see if we can perform action with a command**
18. **To test if a service account is having permission or not**
19. **Now lets try to see if we can get the pods**


# 17: Kubernetes DaemonSets:

## 1.Key Points
1. **Automatic Scheduling** 
2. **Use Cases** 
3. **Node Selectors**
4. **Updates**

## 2.DaemonSets for Node-Specific Tasks
1. **DaemonSets** 
    - **Collecting Logs** 
    - **Monitoring Metrics** 
    - **Node Maintenance Tasks** 

## 3.Monitoring Nodes with DaemonSets
1. **DaemonSets** 
    - **Here's how DaemonSets address the challenge**
       - **Automatic Scheduling** 
       - **Consistency** 
       - **Difference from Deployments** 

## 4.Common Use Cases for DaemonSets
1. **Logging Agents**
    - **Example**
    - **Purpose** 

2. **Monitoring Agents**
    - **Example** 
    - **Purpose** 

3. **Network Agents**
    - **Example** 
    - **Purpose** 

## 5.Practical Example: Running Prometheus Node Exporter with DaemonSet
1. **Now create a simple daemonset**
    - **To get the api-version of daemonset in our cluster**
2. **Apply the changes in the cluster**
3. **We can verify that using**
4. **Now list down the pods**
5. **We can verify that using**
6. **Now try to add another node to our cluster**
7. **To get the list of nodes in our cluster**
8. **We can verify that by using**
9. **Now lets try to delete the second node i.e, minikube-m02**
10. **Now try to list down the pods**
11. **Now that we have monitoring pods running, let's check if they are providing metrics by port-forwarding these pods**
12. **Now lets go to the browser**
13. **To verify the daemonsets in all namespaces**
14. **Now list down the pods**
15. **When ever we delete a daemonset all pods managed by it are automatically deleted.We can verify that by using**
16. **To get the list of pods**
17. **Deleting a DaemonSet Without Deleting Managed Pods (Deprecated Method)**
18. **Deleting a DaemonSet Without Deleting Managed Pods (Updated Method)**


# 18: Kubernetes Jobs and CronJobs

## 1.Kubernetes Jobs Lifecycle

## 2.Use Cases
1. **DB Backups**
2. **Log Rotation**
3. **Data Processing**

## 3.Now let us see how we can take the backup of the mongodb using job and then schedule it with cronjob
1. **To get the information about StatefulSets**
2. **Create the mongo-configmap.yaml**
3. **Apply the changes in the cluster**
4. **To encode a text from a command line**
5. **Create the mongo-secret.yaml**
6. **Apply the changes in the cluster**
7. **Create the sc.yaml**
8. **Apply the changes in the cluster**
9. **Create the statefulset.yaml**
10. **Apply the changes in the cluster**
11. **Create headless-service.yaml**
12. **Apply the changes in the local-cluster**
13. **To verify that the headless services are created in the local-cluster**
14. **Now let us try create the mongodb replicasets in local-cluster**
15. **To get the list of pods in the local-cluster**
16. **To get into the mongo pod in the local-cluster**
17. **Now lets try to execute the same command inside the mongo-0 pod in local-cluster**
18. **Now exit and get into the mongo-0 pod again**
19. **Create a database with a collection in the Primary Node and verify if the same data is replicated in the Secondary Nodes**
20. **Now log into the second node of the pod and see that it have the same data as primary node**
21. **Now exit from this mongo-1 pod and verify that the mongo-2 is having the same replicated data**
22. **Now list the pods in cluster**
23. **Now verify the statefulsets in the cluster**
24. **Now create the pvc.yaml manifest file**
25. **Now apply the changes into the cluster**

## 4.Kubernetes Jobs
1. **Now create a job manifest file in cluster**
2. **Now apply this changes and see whether the database is getting exported or not** 
3. **Now list down the jobs that are running in cluster**
4. **Now list down the pods that are running in the cluster**
5. **Now try to get the logs of error which is effecting pod to get created in cluster**
6. **Whenever you are getting this error we need to do the following changes in your mongodb database**
    - **1.Access MongoDB** 
    - **2.Start MongoDB Shell** 
    - **3.Check the Replica Set Status** 
    - **4.Check Users and Roles** 
    - **5.Test Authentication** 
    - **6.Check Authentication Mechanism** 
       - **Create the Username and Password in the mongodb**
    - **7.Now apply the job.yaml again in the cluster**
    - **8.Now try to list down the pods in the cluster**
    - **9.Now try to list down the jobs in cluster**
    - **10.Now see whether our data is exported or not.This data should be exported to volumes,so list down the volumes**
       - **To get PVC**
       - **To get PV**
    - **11.Now describe this volume where the data is getting stored** 
    - **12.Now lets get into our host i.e, minikube**
    - **13.Now try to list down the contents of the directory**
    - **14.Now get into the admin, test databases**
       - **Getting into admin database**
       - **Getting into test database**
    - **15.If we are doubtful, let’s get into the MongoDB pod and check if that collection exists**
    - **16.Now try to list down the databases in the mongodb**
    - **17.Now try to list down the collections in the database as we have created collections in test not in admin database**
    - **18.Now try to list down the jobs again in cluster**
    - **19.Now verify this by listing the pods**
    - **20.Create home-work.yaml**
    - **21.To get the list of jobs in the cluster**
    - **22.Apply the changes in the cluster**
    - **22.Make the changes in the home-work.yaml**
    - **23.Now apply the changes in cluster**
    - **24.Now list down the pods**
    - **25.Now get the logs of the two pods**
    - **26.Now list down the jobs in cluster**
    - **27.To get the list of pod**
    - **28.To stop this Job from creating more pods or to check the final status of the Job, you can delete the Job**
    - **29.To list down the pods**
    - **30.This will terminate the Job, and you can then check its final status using `kubectl get jobs`**

## 5.Kubernetes CronJobs
1. **Now create a cronjob manifest in cluster**
    - **CronJobs Concurrency Policy**
       - **The available options are**
          - **Allow** 
          - **Forbid** 
          - **Replace** 
2. **Now apply the changes in the cluster**
3. **Now list down the all the cronjobs in cluster**
4. **Now list down the pods**
5. **Now list down the jobs**
6. **Now verify the list of pods in cluster**
7. **Now get the list of jobs in the cluster**
8. **Now apply the changes in the cluster**
9. **Now list down the jobs in cluster**
10. **Now get into the minikube and see whether data is being exported to the file specified i.e, mongodump**
11. **Now list down the jobs**
12. **We can also suspend the CronJob to stop creating jobs temporarily. To do this, edit the CronJob**
13. **Jobs created before applying patching**
14. **Now get the last schedule of the cronjob**
15. **Now lets try to rollback the suspend patch so that the pods will be created normally**
16. **To get the list of CronJobs**
17. **To get the list of jobs in the cluster**
18. **To get the list of pods**
19. **Now, to delete the CronJob we created: when we delete the CronJob, the associated jobs will also be deleted**

## 👥 Who Is This For?

> [!IMPORTANT]
> This collection is perfect for:
>
> - **DevOps Engineers**: Get quick access to the tools you use every day.
> - **Sysadmins**: Simplify operations with easy-to-follow guides.
> - **Developers**: Understand the infrastructure behind your applications.
> - **DevOps Newcomers**: Transform from beginner to expert with in-depth concepts and hands-on projects.

## 🛠️ How to Use This Repository

> [!NOTE]
> 1. **Explore the Categories**: Navigate through the folders to find the tool or technology you’re interested in.
> 2. **Use the Repositories**: Each repository is designed to provide quick access to the most important concepts and projects.

## 🤝 Contributions Welcome!

We believe in the power of community! If you have a tip, command, or configuration that you'd like to share, please contribute to this repository. Whether it’s a new tool or an addition to an existing content, your input is valuable.

## 📢 Stay Updated

This repository is constantly evolving with new tools and updates. Make sure to ⭐ star this repo to keep it on your radar!

## Liking the Project?

# ⭐❤️

If you find this project helpful, please consider giving it a ⭐! It helps others discover the project and keeps me motivated to improve it.

Thank you for your support!
---
## ✍🏼 Author

![Author Image](https://github.com/balusena/docker-for-devops/blob/main/banner.png)

---
Made with ❤️ and passion to contribute to the DevOps community by [Bala Senapathi](https://github.com/balusena)







































































































































































































