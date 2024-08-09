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
      ```
      {
        "title": "Refer Docker Volumes"
      }
      ```

   ![MongoDB Add Data](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/data_insertion.png)

4. **Verify Inserted Data**:
    - After inserting, the data should appear as:
      ```
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
### 2.To get into the pod with bin/bash command.
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
![MongoDB Refresh Data](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/data_refresh.png)

**Note:**
As we can see, the entire database has been deleted. The issue with this approach is that data is not 
shared across containers, and when a container is deleted, the data associated with it is also lost.

**Solution:**
The solution to this problem is to store the data at the pod level instead of the container level. 
Kubernetes provides a volume type called emptyDir for this purpose. An emptyDir volume is created 
when a pod is first assigned to a node and remains active as long as the pod is running on that node. 
Containers within the pod can share the same data because it is stored at the pod level. Thus, even if 
a container is restarted, the data remains available within the pod.

**1.emptyDir:**
Instead of storing data at the container level, we can store it at the pod level. In Kubernetes, there 
is a type of volume called emptyDir, which allows us to store data at the pod level. An emptyDir volume 
is created when a pod is first assigned to a node and remains active as long as the pod is running on 
that node. This allows containers within the pod to share the same data since it is stored at the pod 
level. Even if a container is restarted, the data will still be available in the pod.

![Kubernetes emptyDir](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/emptydir.png)

**Note:**
To store data at the pod level, all we need to do is declare the volume under the spec section. Under 
volumes, we specify the name as "mongo-volume" and set the type of the volume to "emptyDir: {}".

### 1.Create a Deployment File
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
          volumeMounts:
            - mountPath: /data/db
              name: mongo-volume    
      volumes:
        - name: mongo-volume
          emptyDir: {}
```
**Note:** 
Once the volume is declared, we need to mount this volume onto the container. Under the containers 
section, add volumeMounts, and set the mountPath to "/data/db" this is the directory inside the 
container. Also, specify the volume name as "mongo-volume". Remember, if there are multiple containers
in the pod, you can mount the same volume into different or the same directories in each container.

### 2.Apply the Changes to the Cluster
```
ubuntu@balasenapathi:~$ kubectl apply -f deployment.yaml
deployment.apps/mongo configured
```
### 3.Port-Forward the Service
**Note:** We will access it on port 32000 as the Mongo service is running on port 27017.
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
```
### 4.Creating a Simple Database in MongoDB

1. **Go to Databases**:
   - Click on `+ Create Database`.

2. **Enter Database Details**:
   - **Database Name**: `db`
   - **Collection Name**: `todos`
   - Click on `Create Database`.

3. **Add Data**:
   - Select the `db.todos` collection.
   - Click on `Insert to Collection db.todos`.
   - **Insert Data**:
   ```json
   {
     "title": "Refer Docker Volumes"
   }

4. **Verify Inserted Data**:
   - After inserting, the data should appear as:
     ```
     {
       "_id": {
         "$oid": "650314ddd752a335b8da81a2"
       },
       "title": "Testing"
     }
     ```
### 5.To get the list of all pods in cluster:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
mongo-5dc459c68f-mr79x   1/1     Running   0          11m
```
### 6.To get into the pod with bin/bash command:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-5dc459c68f-mr79x -- /bin/bash
root@mongo-5dc459c68f-mr79x:/#
```
### 7.To see on which pid the mongodb service is running
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-5dc459c68f-mr79x -- /bin/bash
root@mongo-5dc459c68f-mr79x:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mongodb        1  1.0  3.6 2689220 146320 ?      Ssl  14:05   0:07 mongod --dbpath /data/db --auth --bind_ip_all
mongodb       27  0.1  0.0      0     0 ?        Z    14:05   0:00 [mongod] <defunct>
root         165  0.1  0.0   4624  3612 pts/0    Ss   14:17   0:00 /bin/bash
root         172  0.0  0.0   7060  1604 pts/0    R+   14:18   0:00 ps aux
root@mongo-5dc459c68f-mr79x:/#
```
**Note:** Our mongodb service/process is running in PID 1

### 8.To kill the service/process:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-5dc459c68f-mr79x -- /bin/bash
root@mongo-5dc459c68f-mr79x:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mongodb        1  1.0  3.6 2689220 146320 ?      Ssl  14:05   0:07 mongod --dbpath /data/db --auth --bind_ip_all
mongodb       27  0.1  0.0      0     0 ?        Z    14:05   0:00 [mongod] <defunct>
root         165  0.1  0.0   4624  3612 pts/0    Ss   14:17   0:00 /bin/bash
root         172  0.0  0.0   7060  1604 pts/0    R+   14:18   0:00 ps aux
root@mongo-5dc459c68f-mr79x:/# kill 1
root@mongo-5dc459c68f-mr79x:/# command terminated with exit code 137
```
**Note**: We can see that our MongoDB container has been deleted, and we are now outside of that container.

