## Kubernetes Jobs and CronJobs:
Kubernetes Jobs are used to manage tasks that need to run to completion. They ensure that a specified number of pods 
execute successfully. This makes Jobs ideal for batch processing tasks, running scripts, or any task that needs to run
once until completion. Kubernetes will create the necessary pods and manage their execution to ensure the job completes
successfully, even restarting pods if needed.

CronJobs are an extension of Jobs and allow you to schedule tasks at specific times or intervals, similar to cron jobs 
in Unix-like systems. They are useful for automating repetitive tasks such as backups, report generation, or cleanup 
operations. CronJobs ensure that these tasks run automatically at the scheduled times, without the need for manual 
intervention. This makes them powerful tools for maintaining the regular operations of a Kubernetes cluster.

In Kubernetes, we have different controllers for managing pods, such as ReplicaSets, Deployments, StatefulSets, and DaemonSets.
These controllers ensure that their pods are always running. If a pod fails, the controller restarts it or reschedules it to 
another node to maintain the desired number of running pods. However, there are scenarios where you may want to run your pods 
only once, such as taking database backups or sending emails in a batch. These processes should not be running continuously; 
they should run for a specific amount of time and at particular intervals.

Using controllers like Deployments for such tasks is not ideal, as they ensure that the pod runs continuously. Instead, for these
kinds of batch jobs, you can use Jobs and CronJobs in Kubernetes. Jobs allow you to run processes only once, while CronJobs enable
you to schedule them to run at specific intervals.

