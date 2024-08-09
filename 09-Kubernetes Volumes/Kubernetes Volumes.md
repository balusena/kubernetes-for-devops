# kubernetes Volumes
Containers are considered 'ephemeral,' meaning that when a container is deleted, all data associated with
it will be lost. However, we can persist the data even if the container is deleted by using Docker volumes,
and the same concept applies to Kubernetes.

![Container Pod](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/container_pod.png)

When a pod is deleted, all the data associated with it is also deleted. Now, we will look at how to persist
the data even if the pod is deleted, using Kubernetes Volumes.

**Problem Statement-1**
When we create pods using a Deployment or ReplicaSet, Kubernetes ensures that the specified number of 
replicas is maintained. For example, if we create three (3) replicas of our application and write some 
data to those pods, Kubernetes will automatically create a new pod if any replica or pod goes down, to 
maintain the desired state of three (3) replicas. However, this new pod will start with a clean state, 
and any data previously written to the pod will be lost. This leads to data loss when a pod is deleted 
or restarted.

![Problem Statement 1](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/ps-1.png)

**Problem Statement-2**
When we have multiple replicas of the same application, how do they share the same data? By default, the
data is stored in the container and is not accessible to other pods, meaning the data is not shared across
pods. These are the two main challenges we encounter when storing data in Kubernetes.

![Problem Statement 2](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/ps-2.png)

This is where Kubernetes Volumes come into the picture. With Kubernetes Volumes, we can solve the two problems mentioned above:

- 1.Sharing data across pods.
- 2.Persisting data even if a pod restarts or a node goes down.

## 1.What are kubernetes Volumes?
Kubernetes Volumes are storage abstractions that allow data to be shared and persisted across containers
within a pod or even across pods. Unlike container storage, which is ephemeral and lost when the container 
stops or restarts, Kubernetes Volumes provide a way to store data that outlasts the lifecycle of individual
containers. Volumes can be backed by various storage types, such as local disks, network file systems, or
cloud storage solutions. They enable data persistence, sharing among pods, and storage management in a 
Kubernetes environment, ensuring that critical data is not lost even if pods or nodes fail.

![Kubernetes Volumes](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/kubernetes_volumes.png)

**Note:** In simple terms, a volume is a directory with some data in it, and this data is accessible to 
the containers within a pod.

## 2.Different types of volumes available in kubernetes with examples.

## 1.emptyDir
- **Description:** An `emptyDir` volume is a temporary storage directory created when a pod is assigned to a node. It starts empty and is used by the containers within the pod.
- **Lifecycle:** The data in an `emptyDir` volume is ephemeral and will be deleted when the pod is removed.
- **Use Cases:** Useful for temporary data that does not need to be preserved beyond the pod's lifecycle, such as caches or scratch space.

**Example emptydir.yaml:**
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: myimage
    volumeMounts:
    - mountPath: /data
      name: myemptydir
  volumes:
  - name: myemptydir
    emptyDir: {}
```
## 2.hostPath
- **Description:** A hostPath volume mounts a file or directory from the host node's filesystem into a pod. This allows containers to access files and directories on the host machine.
- **Lifecycle:** The data persists as long as the pod is running but is tied to the host node's lifecycle.
- **Use Cases:** Useful for accessing files on the host machine, such as log files or configuration files.

**Example hostpath.yaml:**
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: myimage
    volumeMounts:
    - mountPath: /data
      name: myhostpath
  volumes:
  - name: myhostpath
    hostPath:
      path: /path/on/host
      type: Directory
```
## 3.PersistentVolume (PV)
- **Description:** A PersistentVolume (PV) represents a piece of storage in the cluster, provisioned by an administrator or dynamically using StorageClasses. It is a resource that provides durable storage.
- **Lifecycle:** The PV exists independently of the pods and can be reused across different pods. It outlives individual pods.
- **Use Cases:** Provides persistent storage that remains available beyond pod restarts and re-creations, often used for databases or applications needing durable storage.

**Example pv.yaml:**
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /path/on/host
  storageClassName: manual
```
## 4.PersistentVolumeClaim (PVC)
- **Description:** A PersistentVolumeClaim (PVC) is a request for storage by a user. It abstracts the details of the underlying storage infrastructure and allows users to claim storage resources.
- **Lifecycle:** The PVC binds to a PV that satisfies its requirements, and the data persists beyond individual pods.
- **Use Cases:** Used to request and claim storage resources for pods, separating storage requests from infrastructure details.

**Example pvc.yaml:**
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```
## 5.StorageClass
- **Description:** A StorageClass defines different storage types and their parameters in Kubernetes. It allows administrators to specify classes of storage and provisioning policies.
- **Lifecycle:** Defines policies for storage provisioning and can dynamically provision PVs when a PVC is created.
- **Use Cases:** Enables dynamic provisioning of storage based on predefined policies and configurations, such as specifying SSDs or HDDs.