### 9.To list down all the pods:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS      AGE
mongo-5dc459c68f-mr79x   1/1     Running   1 (34s ago)   15m
```
**Note:** Here the RESTARTS are 1 meaning the container is restarted,please note that we restarted the container not the pod

### 10.Now try to access the data we created earlier in mongodb db.todos
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
```
**Note:** This time, we can see that the data is not deleted, even after restarting the container. 
This is because we are storing the data at the pod level using an emptyDir volume in Kubernetes. 
When data is stored in an emptyDir volume, it remains available as long as the pod exists. Even if the
container restarts, the data is preserved.

### 11.Data Storage in emptyDir Volumes on Minikube
We might be wondering where the data is being stored. The emptyDir data gets stored on the node where our 
pod is running. In this case, our pod is running in Minikube.

### 1.Logging into Minikube Node.
```
ubuntu@balasenapathi:~$ minikube ssh
docker@minikube:~$
```
**Note:** The data of the emptyDir volume is stored in /var/lib/kubelet/pods.
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods
0b0496e5-46f0-4f64-8ccd-1833c64d1112  8af0e85a28544808d52bb7c47ad824ed	    ba67b72e-56ce-449f-8ead-0f2aed53fd91
20e5d451-c543-46a7-82e7-5a2b22399ad3  9605cf50-b014-4629-95e0-50af82ecaf57  e14e2f92c469337ac62a252dad99dcc5
2f0b7cfd-57d6-426a-9877-7e334bb293bd  9bb53c02-9363-4a19-b9d3-470e36e6cf7b  e33f7a2a0d6aad5df18c7258d3116e25
4e275e35949ad3fdfeb753c1099308e7      a4126030-99a0-463e-8bb5-f0d8294e32c6
6596a2e8-0070-46da-920e-2e28e0fbfb83  abd589fb-83a4-4c82-ac49-efbc8216e29d
```
**Note:** As we can see, these are directories with pod IDs.

### 2.Getting the Pod ID.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS      AGE
mongo-5dc459c68f-mr79x   1/1     Running   1 (16m ago)   31m
```
```
ubuntu@balasenapathi:~$ kubectl get pod mongo-5dc459c68f-mr79x -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2023-09-14T14:05:29Z"
  generateName: mongo-5dc459c68f-
  labels:
    app: mongo
    pod-template-hash: 5dc459c68f
  name: mongo-5dc459c68f-mr79x
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: mongo-5dc459c68f
    uid: adadab91-5ed2-4ce8-8eac-b7196d6da68e
  resourceVersion: "211370"
  uid: 9bb53c02-9363-4a19-b9d3-470e36e6cf7b ====> This is our pod id
spec:
  containers:
  - args:
    - --dbpath
    - /data/db
    env:
    - name: MONGO_INITDB_ROOT_USERNAME
      value: admin
    - name: MONGO_INITDB_ROOT_PASSWORD
      value: password
    image: mongo
    imagePullPolicy: Always
    name: mongo
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /data/db
      name: mongo-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-xhxgx
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir: {}
    name: mongo-volume
  - name: kube-api-access-xhxgx
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-09-14T14:05:30Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-09-14T14:20:28Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-09-14T14:20:28Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-09-14T14:05:30Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://a7acd68fecb94c77e779021ce8729b2b6358eec09327f9c5b3f155e7e7c6589a
    image: mongo:latest
    imageID: docker-pullable://mongo@sha256:31bf43c4959c283733a004b0a3a9c4ddc52efafb4cf9a38e42d2da93e9a72546
    lastState:
      terminated:
        containerID: docker://aa9c5c3b42fd1c6711b8a72f7004e5365cf594896ad7b568aef0263a00b7efed
        exitCode: 0
        finishedAt: "2023-09-14T14:20:23Z"
        reason: Completed
        startedAt: "2023-09-14T14:05:41Z"
    name: mongo
    ready: true
    restartCount: 1
    started: true
    state:
      running:
        startedAt: "2023-09-14T14:20:27Z"
  hostIP: 192.168.49.2
  phase: Running
  podIP: 10.244.0.143
  podIPs:
  - ip: 10.244.0.143
  qosClass: BestEffort
  startTime: "2023-09-14T14:05:30Z"
```
**Note:** From the YAML output, the pod UID is:
```
uid: 9bb53c02-9363-4a19-b9d3-470e36e6cf7b ====> This is our pod id
```
### 3.To get the pod ID directly.
```
ubuntu@balasenapathi:~$ kubectl get pod mongo-5dc459c68f-mr79x -o jsonpath='{.metadata.uid}'
9bb53c02-9363-4a19-b9d3-470e36e6cf7b
```
### 4.Navigating to the Volume Data.
```
ubuntu@balasenapathi:~$ minikube ssh
docker@minikube:~$ sudo ls /var/lib/kubelet/pods
0b0496e5-46f0-4f64-8ccd-1833c64d1112  8af0e85a28544808d52bb7c47ad824ed	    ba67b72e-56ce-449f-8ead-0f2aed53fd91
20e5d451-c543-46a7-82e7-5a2b22399ad3  9605cf50-b014-4629-95e0-50af82ecaf57  e14e2f92c469337ac62a252dad99dcc5
2f0b7cfd-57d6-426a-9877-7e334bb293bd  9bb53c02-9363-4a19-b9d3-470e36e6cf7b  e33f7a2a0d6aad5df18c7258d3116e25
4e275e35949ad3fdfeb753c1099308e7      a4126030-99a0-463e-8bb5-f0d8294e32c6
6596a2e8-0070-46da-920e-2e28e0fbfb83  abd589fb-83a4-4c82-ac49-efbc8216e29d
```
**Note:** Among the directories listed, 9bb53c02-9363-4a19-b9d3-470e36e6cf7b corresponds to our pod ID.