![Kubernetes Jobs CronJobs](https://github.com/balusena/kubernetes-for-devops/blob/main/18-Kubernetes%20Jobs%20and%20CronJobs/jobs_cronjobs_intro.png)

## Kubernetes Jobs Lifecycle:
When a Kubernetes job is created, it launches a pod with the image specified in the manifest. This image can be any application
that contains the logic for our use case. If there are errors in running the application in the pod due to reasons such as memory
or CPU issues, the job retries a given number of times by recreating the pod. The number of retries can be specified using the 
`backoffLimit`. For example, if we set the `backoffLimit` to three (3) and the pod fails, Kubernetes will create a new pod to 
retry the task. If the new pod also fails, Kubernetes will create another pod, and so on, until three (3) attempts have been made.
If the job still fails after three (3) attempts, Kubernetes will mark the job as failed with a "BackoffLimitExceeded" error.

The `backoffLimit` field is very useful to ensure that our job is not retried indefinitely, which could waste resources and cause
unnecessary load on the system. If we don’t set anything, the `backoffLimit` defaults to six (6).

Another important property in the job manifest file is `activeDeadlineSeconds`. This property defines the maximum duration
for which a job should run. If the job runs longer than this period, it will be marked as incomplete with a "DeadlineExceeded"
error, regardless of how many pods are created. Note that `activeDeadlineSeconds` takes precedence over `backoffLimit`.

If there are no errors in the pod, the job controller checks the number of pod completions. This can be specified in the job
manifest, indicating the minimum number of successful runs required. This completion count in the job is similar to replicas
in deployments. The job will continue to spin up new pods until the required completions are met. Once the specified number
of successful completions is reached, the job will be marked as completed.

When we set `completions` > 1, by default, pods are created one by one sequentially. However, we can create pods in parallel
by setting the `parallelism` value. For example, if `parallelism` is set to two (2) and `completions` to three (3), two (2)
pods will be created initially. Once they are completed, another one (1) pod will be created. This is the lifecycle of a job.
These jobs are executed only once, meaning when a job is created, the pod is created, the execution runs in the pod, and the 
pod is then completed.

However, sometimes we want to run a job on a scheduled basis, such as for database backups where we want to back up our database
every day at midnight. For this, Kubernetes provides another resource called a `CronJob`. In a `CronJob`, we can specify a cron
expression to define the schedule. When the `CronJob` is created, Kubernetes creates a job according to the specified schedule,
and from there, the same job flow applies.

In short, Kubernetes Jobs and CronJobs are useful for automating and managing a wide variety of tasks in a containerized environment.
Jobs and CronJobs can be used in multiple use cases.

![Kubernetes Jobs Lifecycle](https://github.com/balusena/kubernetes-for-devops/blob/main/18-Kubernetes%20Jobs%20and%20CronJobs/kubernetes_jobs_lifecycle.png)

## Use Cases:

![Kubernetes Jobs CronJobs Usecases](https://github.com/balusena/kubernetes-for-devops/blob/main/18-Kubernetes%20Jobs%20and%20CronJobs/jobs_cronjobs_usecases.png)

**1. DB Backups:**
This ensures that our database is always backed up and can be restored later when needed.

**2. Log Rotation:**
Jobs and CronJobs can also be used for log rotation to ensure that our log files do not grow too large, while still providing
access to historical log data when needed.

**3. Data Processing:**
These resources can be utilized for data processing tasks such as parsing logs, extracting data, and transforming data. This is
particularly useful for tasks that need to be performed on an ad-hoc basis or on a schedule. Beyond these examples, Jobs and 
CronJobs can be applied to many more use cases.

- Now let us see how we can take the backup of the mongodb using job and then schedule it with cronjob:

### 1.To get the information about StatefulSets.
```
ubuntu@balasenapathi:~$ kubectl get sts
No resources found in default namespace.
```
### 2.Create the mongo-configmap.yaml.
```
ubuntu@balasenapathi:~$ nano mongo-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: mongodb-config
immutable: false
data:
  username: admin
  mongodb.conf: |
    storage:
      dbPath: /data/db
    replication:
      replSetName: "rs0"
```
### 3.Apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f mongo-configmap.yaml
configmap/mongodb-config created
```
### 4.To encode a text from a command line.
```
ubuntu@balasenapathi:~$ echo -n password | base64
cGFzc3dvcmQ=
```
Note: Please note that the data in the secret must be given as base64 encoded data.

### 5.Create the mongo-secret.yaml.
```
ubuntu@balasenapathi:~$ nano mongo-secret.yaml
apiVersion: v1
kind: Secret
metadata: 
  name: mongodb-secret
immutable: false
type: Opaque
data:
  password: cGFzc3dvcmQ= 
```
### 6.Apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f  mongo-secret.yaml
secret/mongodb-secret created
```
### 7.Create the sc.yaml.
```
ubuntu@balasenapathi:~$ nano sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: demo-storage
provisioner: k8s.io/minikube-hostpath
volumeBindingMode: Immediate
reclaimPolicy: Delete 
```
### 8.Apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f  sc.yaml
storageclass.storage.k8s.io/demo-storage created
```
### 9.Create the statefulset.yaml.
```
ubuntu@balasenapathi:~$ nano statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:4.0.8
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                configMapKeyRef:
                  key: username
                  name: mongodb-config
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: password
                  name: mongodb-secret
          command:
            - mongod
            - "--bind_ip_all"
            - --config=/etc/mongo/mongodb.conf
          volumeMounts:
            - name: mongo-volume
              mountPath: /data/db
            - name: mongodb-config
              mountPath: /etc/mongo
      volumes:
        - name: mongodb-config
          configMap:
            name: mongodb-config
            items:
              - key: mongodb.conf
                path: mongodb.conf
  volumeClaimTemplates:
    - metadata:
        name: mongo-volume
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: demo-storage
        resources:
          requests:
            storage: 1Gi 
```
### 10.Apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo created
```
### 11.Create headless-service.yaml.
```
ubuntu@balasenapathi:~$ nano headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  ports:
    - name: mongo
      port: 27017
      targetPort: 27017
  clusterIP: None
  selector:
    app: mongo
```    
### 12.Apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f headless-service.yaml
service/mongo created
```
### 13.To verify that the headless services are created in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP     110m
mongo        ClusterIP   None         <none>        27017/TCP   100s
```
**Note:** Here the cluster-ip is none, note that headless services cannot be accessed directly from the outside of the cluster.

### 14.Now let us try create the mongodb replicasets in local-cluster.

As we have 3 pods, we should be able to specify a specific pod as the Master Node and the others as Slave Nodes. Note that
StatefulSets only manage the pods; they do not create the replication for us. We need to handle the replication ourselves,
which requires getting into the pod.

### 15.To get the list of pods in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          35m
mongo-1   1/1     Running   0          67m
mongo-2   1/1     Running   0          67m
```
### 16.To get into the mongo pod in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("f51c393c-d744-4888-aecf-3427a479cb9e") }
MongoDB server version: 4.0.8
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2023-09-25T18:43:24.440+0000 I STORAGE  [initandlisten] 
2023-09-25T18:43:24.441+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger 
storage engine
2023-09-25T18:43:24.441+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] 
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not 
recommended.
2023-09-25T18:43:26.049+0000 I CONTROL  [initandlisten] 
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
> 

Note: Now we are in the mongo-0 pod in local-cluster

Asper the official documentation of mongodb this is how we initiate the replicaset

rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "mongodb0.example.net:27017" },
      { _id: 1, host: "mongodb1.example.net:27017" },
      { _id: 2, host: "mongodb2.example.net:27017" }
   ]
})

***Our Custom replicaset with DNS:

rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
      { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
      { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
   ]
})
```
**Note:** The name of the ReplicaSet is `"rs0"`, as specified in our StatefulSet manifest file. The DNS names of our 3 pods are as follows:
- `mongo-0` host: `"mongodb0.example.net:27017"`
- `mongo-1` host: `"mongodb1.example.net:27017"`
- `mongo-2` host: `"mongodb2.example.net:27017"`

### 17.Now lets try to execute the same command inside the mongo-0 pod in local-cluster.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c6badafa-2063-4000-b789-18c200f371ed") }
MongoDB server version: 4.0.8
Server has startup warnings: 
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] 
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger 
storage engine
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-09-25T21:46:22.123+0000 I CONTROL  [initandlisten] 
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not 
recommended.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] 
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

> rs.initiate( {
...    _id : "rs0",
...    members: [
...       { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
...       { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
...       { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
...    ]
... })
{
	"ok" : 1,
	"operationTime" : Timestamp(1695678772, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695678772, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
rs0:SECONDARY> 
```
**Note:** As we can see, `"rs0"` is the ReplicaSet name that we specified in the StatefulSet. The hosts for `mongo-0`, 
`mongo-1`, and `mongo-2` are as follows:
`"mongo-0.mongo.default.svc.cluster.local:27017"`, 
`"mongo-1.mongo.default.svc.cluster.local:27017"`, 
`"mongo-2.mongo.default.svc.cluster.local:27017"` 
- where 
`"mongo-0"` is the pod name, 
`"mongo"` is the service name, and 
`"default"` is the namespace. 
This is how we define the DNS for the 3 pods. As we can see, the ReplicaSet has been initiated with `"ok": 1`.

### 18.lets exit and get into the mongo-0 pod again:
```
> rs.initiate( {
...    _id : "rs0",
...    members: [
...       { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
...       { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
...       { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
...    ]
... })
{
	"ok" : 1,
	"operationTime" : Timestamp(1695678772, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695678772, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
rs0:SECONDARY> exit
bye
```
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("19288f4c-402c-4d8c-9248-a2c8d85c23d5") }
MongoDB server version: 4.0.8
Server has startup warnings: 
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] 
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger 
storage engine
2023-09-25T21:46:21.032+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-09-25T21:46:22.123+0000 I CONTROL  [initandlisten] 
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not 
recommended.
2023-09-25T21:46:22.124+0000 I CONTROL  [initandlisten] 
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

rs0:PRIMARY> 

Note: Now you can see that the node is changed to PRIMARY node

#We can verify that by using rs.status() in the pod:

rs0:PRIMARY> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2023-09-25T22:05:37.139Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1695679535, 1),
			"t" : NumberLong(1)
		}
	},
	"lastStableCheckpointTimestamp" : Timestamp(1695679505, 1),
	"members" : [
		{
			"_id" : 0,
			"name" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 1157,
			"optime" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-09-25T22:05:35Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"electionTime" : Timestamp(1695678784, 1),
			"electionDate" : ISODate("2023-09-25T21:53:04Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "mongo-1.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 764,
			"optime" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-09-25T22:05:35Z"),
			"optimeDurableDate" : ISODate("2023-09-25T22:05:35Z"),
			"lastHeartbeat" : ISODate("2023-09-25T22:05:36.658Z"),
			"lastHeartbeatRecv" : ISODate("2023-09-25T22:05:36.687Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceHost" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "mongo-2.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 764,
			"optime" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1695679535, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-09-25T22:05:35Z"),
			"optimeDurableDate" : ISODate("2023-09-25T22:05:35Z"),
			"lastHeartbeat" : ISODate("2023-09-25T22:05:36.660Z"),
			"lastHeartbeatRecv" : ISODate("2023-09-25T22:05:36.692Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceHost" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1695679535, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695679535, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
```
**Note:** If we observe, we have `"stateStr": "PRIMARY"`, `"stateStr": "SECONDARY"`, and `"stateStr": "SECONDARY"` for 
the three nodes. After the initial clone, all the Secondary/Slave Nodes will be in sync with the Primary/Master Node.

### 19.Create a database with a collection in the Primary Node and verify if the same data is replicated in the Secondary Nodes.
```
rs0:PRIMARY> use test
switched to db test
rs0:PRIMARY> db.todos.insert({"title":"Test"})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.todos.find()
{ "_id" : ObjectId("651206cbf0530d8dc0720209"), "title" : "Test" }
rs0:PRIMARY>exit 
bye
```
### 20.Now log into the second node of the pod and see that it have the same data as primary node.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-1 -- mongo
rs0:SECONDARY> db.todos.find()
Error: error: {
	"operationTime" : Timestamp(1695680385, 1),
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk",
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695680385, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```
**Note:** As we can see, there is an error indicating that this node is not set up to perform reads. To enable reads on
the Secondary node, use the following command:
```
rs0:SECONDARY> rs.slaveOk()

rs0:SECONDARY> db.todos.find()
{ "_id" : ObjectId("651206cbf0530d8dc0720209"), "title" : "Test" }

rs0:SECONDARY> exit
bye
```
### 21.Now exit from this mongo-1 pod and verify that the mongo-2 is having the same replicated data.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-2 -- mongo
rs0:SECONDARY> db.todos.find()
Error: error: {
	"operationTime" : Timestamp(1695681005, 1),
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk",
	"$clusterTime" : {
		"clusterTime" : Timestamp(1695681005, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```
**Note:** As we can see, there is an error indicating that this node is not set up to perform reads. To enable reads on
the Secondary node, use the following command:
```
rs0:SECONDARY> rs.slaveOk()

rs0:SECONDARY> db.todos.find()
{ "_id" : ObjectId("651206cbf0530d8dc0720209"), "title" : "Test" }
```
**Note:** As we can see, the data from the `secondary1` node is replicated to the `secondary2` node. This data transfer 
occurred with the help of the headless service. This is how pods in the StatefulSet communicate with each other for data
replication.

### 22.Now list the pods in cluster.
```       
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          106s
mongo-1   1/1     Running   0          32s
mongo-2   1/1     Running   0          20s
```
### 23.Now verify the statefulsets in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get sts
NAME    READY   AGE
mongo   3/3     111s
```
**Note:** This is the MongoDB StatefulSet we deployed. As a result, we have 3 replicas of MongoDB running, deployed using StatefulSets.

### 24.Now create the pvc.yaml manifest file.
```
ubuntu@balasenapathi:~$ nano pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodump
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "demo-storage"  # Use the appropriate storage class name
  resources:
    requests:
      storage: 1G
```
#Now apply the changes into the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f pvc.yaml
persistentvolumeclaim/mongodump created
```

**Kubernetes Jobs:**

- Now let us write a kubernetes job that takes the backup of this mongodb:

### 1.Now create a job manifest file in cluster.
```
ubuntu@balasenapathi:~$ nano job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: mongodb-backup-job
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  ttlSecondsAfterFinished: 60 # delete job after 60 seconds
  template:
    spec:
      containers:
        - name: mongodb
          image: mongo
          command: ["/bin/sh", "-c"]
          args:
            [
              'mongodump --uri "mongodb://test1:${MONGO_PASSWORD}@mongo-0.mongo.default.svc.cluster.local:
              27017,mongo-1.mongo.default.svc.cluster.local:27017,mongo-2.mongo.default.svc.cluster.local:27017/?
              replicaSet=rs0&readPreference=secondaryPreferred&authSource=admin&ssl=false" -o /usr/share/mongodump/$(date +"%d%m%Y-
              %H%M%S")',
            ]
          volumeMounts:
            - mountPath: "/usr/share/mongodump/"
              name: mongodump
          env:
            - name: MONGO_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: password
                  name: mongodb-secret
      volumes:
        - name: mongodump
          persistentVolumeClaim:
            claimName: mongodump
      restartPolicy: Never
```
**Note:** We have given:

- `apiVersion: batch/v1`
- `kind: Job`
- `name: mongodb-backup-job` — name of the job we are creating.
- `backoffLimit: 5` — We are setting this limit to 5, which means if the job fails, the pod retries up to 5 times.
- `activeDeadlineSeconds: 100` — We are setting this limit to 100 seconds, meaning if the job runs for more than 100 seconds, the job will be marked as an error.
- `ttlSecondsAfterFinished: 60` — This limit is set to 60 seconds. When the job is finished, it will be automatically deleted after 60 seconds.

**Note:** When a job is deleted, the pods associated with that job are also deleted.

- **Template:** In this section, we provide the container information just like we do in a pod.
  - `image: mongo` — Here we are using the MongoDB image, and in this container, we are running the `mongodump` command.

**Note:** `mongodump` is a utility that creates the binary export of our database.

The command used is:
```
mongodump --uri "mongodb://admin:${MONGO_PASSWORD}@mongo-0.mongo.default.svc.cluster.local:27017,mongo-1.mongo.default.svc.cluster.local:27017,mongo-2.mongo.default.svc.cluster.local:27017/?replicaSet=rs0&readPreference=secondaryPreferred&authSource=admin&ssl=false" -o /usr/share/mongodump/$(date +"%d%m%Y-%H%M%S")
```

**The above is the connection string of our database:**

- `admin` is the username.
- `${MONGO_PASSWORD}` is taken from a secret.
- `@mongo-0` is our first pod name.
- `mongo` is our headless service.
- `default` is the namespace.
- `svc.cluster.local` is the service DNS.
- `27017` is the port number on which our MongoDB is running.
- `@mongo-1` is our second pod name.
- `@mongo-2` is our third pod name.
- `/usr/share/mongodump/$(date +"%d%m%Y-%H%M%S")` — Finally, we are giving the path to which the exported dump should be saved in the container.

**Volume Mounts:**

- `mountPath: "/usr/share/mongodump/"`
- `name: mongodump`

To store this volume, we are mounting this volume at `/usr/share/mongodump/` with the name `mongodump`.

**Volumes:**

- `name: mongodump`
- `persistentVolumeClaim:`
  - `claimName: mongodump`

**restartPolicy: Never** — As we are using `mongodump` with PersistentVolumeClaim.

### 2.Now apply this changes and see whether the database is getting exported or not. 
```
ubuntu@balasenapathi:~$ kubectl apply -f job.yaml
job.batch/mongodb-backup-job created
```
### 3.Now list down the jobs that are running in cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                 COMPLETIONS   DURATION   AGE
mongodb-backup-job   0/1           7s         7s
```
### 4.Now list down the pods that are running in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME                       READY   STATUS    RESTARTS   AGE
mongo-0                    1/1     Running   0          34m
mongo-1                    1/1     Running   0          34m
mongo-2                    1/1     Running   0          34m
mongodb-backup-job-rmt4p   0/1     Error     0          15s
mongodb-backup-job-cdqsj   0/1     Pending   0          0s
mongodb-backup-job-cdqsj   0/1     Pending   0          0s
mongodb-backup-job-cdqsj   0/1     ContainerCreating   0          0s
```
**Note:** Now we can see that we are getting an error while the `mongodb-backup-job` is running and attempting to create 
a pod in the cluster.

### 5.Now try to get the logs of error which is effecting pod to get created in cluster.
```
ubuntu@balasenapathi:~$ kubectl logs mongodb-backup-job-cdqsj 
2023-10-12T23:41:03.835+0000	Failed: can't create session: failed to connect to mongodb://
admin:password@mongo-0.mongo.default.svc.cluster.local:27017,mongo-1.mongo.default.svc.cluster.local:
27017,mongo-2.mongo.default.svc.cluster.local:27017/?replicaSet=rs0&readPreference=secondaryPreferred&authSource=admin&ssl=false: 
connection() error occurred during connection handshake: auth error: sasl conversation error: unable to authenticate using mechanism 
"SCRAM-SHA-1": (AuthenticationFailed) Authentication failed.
```
**Note:** We are encountering an "Authentication failed" error related to MongoDB. This indicates that there is an issue
with the username or password used for authentication.

### 6.Whenever you are getting this error we need to do the following changes in your mongodb database.

1. **Access MongoDB:** First, access one of your MongoDB cluster nodes. You can use `kubectl exec` to access a container
within a pod, similar to what you've done before:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# 
```
2. **Start MongoDB Shell:** Once inside the container, start the MongoDB shell by running.
```
root@mongo-0:/# mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("a2311379-9e54-41c2-9e41-bd81140f42dc") }
MongoDB server version: 4.0.8
Server has startup warnings: 
2023-10-12T23:06:37.577+0000 I STORAGE  [initandlisten] 
2023-10-12T23:06:37.577+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger 
storage engine
2023-10-12T23:06:37.577+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-10-12T23:06:38.494+0000 I CONTROL  [initandlisten] 
2023-10-12T23:06:38.495+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-10-12T23:06:38.495+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-10-12T23:06:38.495+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not 
recommended.
2023-10-12T23:06:38.495+0000 I CONTROL  [initandlisten] 
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
```
3.**Check the Replica Set Status:** Verify that the replica set is properly configured by running.

**Note:** This command should provide information about the replica set configuration, including the hostnames of members
and their status.
```
rs0:PRIMARY> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2023-10-13T00:16:24.877Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1697156182, 1),
			"t" : NumberLong(1)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1697156182, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1697156182, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1697156182, 1),
			"t" : NumberLong(1)
		}
	},
	"lastStableCheckpointTimestamp" : Timestamp(1697156142, 1),
	"members" : [
		{
			"_id" : 0,
			"name" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 4187,
			"optime" : {
				"ts" : Timestamp(1697156182, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-10-13T00:16:22Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"electionTime" : Timestamp(1697152119, 1),
			"electionDate" : ISODate("2023-10-12T23:08:39Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "mongo-1.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 4075,
			"optime" : {
				"ts" : Timestamp(1697156182, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1697156182, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-10-13T00:16:22Z"),
			"optimeDurableDate" : ISODate("2023-10-13T00:16:22Z"),
			"lastHeartbeat" : ISODate("2023-10-13T00:16:24.291Z"),
			"lastHeartbeatRecv" : ISODate("2023-10-13T00:16:24.282Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceHost" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "mongo-2.mongo.default.svc.cluster.local:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 4075,
			"optime" : {
				"ts" : Timestamp(1697156182, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1697156182, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2023-10-13T00:16:22Z"),
			"optimeDurableDate" : ISODate("2023-10-13T00:16:22Z"),
			"lastHeartbeat" : ISODate("2023-10-13T00:16:24.192Z"),
			"lastHeartbeatRecv" : ISODate("2023-10-13T00:16:24.185Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceHost" : "mongo-0.mongo.default.svc.cluster.local:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1697156182, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1697156182, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```
4.**Check Users and Roles:** To see the list of users and their roles, you can run.

**Note:**This will list all users in the admin database along with their roles.
```
rs0:PRIMARY> use admin
switched to db admin

rs0:PRIMARY> db.getUsers()
[ ]
```

5.**Test Authentication:** To test authentication with a specific username and password, you can authenticate using the db.auth() method. 
Replace yourUsername and yourPassword with the actual username and password.

**Note:** If authentication is successful, it will return 1.
```
rs0:PRIMARY> db.auth("yourUsername", "yourPassword")
Error: Authentication failed.
0
```
6.**Check Authentication Mechanism:** Verify the authentication mechanism used in your MongoDB deployment. For SCRAM-SHA-1 authentication 
(as indicated by your error message), ensure that the users are created with the SCRAM-SHA-1 mechanism.
```
rs0:PRIMARY> use admin
switched to db admin
rs0:PRIMARY> db.auth("admin", "password")
Error: Authentication failed.
0
```
It appears that the MongoDB cluster is running correctly and the replica set rs0 is operational. However, when you attempted to 
authenticate using db.auth("yourUsername", "yourPassword"), it failed with "Authentication failed."

**Note:** As it is clear, we haven't provided the username and password for the MongoDB database. This is the reason we are 
unable to connect or authenticate to this database and, consequently, unable to create the job.

7.**Create the Username and Password in the mongodb:**
```
rs0:PRIMARY> use admin
switched to db admin
rs0:PRIMARY> db.createUser({
...     user: "admin",
...     pwd: "password",
...     roles: [
...         { role: "userAdminAnyDatabase", db: "admin" },
...         "readWriteAnyDatabase"
...     ]
... })

Successfully added user: {
	"user" : "admin",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		},
		"readWriteAnyDatabase"
	]
}
```
It looks like we have successfully created a user with the username "admin" and the password "password" with the necessary roles.
Now we should be able to use these credentials for authentication in our application or job manifest for dbbackup.

### 7.Now apply the job.yaml again in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f job.yaml
job.batch/mongodb-backup-job created
```
### 8.Now try to list down the pods in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME                       READY   STATUS      RESTARTS   AGE
mongo-0                    1/1     Running     0          78m
mongo-1                    1/1     Running     0          78m
mongo-2                    1/1     Running     0          78m
mongodb-backup-job-jj6g6   0/1     Completed   0          15s
```
**Note:** We can see,that the pod has been created by the job, and the status of `mongodb-backup-job-jj6g6` has changed to 
"Completed." This indicates that our job has run successfully.

### 9.Now try to list down the jobs in cluster:
```
ubuntu@balasenapathi:~$ kubectl get jobs 
NAME                 COMPLETIONS   DURATION   AGE
mongodb-backup-job   0/1           112s       112s

ubuntu@balasenapathi:~$ kubectl get jobs 
NAME                 COMPLETIONS   DURATION   AGE
mongodb-backup-job   1/1           7s         34s
```
**Note:** We can see that the mongodb-backup-job COMPLETIONS status is 1/1 means one backup job is finished/completed.

### 10. Now see whether our data is exported or not.This data should be exported to volumes,so list down the volumes.
### To get PVC.
```
ubuntu@balasenapathi:~$ kubectl get pvc
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mongo-volume-mongo-0   Bound    pvc-3379281b-2dec-4db1-9be6-fdf02b1035d7   1Gi        RWO            demo-storage   118m
mongo-volume-mongo-1   Bound    pvc-659ea510-51c8-42df-b725-d1a3e619094e   1Gi        RWO            demo-storage   118m
mongo-volume-mongo-2   Bound    pvc-258e2dac-f556-4810-a6bc-d1a6eaa83239   1Gi        RWO            demo-storage   118m
mongodump              Bound    pvc-cc9fc3d5-2d06-42d5-aad7-d3b7ef1459bf   1Gi        RWO            demo-storage   40m
```
### To get PV.
```
ubuntu@balasenapathi:~$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          
STORAGECLASS   REASON   AGE
pvc-258e2dac-f556-4810-a6bc-d1a6eaa83239   1Gi        RWO            Delete           Bound    default/mongo-volume-mongo-2   demo-
storage            118m
pvc-3379281b-2dec-4db1-9be6-fdf02b1035d7   1Gi        RWO            Delete           Bound    default/mongo-volume-mongo-0   demo-
storage            118m
pvc-659ea510-51c8-42df-b725-d1a3e619094e   1Gi        RWO            Delete           Bound    default/mongo-volume-mongo-1   demo-
storage            118m
pvc-cc9fc3d5-2d06-42d5-aad7-d3b7ef1459bf   1Gi        RWO            Delete           Bound    default/mongodump              demo-
storage            40m
```
Note: This is the volume "pvc-cc9fc3d5-2d06-42d5-aad7-d3b7ef1459bf " that is created.

### 11.Now describe this volume where the data is getting stored. 
```
ubuntu@balasenapathi:~$ kubectl describe pv pvc-cc9fc3d5-2d06-42d5-aad7-d3b7ef1459bf
Name:            pvc-cc9fc3d5-2d06-42d5-aad7-d3b7ef1459bf
Labels:          <none>
Annotations:     hostPathProvisionerIdentity: 71dded11-f000-454f-a977-1312b2921c0e
                 pv.kubernetes.io/provisioned-by: k8s.io/minikube-hostpath
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    demo-storage
Status:          Bound
Claim:           default/mongodump
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/hostpath-provisioner/default/mongodump
    HostPathType:  
Events:            <none>
```
**Note:** We can see this is stored in hostpath i.e, " /tmp/hostpath-provisioner/default/mongodump " in this location.

### 12.Now lets get into our host i.e, minikube.
```
ubuntu@balasenapathi:~$ minikube ssh

docker@minikube:~$ ls /tmp/hostpath-provisioner/default/mongodump
13102023-002517
```
**Note:** As we can see, the folder `13102023-002517` has been created, which matches the format specified in the 
`job.yaml` manifest, i.e., `(date +"%d%m%Y-%H%M%S")`.

### 13.Now try to list down the contents of the directory.
```
docker@minikube:~$ ls /tmp/hostpath-provisioner/default/mongodump/13102023-002517
admin  test
```
**Note:** We can see that the admin, test databases got exported.

### 14.Now get into the admin, test databases.

### Getting into admin database.
```
docker@minikube:~$ ls /tmp/hostpath-provisioner/default/mongodump/13102023-002517/admin
system.users.bson  system.users.metadata.json  system.version.bson  system.version.metadata.json
```
### Getting into test database.
```
docker@minikube:~$ ls /tmp/hostpath-provisioner/default/mongodump/13102023-002517/test 
todos.bson  todos.metadata.json
```
**Note:** We can see that the system.users.bson, todos.bson are the collections of the database.

### 15.If we are doubtful, let’s get into the MongoDB pod and check if that collection exists.
```
ubuntu-dsbda@ubuntudsbda-virtual-machine:~$ kubectl exec -it mongo-0 -- mongo
MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("3f61bfb2-0446-424d-bfa3-24c129e8cf2d") }
MongoDB server version: 4.0.8
Server has startup warnings: 
2023-10-13T15:05:51.553+0000 I STORAGE  [initandlisten] 
2023-10-13T15:05:51.553+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger 
storage engine
2023-10-13T15:05:51.553+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-10-13T15:06:10.629+0000 I CONTROL  [initandlisten] 
2023-10-13T15:06:10.629+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2023-10-13T15:06:10.629+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2023-10-13T15:06:10.629+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not 
recommended.
2023-10-13T15:06:10.629+0000 I CONTROL  [initandlisten] 
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

rs0:SECONDARY> show dbs
2023-10-13T15:26:47.845+0000 E QUERY    [js] Error: listDatabases failed:{
	"operationTime" : Timestamp(1697210798, 1),
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk",
	"$clusterTime" : {
		"clusterTime" : Timestamp(1697210798, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:139:1
shellHelper.show@src/mongo/shell/utils.js:882:13
shellHelper@src/mongo/shell/utils.js:766:15
@(shellhelp2):1:1
```
**Note:**To resolve above error do as shown below:
```
rs0:SECONDARY> rs.slaveOk()
```
### 16.Now try to list down the databases in the mongodb:
```
rs0:SECONDARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
```
**Note:** We can see the admin and test databases in the databases list:

### 17.Now try to list down the collections in the database as we have created collections in test not in admin database.
```
rs0:SECONDARY> use admin
switched to db admin
rs0:SECONDARY> show collections

rs0:SECONDARY> use test
switched to db test
rs0:SECONDARY> show collections
todos
```
**Note:** We can clearly see that todos collection is there in test database.Now we have taken backup for admin,test 
databases using jobs.

### 18.Now try to list down the jobs again in cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
No resources found in default namespace.
```
**Note:** As we can see, the job is deleted automatically as specified by the `ttlSecondsAfterFinished: 60` setting. 
This means the job is deleted automatically after the given number of seconds. Not only the job, but the associated 
pod is also deleted.

### 19.Now verify this by listing the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   1          16h
mongo-1   1/1     Running   1          16h
mongo-2   1/1     Running   1          16h
```
**Note:** We can see the pod created by running the job also got deleted.

You might be wondering if we could take this database backup with a simple pod. However, using jobs provides several advantages, such as:

- **Retry Mechanism:** Jobs can automatically retry if they fail, improving reliability.
- **Parallelism:** Jobs allow for parallel execution of tasks, speeding up processes.
- **Controlled Execution:** Jobs offer better control over execution and completion, compared to a single pod.

**Note:** Homework - Try parallesim activedeadlineSeconds.

### 20.Create home-work.yaml.
```
ubuntu@balasenapathi:~$ nano home-work.yaml 
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 50
  completions: 3
  parallelism: 2
  template:
    spec:
      containers:
        - name: example-container
          image: alpine
          command: ["sh", "-c", "echo 'Started' && sleep 10 && exit 1"]
```
### 21.To get the list of jobs in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
No resources found in default namespace.
```
### 22.Apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f home-work.yaml
The Job "example-job" is invalid: spec.template.spec.restartPolicy: Required value: valid values: "OnFailure", "Never"
```
**Note:** The error message you're encountering is because the `restartPolicy` field is missing in your Job's pod template
specification. The `restartPolicy` field is required, and it should have one of the valid values, which are `"OnFailure"`
or `"Never"`.

You can fix this issue by adding the `restartPolicy` field to your Job definition. Since you want your Job to run to 
completion and not be restarted automatically, you should set the `restartPolicy` to `"Never."` 

### 22.Make the changes in the home-work.yaml.
```
ubuntu@balasenapathi:~$ nano home-work.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 50
  completions: 3
  parallelism: 2
  template:
    spec:
      restartPolicy: Never  # Add this line
      containers:
        - name: example-container
          image: alpine
          command: ["sh", "-c", "echo 'Started' && sleep 10 && exit 1"]
```          
It looks like you have created a Kubernetes Job definition in a YAML file named `home-work.yaml`. This Job is configured with certain parameters, and it specifies a container that runs in the Job.

Here's an explanation of the key settings in your Job definition:

- **`backoffLimit: 3`**: This setting specifies the number of retries the Job should make before considering it as failed. In this case, the Job will retry up to 3 times.

- **`activeDeadlineSeconds: 50`**: This sets a deadline for the Job to complete. If the Job doesn't complete within 50 seconds, it will be terminated.

- **`completions: 3`**: This specifies that you want to run the task (the container in this case) 3 times successfully.

- **`parallelism: 2`**: This controls how many pods can run concurrently. In this case, you've set it to 2, meaning that up to 2 pods can run in parallel.

- **`containers`**: This is where you define the container that should run in the Job. It uses the Alpine Linux image, and the command in the container is `echo 'Started' && sleep 10 && exit 1`, which will print "Started," sleep for 10 seconds, and then exit with a non-zero status code (1), simulating a failed job.

- **`restartPolicy: Never`**: Since you want your Job to run to completion and not be restarted automatically, you should set the `restartPolicy` to `"Never."`

When you apply this configuration using `kubectl apply -f home-work.yaml`, Kubernetes will create the Job, and it will 
attempt to run 3 pods in parallel, with a maximum runtime of 50 seconds. If any of the pods fail, they will be retried 
up to 3 times.

Please note that the actual behavior of the Job depends on your Kubernetes cluster's resources and scheduling, so make sure 
you have enough resources available to run the specified number of pods in parallel.

### 23.Now apply the changes in cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f home-work.yaml
job.batch/example-job created
```
### 24.Now list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                READY   STATUS              RESTARTS   AGE
example-job-7kkzd   0/1     ContainerCreating   0          8s
example-job-h4vgm   0/1     ContainerCreating   0          8s
mongo-0             1/1     Running             1          16h
mongo-1             1/1     Running             1          16h
mongo-2             1/1     Running             1          16h

ubuntu@balasenapathi:~$ kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
example-job-7kkzd   1/1     Running   0          23s
example-job-h4vgm   1/1     Running   0          23s
mongo-0             1/1     Running   1          16h
mongo-1             1/1     Running   1          16h
mongo-2             1/1     Running   1          16h
```
**Note:** We can see that the pods example-job-7kkzd, example-job-h4vgm  are running in the cluster.

### 25.Now get the logs of the two pods.
```
ubuntu@balasenapathi:~$ kubectl logs example-job-7kkzd 
Started

ubuntu@balasenapathi:~$ kubectl logs example-job-h4vgm
Started
```
**Note:** We can see that the both pods are started.

### 26.Now list down the jobs in cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME          COMPLETIONS   DURATION   AGE
example-job   0/3           111s       112s
```
### 27.To get the list of pod.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME                READY   STATUS    RESTARTS   AGE
example-job-7kkzd   0/1     Error     0          2m3s
example-job-h4vgm   0/1     Error     0          2m3s
mongo-0             1/1     Running   1          16h
mongo-1             1/1     Running   1          16h
mongo-2             1/1     Running   1          16h
```
It appears that the Job `example-job` is running as expected. Here's what happened:

1. Initially, the Job created two pods with the names `example-job-7kkzd` and `example-job-h4vgm`.

2. The pods started, and you can see from the logs that they printed "Started."

3. After the 10-second sleep, the pods exited with a non-zero status code, as specified in your container's command.

4. The Job automatically attempted to restart the pods to reach the specified `completions` count of 3.

5. The pods continued to start, print "Started," and then exit with an error.

6. The Job kept retrying, and you can see this in the output of `kubectl get jobs`. The "COMPLETIONS" count remained at 0/3 because the pods were failing.

7. The pods continued to cycle through these states with the Job retrying them.

From the logs, it's clear that the pods are indeed running and executing the specified command. They are exiting with a non-zero status code because of the `exit 1` command, which indicates a failure. The Job is working as expected and retrying the pods as per the backoff limit you've set.

**Note:** As we can see, the jobs run using the `home-work.yaml` manifest file are running correctly as expected.

### 28.To stop this Job from creating more pods or to check the final status of the Job, you can delete the Job.
```
ubuntu@balasenapathi:~$ kubectl delete job example-job
job.batch "example-job" deleted
```
### 29.To list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   1          17h
mongo-1   1/1     Running   1          17h
mongo-2   1/1     Running   1          17h
```
### 30.This will terminate the Job, and you can then check its final status using `kubectl get jobs`.
```
ubuntu@balasenapathi:~$ kubectl get jobs
No resources found in default namespace.
```

**CronJobs:**

Now that we have created the job that takes database backups, let's write a simple CronJob to take database backups on a
regular schedule instead of just once.

### 1.Now create a cronjob manifest in cluster.
```
ubuntu@balasenapathi:~$ nano cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mongodb-backup-cronjob
spec:
  schedule: "* * * * *"
  concurrencyPolicy: Allow
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      ttlSecondsAfterFinished: 60 # delete job after 60 seconds
      template:
        spec:
          containers:
            - name: mongodb
              image: mongo
              command: ["/bin/sh", "-c"]
              args:
                [
                  'mongodump --uri "mongodb://admin:${MONGO_PASSWORD}@mongo-0.mongo.default.svc.cluster.local:
                  27017,mongo-1.mongo.default.svc.cluster.local:27017,mongo-2.mongo.default.svc.cluster.local:27017/?
                  replicaSet=rs0&readPreference=secondaryPreferred&authSource=admin&ssl=false" -o /usr/share/mongodump/$(date +"%d%m%Y-
                  %H%M%S")',
                ]
              volumeMounts:
                - mountPath: "/usr/share/mongodump/"
                  name: mongodump
              env:
                - name: MONGO_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      key: password
                      name: mongodb-secret
          volumes:
            - name: mongodump
              persistentVolumeClaim:
                claimName: mongodump
          restartPolicy: Never
```
**Note:** We have given:

- `apiVersion: batch/v1`
- `kind: CronJob`
- `name: mongodb-backup-job` — the name of the job we are creating.
- `schedule: "* * * * *"` — the cron expression is the heart of the CronJob. It defines the time-based schedule for running recurring jobs.

**Note:** We can use online tools like Crontab Guru to get the cron expression. Here `"* * * * *"` means the CronJob runs every minute. If you want the CronJob to run every midnight, replace the first two stars with `00`, i.e., `"0 0 * * *"`, which means it runs every day at midnight. In our case, the CronJob runs every minute.

**CronJobs Concurrency Policy:**

We have discussed that CronJobs create jobs based on a schedule. The concurrency policy determines the number of concurrent
job instances allowed to run at any given time. By default, a CronJob will create only one job instance at a time. If a job
is still running when the next scheduled run time arrives, the new job instance will be queued until the first one completes.
However, you can set the concurrency policy field in a CronJob to control how concurrent jobs are handled. 

The available options are:

- **Allow:** This option allows multiple job instances to run concurrently. It is useful for handling overlapping tasks or if you want to run multiple instances of the same job in parallel.

- **Forbid:** This option prevents any new job instances from starting while an existing one is still running.

- **Replace:** This option stops the currently running job and creates a new job instance. 

The default concurrency policy is **Forbid**.

- `concurrencyPolicy: Allow` — we are setting the concurrency policy to `Allow` for now.
- `successfulJobsHistoryLimit: 2` — this limit specifies how many completed jobs should be stored for reference in the future. With a limit of 2, only the two most recent completed jobs will be kept; older jobs will be automatically deleted.
- `failedJobsHistoryLimit: 3` — this limit is similar to `successfulJobsHistoryLimit` but stores the 3 most recent failed jobs.

`template:` — This is the template of the job because the CronJob ultimately creates the job. This is similar to the job manifest we have seen before. Here we provide the container information, just like we do in a Pod.

- `image: mongo` — We are using the MongoDB image, and in this container, we are running the `mongodump` command.

**Note:** `mongodump` is a utility that creates a binary export of our database.
```
'mongodump --uri "mongodb://admin:${MONGO_PASSWORD}@mongo-0.mongo.default.svc.cluster.local:27017,mongo-1.mongo.default.svc.cluster.local:27017,mongo-2.mongo.default.svc.cluster.local:27017/?replicaSet=rs0&readPreference=secondaryPreferred&authSource=admin&ssl=false" -o /usr/share/mongodump/$(date +"%d%m%Y-%H%M%S")'
```
**The above is the connection string of our database:**

- `admin` is the username.
- `${MONGO_PASSWORD}` is taken from a secret.
- `@mongo-0` is our first pod name.
- `mongo` is our headless service.
- `default` is the namespace.
- `svc.cluster.local` is the service DNS.
- `27017` is the port number on which our MongoDB is running.
- `@mongo-1` is our second pod name.
- `@mongo-2` is our third pod name.
- `/usr/share/mongodump/$(date +"%d%m%Y-%H%M%S")` — Finally, we are giving the path to which the exported dump should be saved in the container.

**Volume Mounts:**

- `mountPath: "/usr/share/mongodump/"`
- `name: mongodump`

To store this volume, we are mounting this volume at `/usr/share/mongodump/` with the name `mongodump`.

**Volumes:**

- `name: mongodump`
- `persistentVolumeClaim:`
  - `claimName: mongodump`

**restartPolicy: Never** — As we are using `mongodump` with PersistentVolumeClaim.

### 2.Now apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f cronjob.yaml
cronjob.batch/mongodb-backup-cronjob created
```
**Note:** As the cronjob is created.

### 3.Now list down the all the cronjobs in cluster.
```
ubuntu@balasenapathi:~$ kubectl get cj
NAME                     SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mongodb-backup-cronjob   * * * * *   False     0        <none>          13s
```
**Note:** The cj is shortform of cronjobs and our schedule is "* * * * * " the job runs for every second.

### 4. Now list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                    READY   STATUS              RESTARTS   AGE
mongo-0                                 1/1     Running             1          20h
mongo-1                                 1/1     Running             1          20h
mongo-2                                 1/1     Running             1          20h
mongodb-backup-cronjob-28287113-2gqdf   0/1     ContainerCreating   0          16s

ubuntu@balasenapathi:~$ kubectl get pods -w
NAME                                    READY   STATUS              RESTARTS   AGE
mongo-0                                 1/1     Running             1          20h
mongo-1                                 1/1     Running             1          20h
mongo-2                                 1/1     Running             1          20h
mongodb-backup-cronjob-28287113-2gqdf   0/1     ContainerCreating   0          76s
mongodb-backup-cronjob-28287114-bw7wk   0/1     ContainerCreating   0          16s
mongodb-backup-cronjob-28287113-2gqdf   1/1     Running             0          107s
mongodb-backup-cronjob-28287114-bw7wk   1/1     Running             0          47s
mongodb-backup-cronjob-28287114-bw7wk   0/1     Completed           0          52s
```
**Note:** After every one minute scheduled time our job is running as specified/scheduled.

### 5.Now list down the jobs.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287114   1/1           55s        113s
mongodb-backup-cronjob-28287115   1/1           8s         53s
```
**Note:** Our jobs are being created according to the schedule and are completing in a timely manner. They are being 
deleted after completion, and new jobs are being created as per the defined schedule.

### 6.Now verify the list of pods in cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                    READY   STATUS              RESTARTS   AGE
mongo-0                                 1/1     Running             1          20h
mongo-1                                 1/1     Running             1          20h
mongo-2                                 1/1     Running             1          20h
mongodb-backup-cronjob-28287115-mk3we   0/1     Completed           0          63s
mongodb-backup-cronjob-28287116-bk6kv   0/1     Completed           0          3s
```
### 7.Now get the list of jobs in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287115   1/1           7s         10s
mongodb-backup-cronjob-28287116   1/1           8s         53s
```
**Note:** Now comment #ttlSecondsAfterFinished: 60 # delete job after 60 seconds in cronjob.yaml maniifest and apply this again:

### 8.Now apply the changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f cronjob.yaml
cronjob.batch/mongodb-backup-cronjob configured
```
### 9.Now list down the jobs in cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287130   1/1           6s         72s
mongodb-backup-cronjob-28287131   1/1           11s        12s
```
### 10.Now get into the minikube and see whether data is being exported to the file specified i.e, mongodump:
```
ubuntu@balasenapathi:~$ minikube ssh
docker@minikube:~$ ls /tmp/hostpath-provisioner/default/mongodump
13102023-002517  13102023-195603  13102023-195904  13102023-200203  13102023-200503  13102023-200804  13102023-201108
13102023-195447  13102023-195703  13102023-200004  13102023-200303  13102023-200603  13102023-200904  13102023-201208
13102023-195505  13102023-195803  13102023-200103  13102023-200404  13102023-200703  13102023-201003
```
**Note:** We can see multiple folders are getting created because for every one minute a new job is getting created.

### 11.Now list down the jobs.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287132   1/1           12s        98s
mongodb-backup-cronjob-28287133   1/1           7s         38s
```
**Note:** We can observe that old jobs are being deleted, and in the job history, we configured the system to display 
only the two most recent jobs in the manifest by setting `successfulJobsHistoryLimit: 2`.

### 12.We can also suspend the CronJob to stop creating jobs temporarily. To do this, edit the CronJob.
```
ubuntu@balasenapathi:~$ kubectl edit mongodb-backup-cronjob

or

ubuntu@balasenapathi:~$ kubectl patch cj mongodb-backup-cronjob --patch '{"spec":{"suspend":true}}'
cronjob.batch/mongodb-backup-cronjob patched
```
**Note:** We can see that the cronjob is patched.

### 13.Jobs created before applying patching
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287139   1/1           6s         94s
mongodb-backup-cronjob-28287140   1/1           6s         34s

ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287139   1/1           6s         3m6s
mongodb-backup-cronjob-28287140   1/1           6s         2m6s
```
**Note:** We can see that no new jobs were created after applying patching because we have suspended the cronjob.

### 14.Now get the last schedule of the cronjob.
```
ubuntu@balasenapathi:~$ kubectl get cj
NAME                     SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mongodb-backup-cronjob   * * * * *   True      0        4m14s           31m
```
**Note:** The last schedule was done at 4m14s till now no new pod is created because we have suspended the cronjob.

### 15.Now lets try to rollback the suspend patch so that the pods will be created normally.
```
ubuntu@balasenapathi:~$ kubectl patch cj mongodb-backup-cronjob --patch '{"spec":{"suspend":false}}'
cronjob.batch/mongodb-backup-cronjob patched
```
### 16.To get the list of CronJobs.
```
ubuntu@balasenapathi:~$ kubectl get cj
NAME                     SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mongodb-backup-cronjob   * * * * *   True      0        7m5s            34m
```
### 17.To get the list of jobs in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287139   1/1           6s         8m16s
mongodb-backup-cronjob-28287140   1/1           6s         7m16s
```
### 18.To get the list of pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                    READY   STATUS      RESTARTS   AGE
mongo-0                                 1/1     Running     1          21h
mongo-1                                 1/1     Running     1          21h
mongo-2                                 1/1     Running     1          21h
mongodb-backup-cronjob-28287139-nn6n7   0/1     Completed   0          8m25s
mongodb-backup-cronjob-28287140-kfxtl   0/1     Completed   0          7m25s

ubuntu@balasenapathi:~$ kubectl patch cj mongodb-backup-cronjob --patch '{"spec":{"suspend":false}}'
cronjob.batch/mongodb-backup-cronjob patched

ubuntu@balasenapathi:~$ kubectl get jobs
NAME                              COMPLETIONS   DURATION   AGE
mongodb-backup-cronjob-28287147   1/1           7s         33s
mongodb-backup-cronjob-28287148   1/1           6s         23s
```

### 19.Now, to delete the CronJob we created: when we delete the CronJob, the associated jobs will also be deleted.
```
ubuntu@balasenapathi:~$ kubectl delete cj mongodb-backup-cronjob
cronjob.batch "mongodb-backup-cronjob" deleted

ubuntu@balasenapathi:~$ kubectl get jobs
No resources found in default namespace.

ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   1          21h
mongo-1   1/1     Running   1          21h
mongo-2   1/1     Running   1          21h
```