**Example sc.yaml:**
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

**For better understanding try to deploy mongodb into kubernetes cluster:**

### 1.Download the mongodb compass "amd64.deb" in your local machine from MongoDB official website.
```
ubuntu@balasenapathi:~$ cd Downloads

ubuntu@balasenapathi:~/Downloads$ ls
apache-hive-3.1.2-bin                   Oracle_VM_VirtualBox_Extension_Pack-7.0.8.vbox-extpack
apache-hive-3.1.2-bin.tar.gz            prometheus-2.44.0.linux-amd64.tar.gz
google-chrome-stable_current_amd64.deb  spark-3.2.0-bin-hadoop3.2
hadoop-3.2.1                            spark-3.2.0-bin-hadoop3.2.tgz
hadoop-3.2.1.tar.gz                     ubuntu-20.04.6-live-server-amd64.iso
iris.csv                                ubuntu-22.04.2-live-server-amd64.iso
mongodb-compass_1.39.4_amd64.deb        virtualbox-7.0_7.0.8-156879~Ubuntu~focal_amd64.deb
node_exporter-1.6.0.linux-amd64.tar.gz

ubuntu@balasenapathi:~/Downloads$ sudo dpkg -i mongodb-compass_1.39.4_amd64.deb
[sudo] password for ubuntu:balasenapathi
Selecting previously unselected package mongodb-compass.
(Reading database ... 287332 files and directories currently installed.)
Preparing to unpack mongodb-compass_1.39.4_amd64.deb ...
Unpacking mongodb-compass (1.39.4) ...
Setting up mongodb-compass (1.39.4) ...
Processing triggers for gnome-menus (3.36.0-1ubuntu1) ...
Processing triggers for desktop-file-utils (0.24-1ubuntu3) ...
Processing triggers for mime-support (3.64ubuntu1) ...
```
### 2.Create a deployment.yaml for mongodb:
```
ubuntu@balasenapathi:~$ nano deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - image: mongo
          name: mongo
          args: ["--dbpath", "/data/db"]
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              value: "admin"
            - name: MONGO_INITDB_ROOT_PASSWORD
              value: "password"
```              
### 3.Now apply the changes into the cluster:              
```         
ubuntu@balasenapathi:~$ kubectl apply -f deployment.yaml
deployment.apps/mongo created
```
### 4.To get the list of all pods:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
mongo-6966577c7c-ljtgd   1/1     Running   0          44s
```
**Note:** Our mongo pod is created and it is up and running.

### 5.Now create the service.yaml
```
ubuntu@balasenapathi:~$ nano service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-svc
spec:
  ports:
    - port: 27017
      protocol: TCP
      targetPort: 27017
      nodePort: 32000
  selector:
    app: mongo
  type: NodePort
```
**Note:** To access the mongo deployment we need to create the service.

### 6.Now apply the changes in the cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f service.yaml
service/mongo-svc created
```
### 7.To get the list of all services:
```
ubuntu@balasenapathi:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)           AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP           3m1s
mongo-svc    NodePort    10.99.79.56   <none>        27017:32000/TCP   22s
```
**Note:** The mongo service is created.

### 8.Now port-forward this service so that we can access mongodb from our local machine:

**Note:** we are try to access it on 32000 port as the mongo service is running on 27017
```
ubuntu@balasenapathi:~$ kubectl port-forward svc/mongo-svc 32000:27017
Forwarding from 127.0.0.1:32000 -> 27017
Forwarding from [::1]:32000 -> 27017
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
```
**Note:** Our MongoDB service is exposed on port 32000. Now, try to access the deployed MongoDB from 
MongoDB Compass. MongoDB Compass is a client used to connect to your MongoDB database.

After installing MongoDB Compass on your system, open the MongoDB Compass application and authenticate 
to connect to your MongoDB instance.

# MongoDB Compass Setup and Database Creation

## Connecting to MongoDB Compass

1. **Open MongoDB Compass**:
   Launch the MongoDB Compass application on your system.

2. **Create a New Connection**:
    - **URI**: `mongodb://localhost:32000/`

    - **Advanced Connection Options**:
        - **Authentication Method**: `Username/Password`
        - **Username**: `admin`
        - **Password**: `password`
        - Click on `[Connect]`

   ![MongoDB Compass](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/mongodb_compass.png)