### 5.To list down the files in that directory "9bb53c02-9363-4a19-b9d3-470e36e6cf7b*" .
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods/9bb53c02-9363-4a19-b9d3-470e36e6cf7b
containers  etc-hosts  plugins volumes
```
**Note:** As we can see in this directory there are different directories such as containers,etc-hosts,plugins, volumes

- So our volumes data is stored under "/var/lib/kubelet/pods/9bb53c02-9363-4a19-b9d3-470e36e6cf7b/volumes"
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods/9bb53c02-9363-4a19-b9d3-470e36e6cf7b/volumes
kubernetes.io~empty-dir  kubernetes.io~projected
```
**Note:** The kubernetes.io~empty-dir volume directory contains our data.
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods/9bb53c02-9363-4a19-b9d3-470e36e6cf7b/volumes/kubernetes.io~empty-dir
mongo-volume
```
### 6.Exploring the MongoDB Volume.
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods/9bb53c02-9363-4a19-b9d3-470e36e6cf7b/volumes/kubernetes.io~empty-dir/mongo-volume
WiredTiger	   _mdb_catalog.wt			 collection-7-4062898949116254082.wt  index-5-4062898949116254082.wt  mongod.lock
WiredTiger.lock    collection-0-4062898949116254082.wt	 diagnostic.data		      index-6-4062898949116254082.wt  
sizeStorer.wt
WiredTiger.turtle  collection-2--6189966264042125540.wt  index-1-4062898949116254082.wt       index-8-4062898949116254082.wt  
storage.bson
WiredTiger.wt	   collection-2-4062898949116254082.wt	 index-3--6189966264042125540.wt      index-9-4062898949116254082.wt
WiredTigerHS.wt    collection-4-4062898949116254082.wt	 index-3-4062898949116254082.wt       journal
```
**Note:** These are the actual database files in the emptyDir volume. However, storing data in an emptyDir
volume is not an ideal solution. We still face the same issues as before: challenges with "sharing data"
and "persisting data" with emptyDir.

If multiple pods are running the same application, other pods cannot access the data if it's stored at 
the pod level. Additionally, if the pod gets deleted or restarted for any reason, all the data associated
with that pod is also deleted, leading to data loss.

![Kubernetes emptyDir Drawback](https://github.com/balusena/kubernetes-for-devops/blob/main/09-Kubernetes%20Volumes/emptydir_drawback.png)