## Creating a Simple Database

1. **Go to Databases**:
    - Click on `Create Database`.

2. **Enter Database Details**:
    - **Database Name**: `db`
    - **Collection Name**: `todos`
    - Click on `Create Database`.

   ![MongoDB Database Creation](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/database_creation.png)

3. **Add Data**:
    - Select the `db.todos` collection.
    - Click on `Insert to Collection db.todos`.
    - **Insert Data**:
      ```json
      {
        "title": "Refer Docker Volumes"
      }
      ```

   ![MongoDB Add Data](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/data_insertion.png)

4. **Verify Inserted Data**:
    - After inserting, the data should appear as:
      ```json
      {
        "_id": {
          "$oid": "62fa268ed6c84bf7704d1e5b"
        },
        "title": "Refer Docker Volumes"
      }
      ```

   ![MongoDB Verify Data](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/data_verify.png)

**Note:** Our data is stored in MongoDB, but it resides within the container. As a result, the data cannot
be accessed by other containers. Additionally, if the container is deleted or restarted, all the data 
written to it will be lost.

To demonstrate this, let's try killing the process running in the container and see if the data is 
preserved or lost.

### 1.To get the list of all pods in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
mongo-6966577c7c-ljtgd   1/1     Running   0          7m44s
```
### 2.To get inside into the pod with bin/bash command.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-6966577c7c-dkqxp -- /bin/bash
root@mongo-6966577c7c-dkqxp:/#
```
### 3.To see on which pid the mongodb service is running.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-6966577c7c-ljtgd -- bin/bash
root@mongo-6966577c7c-ljtgd:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mongodb        1  1.1  3.6 2694528 144868 ?      Ssl  13:03   0:07 mongod --dbpath /data/db --auth --bind_ip_all
root         167  0.0  0.0   4624  3516 pts/0    Ss   13:12   0:00 bin/bash
root         174  1.0  0.0   7060  1632 pts/0    R+   13:14   0:00 ps aux
root@mongo-6966577c7c-ljtgd:/#
```
**Note:** Our mongodb service/process is running in PID 1

### 4.To kill the service/process:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-6966577c7c-ljtgd -- bin/bash
root@mongo-6966577c7c-ljtgd:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mongodb        1  1.1  3.6 2694528 144868 ?      Ssl  13:03   0:07 mongod --dbpath /data/db --auth --bind_ip_all
root         167  0.0  0.0   4624  3516 pts/0    Ss   13:12   0:00 bin/bash
root         174  1.0  0.0   7060  1632 pts/0    R+   13:14   0:00 ps aux
root@mongo-6966577c7c-ljtgd:/# kill 1
root@mongo-6966577c7c-ljtgd:/# command terminated with exit code 137
```
**Note**: We can see that our MongoDB container has been deleted, and we are now outside of that container.

### 5.To list down all the pods:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS        AGE
mongo-6966577c7c-ljtgd   1/1     Running   1 (2m18s ago)   16m
```
**Note:** Here the RESTARTS are 1 meaning the container is restarted,please note that we restarted the container not the pod

### 6.Now try to access the data we created earlier in mongodb db.todos
```
ubuntu@balasenapathi:~$ kubectl port-forward svc/mongo-svc 32000:27017
Forwarding from 127.0.0.1:32000 -> 27017
Forwarding from [::1]:32000 -> 27017
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
Handling connection for 32000
E0914 18:48:07.841612   99681 portforward.go:409] an error occurred forwarding 32000 -> 27017: error forwarding port 27017 to pod
e559886297a80542e63f0560bf3460d00229808f60f6da249f7ba50ee1aafca4, uid : exit status 1: 2023/09/14 13:18:07 socat[105426] E connect(5,
AF=2 127.0.0.1:27017, 16): Connection refused
E0914 18:48:07.856152   99681 portforward.go:394] error copying from local connection to remote stream: EOF
error: lost connection to pod
```
**Note:**
As we can see, the entire database has been deleted. The issue with this approach is that data is not 
shared across containers, and when a container is deleted, the data associated with it is also lost.

The solution to this problem is to store the data at the pod level instead of the container level. 
Kubernetes provides a volume type called emptyDir for this purpose. An emptyDir volume is created 
when a pod is first assigned to a node and remains active as long as the pod is running on that node. 
Containers within the pod can share the same data because it is stored at the pod level. Thus, even if 
a container is restarted, the data remains available within the pod.







