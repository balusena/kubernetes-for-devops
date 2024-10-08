# kubernetes Probes

We have learned that a pod is restarted automatically if Kubernetes finds it in an unhealthy state. But how 
does Kubernetes know if a pod is healthy or not? Now, we will learn how Kubernetes identifies whether a pod 
is functioning correctly and how we can customize this behavior using probes.

# Reasons for any application to be unhealthy.
Any application can be in an unhealthy state due to various reasons like:
- 1.Bugs in the code
- 2.Timeouts while communicating with external service.
- 3.DataBase connection failure
- 4.Out Of Memory issues
- 5.etc

![Reasons for unhealthy application](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/appllication_unhealthy.png)

In all these situations, the pod may appear to be running from the outside, but its internal functionality
could be broken due to bugs, preventing users from accessing the application. In such cases, we expect the
container to restart. However, Kubernetes doesn’t restart the container because, by default, Kubernetes 
only checks the container's main process to determine if the container is healthy. It doesn't check the 
internal functionality of our application. Kubernetes will only restart the container if the main process
crashes.

If we don’t address these unhealthy pods, our service can become unstable. Debugging such pods is also 
tricky because the pod status shows as "Running," yet we don’t get the expected output. In the MongoDB 
pods we’ve deployed so far, mongod is the main process. As long as the mongod process is running, 
Kubernetes treats the pod as healthy, regardless of whether the internal functionality is working.

How do we handle scenarios where pods aren’t working correctly, but they aren’t being restarted either? 
What if there was a way for us to signal to Kubernetes that a pod is no longer functioning as expected 
and should be restarted? This is where Kubernetes probes come into play.

# kubernetes Probes and types
Kubernetes probes are mechanisms used to determine the health and readiness of a container running within 
a pod.These probes help Kubernetes make decisions about when to restart a container,when to route traffic
to it,or when to take it out of rotation for maintenance or updates. 

**There are three types of probes in Kubernetes:**

![Probe_Types](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/probe_types.png)
- 1.Liveness Probe
- 2.Readiness Probe
- 3.Startup Probe

## 1.Liveness Probe
- **Purpose:** Determines if a container is still running. If the liveness probe fails, Kubernetes will restart the container.

- **Use Case:** Ensures that a container that has encountered a deadlock or is stuck in an irrecoverable state is automatically restarted.

![Liveness Probe](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/liveness_probe.png)

### 1.Use the statefulset to test the livenessprobe in local-cluster:
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
          livenessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
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
            - "--replSet"
            - "--config=/etc/mongo/mongodb.conf"
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
**Note:** We define probes at the container level, not at the pod level. If we understand how to define the liveness probe,
other probes will be similar. This will instruct Kubernetes to check the liveness of our container.

## Probing Mechanisms.
Kubernetes provides three probing mechanisms to assess the health and readiness of containers within a pod:

![Probing Mechanisms](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/probing_mechanisms.png)

- 1.Exec
- 2.HTTP
- 3.TCP

### 1.Exec
In Kubernetes, when using an Exec Probe, the health of a container is determined by the exit code of the command executed inside the container:

- Exit Code 0: The container is considered healthy.
- Non-Zero Exit Code (e.g., 1): The container is considered unhealthy.

### 2.HTTP
In Kubernetes, the HTTP GET Probe mechanism works as follows:

- HTTP Response Code in the 200-399 Range: Kubernetes considers the container to be healthy.
- HTTP Response Code Outside the 200-399 Range: Kubernetes considers the container to be unhealthy.

The probe makes an HTTP GET request to the specified URL within the container, and the container must respond with a 
status code in the 200-399 range to be deemed healthy. Any other status code indicates an unhealthy state.

## 3.TCP
In Kubernetes, the TCP Socket Probe mechanism works as follows:

- Success: Kubernetes considers the container healthy if it can establish a TCP connection to the specified port on the container.
- Failure: Kubernetes considers the container unhealthy if it cannot establish the TCP connection to the specified port.

This probe simply attempts to connect to a given port on the container, and if the connection is successful, the container 
is considered healthy. If the connection cannot be established, the container is considered unhealthy.

You can also make gRPC health check requests to a port inside the container, and use the results to determine whether 
the probe succeeded.

**Note:** In the statefulset.yaml file, we are running a simple command in "spec" under "exec" section to check if our 
container is healthy.
```
spec:
      containers:
        - name: mongo
          image: mongo:4.0.8
          livenessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db1.adminCommand('ping')" 
```
### 2.For our mongo deployment lets make a simple change in statefulset.yaml file and check if our container is healthy or not.
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
          livenessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db1.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
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
            - "--replSet"
            - "--config=/etc/mongo/mongodb.conf"
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
As we discussed, to run a command, we should use the exec probing mechanism, where we can execute the ping command. 
Essentially, this defines a liveness probe with the command to execute. However, how frequently Kubernetes runs this 
command to check the container's health or how many times it should retry if the command fails can be further customized
using four parameters.

## Probing Customization.
To customize Kubernetes probing, you can adjust four key parameters: initial delay, period, timeout, and failure threshold.
These settings determine how frequently health checks occur, how long they wait, and how many failures trigger container
restarts or other actions.

![Probing Customization](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/probing_customization.png)

**These four parameters are:**

- 1. initialDelaySeconds
- 2. periodSeconds
- 3. timeoutSeconds
- 4. failureThreshold

### 1.initialDelaySeconds:
With initialDelaySeconds, we specify how long Kubernetes should wait after the container is started before executing the
probe. Sometimes, our service may start a little late, and if we are aware of this, we can delay the probe execution by 
setting this value. The default value is 0 seconds, which means the probe is executed immediately.

### 2.periodSeconds:
With periodSeconds, we can define how frequently the probe will be executed after the initial delay. This is very helpful
if we want to check our pod periodically. The default value is 10 seconds, meaning the probe is executed every 10 seconds,
and if it fails, the container is restarted.

### 3.timeoutSeconds:
With timeoutSeconds, each probe will time out and be marked as a failure after this many seconds. The default value is 
1 second.

### 4.failureThreshold
With failureThreshold, we can instruct Kubernetes to retry the probe this many times after the first failure. The container
will only be restarted if the probe fails a number of times equal to the failure threshold. The default value is 3, 
meaning the probe is considered failed if it fails 3 times.

### 5.successThreshold:
With successThreshold, Kubernetes determines how many successful probes are needed to consider the container healthy 
after a failure. The default value is 1, meaning one successful probe is sufficient.

In summary, the liveness probe runs commands, fires HTTP calls, or checks ports defined by us to ensure the container is
running as expected. If Kubernetes detects that the container is not functioning correctly, the kubelet restarts the container.

**Note:** Give the below values in the statefulset.yaml to check the container health using livenessprobe.
```
livenessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db1.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
```
**Note:** We are intentionally using - "db1.adminCommand('ping')" in the exec section to test the container.

### 3.Now apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo configured
```
### 4.To list all the pods in a cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS     AGE
mongo-0   1/1     Running   0            8s
mongo-1   1/1     Running   0            18s
mongo-2   1/1     Running   0            25s
```
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS      AGE
mongo-0   1/1     Running   2 (4s ago)    17s
mongo-1   1/1     Running   2 (6s ago)    27s
mongo-2   1/1     Running   2 (12s ago)   34s
```
```
ubuntu@balasenapathi:~$ kubectl get pods 
NAME      READY   STATUS             RESTARTS      AGE
mongo-0   1/1     Running            5 (13s ago)   2m15s
mongo-1   0/1     CrashLoopBackOff   4 (37s ago)   2m19s
mongo-2   1/1     Running            5 (12s ago)   2m23s
```
```
ubuntu@balasenapathi:~$ kubectl get pods 
NAME      READY   STATUS             RESTARTS      AGE
mongo-0   0/1     CrashLoopBackOff   5 (11s ago)   2m34s
mongo-1   1/1     Running            5 (56s ago)   2m38s
mongo-2   0/1     CrashLoopBackOff   5 (10s ago)   2m42s
```
**Note:** We can clearly see the difference that all the pods have restarted 2 times. This is due to the liveness probe.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS             RESTARTS       AGE
mongo-0   0/1     CrashLoopBackOff   6 (79s ago)    6m1s
mongo-1   0/1     CrashLoopBackOff   6 (90s ago)    6m11s
mongo-2   0/1     CrashLoopBackOff   6 (116s ago)   6m18s
```
### 5.To describe the pod and get all the information about the pod.
```
ubuntu@balasenapathi:~$ kubectl describe pod mongo-0
Name:             mongo-0
Namespace:        default
Priority:         0
Service Account:  default
Node:             local-cluster/192.168.49.2
Start Time:       Tue, 26 Sep 2023 21:31:43 +0530
Labels:           app=mongo
                  controller-revision-hash=mongo-6bddb44dfb
                  statefulset.kubernetes.io/pod-name=mongo-0
Annotations:      <none>
Status:           Running
IP:               10.244.0.27
IPs:
  IP:           10.244.0.27
Controlled By:  StatefulSet/mongo
Containers:
  mongo:
    Container ID:  docker://2e9d6c985b9d1ec15300e3e5f4d1e7f07aecc7157afee115a50f2adca12d2a37
    Image:         mongo:4.0.8
    Image ID:      docker-pullable://mongo@sha256:1dd59670959c3f774c5056e5179145ee9cc06f9003663e14fa44326426a4e6f7
    Port:          <none>
    Host Port:     <none>
    Command:
      mongod
      --bind_ip_all
      --replSet
      --config=/etc/mongo/mongodb.conf
    State:          Running
      Started:      Tue, 26 Sep 2023 21:32:05 +0530
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 26 Sep 2023 21:31:44 +0530
      Finished:     Tue, 26 Sep 2023 21:32:04 +0530
    Ready:          True
    Restart Count:  1
    Liveness:       exec [mongo --eval db1.adminCommand('ping')] delay=1s timeout=5s period=10s #success=1 #failure=2
    Environment:
      MONGO_INITDB_ROOT_USERNAME:  <set to the key 'username' of config map 'mongodb-config'>  Optional: false
      MONGO_INITDB_ROOT_PASSWORD:  <set to the key 'password' in secret 'mongodb-secret'>      Optional: false
    Mounts:
      /data/db from mongo-volume (rw)
      /etc/mongo from mongodb-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-k755p (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  mongo-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mongo-volume-mongo-0
    ReadOnly:   false
  mongodb-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      mongodb-config
    Optional:  false
  kube-api-access-k755p:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  42s   default-scheduler  Successfully assigned default/mongo-0 to local-cluster
  Warning  Unhealthy  31s   kubelet            Liveness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("522330b6-f8dc-4d0d-bbe8-8f129a1a91ce") }
MongoDB server version: 4.0.8
2023-09-26T16:01:54.419+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  21s  kubelet  Liveness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("3ffac8ea-b7ed-4e22-9ab6-870079706e5c") }
MongoDB server version: 4.0.8
2023-09-26T16:02:04.423+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  11s  kubelet  Liveness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7dde325d-206a-4fc4-a468-24942444edda") }
MongoDB server version: 4.0.8
2023-09-26T16:02:14.410+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Normal   Killing    1s (x2 over 21s)  kubelet  Container mongo failed liveness probe, will be restarted
  Warning  Unhealthy  1s                kubelet  Liveness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("17e11667-665d-4dd7-9f23-0bdb9f84772f") }
MongoDB server version: 4.0.8
2023-09-26T16:02:24.407+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Normal  Pulled   0s (x3 over 41s)  kubelet  Container image "mongo:4.0.8" already present on machine
  Normal  Created  0s (x3 over 41s)  kubelet  Created container mongo
  Normal  Started  0s (x3 over 41s)  kubelet  Started container mongo
```
**Note:** In the events, we can see that the liveness probe has failed. As we discussed, if the liveness probe fails, 
the pod is automatically restarted. The reason this liveness probe failed is because db1 is not defined; we intentionally
set it to db1 instead of db to cause the failure. The probe failed twice because it is set to execute every 10 seconds, 
as configured with "periodSeconds: 10". The pod will continue to restart and eventually enter a CrashLoopBackOff state 
until we fix db1 to db.

### 6.Correct the db1 to db in statefulset.yaml and apply the changes in local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo configured
```
### 7. To list all the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods 
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          32s
mongo-1   1/1     Running   0          30s
mongo-2   1/1     Running   0          28s
```
**Note:** The pods are now running successfully without any restarts.

So, as a summary, the liveness probe executes initially after a 1-second delay. If it fails 2 times, the probe is marked
as a failure, and the container is restarted. This process will continue every 10 seconds.

## 2.Readiness Probe:
- **Purpose:** Determines if a container is ready to start accepting traffic. If the readiness probe fails, Kubernetes will stop sending requests to that container until it passes.

- **Use Case:** Useful when your application needs some time to warm up or load configurations before it can handle requests.

![Readiness Probe](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/readiness_probe.png)

Kubernetes will remove the IP address of the pod from the endpoints of all the services it belongs to, and when the pod
is not part of the service, it will not receive any traffic. This probe is very helpful in instructing Kubernetes that a
running container should not receive any traffic until it is ready. This way, we can increase our success response rate.
The difference between a liveness probe and a readiness probe is that when a liveness probe fails, the pod is restarted, 
whereas when the readiness probe fails, the pod is not restarted. Instead, it is removed from the endpoint list of the 
service, so it will not receive any traffic. Once the readiness probe succeeds, the pod is added back to the service and
begins receiving traffic as usual.

**Note:** Similar to the liveness probe, we can add a readiness probe. In the statefulset.yaml file, we are running a 
simple command in the spec section under the exec block to check if our container is healthy.

```
readinessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db1.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2  
```
### 1.Use the statefulset to test the livenessprobe in local-cluster.
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
          livenessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
          readinessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db1.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2  
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
            - "--replSet"
            - "--config=/etc/mongo/mongodb.conf"
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
### 2.Now apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo configured
```
### 3.To get the list of pods with a watch flag.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          67m
mongo-1   1/1     Running   0          67m
mongo-2   0/1     Running   0          4s
```
**Note:** We can clearly observe mongo-2 0/1 this is because readinessprobe is failed this container is not reday to accept the traffic.

### 4.We can check this by decribing the mongo-2 pod.
```
ubuntu@balasenapathi:~$ kubectl describe pod mongo-2
Name:             mongo-2
Namespace:        default
Priority:         0
Service Account:  default
Node:             local-cluster/192.168.49.2
Start Time:       Tue, 26 Sep 2023 22:45:21 +0530
Labels:           app=mongo
                  controller-revision-hash=mongo-74f9484d6d
                  statefulset.kubernetes.io/pod-name=mongo-2
Annotations:      <none>
Status:           Running
IP:               10.244.0.31
IPs:
  IP:           10.244.0.31
Controlled By:  StatefulSet/mongo
Containers:
  mongo:
    Container ID:  docker://31dba2b35ef73fb42d6783b6b12ad04c348fb10f547bf8b1bc41690ed449ca6c
    Image:         mongo:4.0.8
    Image ID:      docker-pullable://mongo@sha256:1dd59670959c3f774c5056e5179145ee9cc06f9003663e14fa44326426a4e6f7
    Port:          <none>
    Host Port:     <none>
    Command:
      mongod
      --bind_ip_all
      --replSet
      --config=/etc/mongo/mongodb.conf
    State:          Running
      Started:      Tue, 26 Sep 2023 22:45:23 +0530
    Ready:          False
    Restart Count:  0
    Liveness:       exec [mongo --eval db.adminCommand('ping')] delay=1s timeout=5s period=10s #success=1 #failure=2
    Readiness:      exec [mongo --eval db1.adminCommand('ping')] delay=1s timeout=5s period=10s #success=1 #failure=2
    Environment:
      MONGO_INITDB_ROOT_USERNAME:  <set to the key 'username' of config map 'mongodb-config'>  Optional: false
      MONGO_INITDB_ROOT_PASSWORD:  <set to the key 'password' in secret 'mongodb-secret'>      Optional: false
    Mounts:
      /data/db from mongo-volume (rw)
      /etc/mongo from mongodb-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w26r7 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  mongo-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mongo-volume-mongo-2
    ReadOnly:   false
  mongodb-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      mongodb-config
    Optional:  false
  kube-api-access-w26r7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  3m5s  default-scheduler  Successfully assigned default/mongo-2 to local-cluster
  Normal   Pulled     3m4s  kubelet            Container image "mongo:4.0.8" already present on machine
  Normal   Created    3m4s  kubelet            Created container mongo
  Normal   Started    3m3s  kubelet            Started container mongo
  Warning  Unhealthy  3m2s  kubelet            Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
2023-09-26T17:15:24.796+0000 E QUERY    [js] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed: 
SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused :
connect@src/mongo/shell/mongo.js:343:13
@(connect):2:6
exception: connect failed
  Warning  Unhealthy  3m  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
2023-09-26T17:15:26.157+0000 E QUERY    [js] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed: 
SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused :
connect@src/mongo/shell/mongo.js:343:13
@(connect):2:6
exception: connect failed
  Warning  Unhealthy  2m54s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("eab6e07b-e5e3-4b3d-8c74-be7962b34312") }
MongoDB server version: 4.0.8
2023-09-26T17:15:32.596+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  2m44s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("31bca676-f28a-4fe2-a441-eed11886e997") }
MongoDB server version: 4.0.8
2023-09-26T17:15:42.301+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  2m34s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("26efbe82-d82b-4262-806a-bd9c7fd2af1e") }
MongoDB server version: 4.0.8
2023-09-26T17:15:52.555+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  2m24s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("b40e7ec5-6607-4c70-97a3-fb1b6c6e5c0e") }
MongoDB server version: 4.0.8
2023-09-26T17:16:02.567+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  2m14s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("dda7c1ea-0a2b-4373-9bb3-b3b4d8735600") }
MongoDB server version: 4.0.8
2023-09-26T17:16:12.439+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  2m4s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("d618e4e0-c6c5-4f57-aa1d-2b4281147e93") }
MongoDB server version: 4.0.8
2023-09-26T17:16:22.506+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  114s  kubelet  Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("17f73ecc-4a38-47e2-b36b-228fb29e78de") }
MongoDB server version: 4.0.8
2023-09-26T17:16:32.310+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  4s (x13 over 104s)  kubelet  (combined from similar events): Readiness probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("4a4f97f3-1069-4d05-9da0-df20b0b7ad2f") }
MongoDB server version: 4.0.8
2023-09-26T17:18:22.315+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
```
**Note:** As we can see that the Readiness probe failed,this is because db1 is not defined.

### 5.Now let us try to describe the service and see this mongo-2 pod's IP is deleted from the endpoints list or not:

Now list down all the IP's of the pods:
```
ubuntu@balasenapathi:~$ kubectl get pods -owide
NAME      READY   STATUS    RESTARTS   AGE    IP            NODE            NOMINATED NODE   READINESS GATES
mongo-0   1/1     Running   0          75m    10.244.0.28   local-cluster   <none>           <none>
mongo-1   1/1     Running   0          75m    10.244.0.29   local-cluster   <none>           <none>
mongo-2   0/1     Running   0          8m1s   10.244.0.31   local-cluster   <none>           <none>
```
### 6.Now try to describe the service mongo:
```
ubuntu@balasenapathi:~$ kubectl describe svc mongo
Name:              mongo
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=mongo
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              mongo  27017/TCP
TargetPort:        27017/TCP
Endpoints:         10.244.0.28:27017,10.244.0.29:27017
Session Affinity:  None
Events:            <none>
```
Note: As we can see in the endpoints section, there are two IPs available, but the third pod's IP is missing. This is 
because the readiness probe failed, so this pod was removed from the service.

### 6.Now fix this redainessprobe and see that the pod is gain added back into the local-cluster:

**Note:** In statefulset.yaml file remove the "db1.adminCommand('ping')" and place "db.adminCommand('ping')" in readinessprobe section.

### 7.Now delete the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl delete -f statefulset.yaml
statefulset.apps/mongo deleted
```
### 8.Now apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo created
```
### 9.Now to list all the pods in a local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          21s
mongo-1   1/1     Running   0          17s
mongo-2   1/1     Running   0          7s
```
**Note:** Now we can see all the pods are running and all the containers are reday.

### 10.Now try to describe the service mongo and see all the three pods are available in local-cluster.
```
ubuntu@balasenapathi:~$ kubectl describe svc mongo
Name:              mongo
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=mongo
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              mongo  27017/TCP
TargetPort:        27017/TCP
Endpoints:         10.244.0.32:27017,10.244.0.33:27017,10.244.0.34:27017
Session Affinity:  None
Events:            <none>
```
**Note:** We can see all the 3 pods are available in service in local-cluster.

## 3.Startup Probe:
- **Purpose:** Determines if an application within a container has started. This probe is useful for applications that have slow start-up times.

- **Use Case:** Ideal for ensuring that the application is fully started before allowing liveness and readiness probes to take over.
![Startup Probe](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/startup_probe.png)

The startup probe provides a way to delay the execution of the liveness and readiness probes until a container is ready 
to handle them. This means that the liveness and readiness probes are executed only if the startup probe succeeds. If a
container fails its startup probe, the container is killed, and the pod's restart policy is followed.

**We can set the restart policy in the pod's spec, which defines when the pod should be restarted:**

- **1.Always:**
- This is the default value.
- The pod will always be restarted, regardless of how it exits.
- Use this option to ensure the pod is always up and running, no matter the exit status.

- **2.OnFailure:**
- The pod will be restarted only if it exits with a non-zero status code.
- This is useful when you want the pod to restart only on errors.

- **3.Never:**

- The pod will not be restarted, no matter how it exits.
-This option is used when you do not want the pod to restart under any circumstances.

**Note:** This restart policy applies to every probe. To ensure the pod restarts when necessary, set the restart policy 
to the default value, "Always." You can configure the startup probe in the same way as the liveness and readiness probes.
For simplicity, we are using the same commands for all the probes. However, you can experiment with different commands 
for HTTP calls based on your application's specific health checks.

### 1.Use the statefulset to test the livenessprobe in local-cluster.
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
          startupProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
          livenessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
          readinessProbe:
            exec:
              command:
                - mongo
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 1
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 2
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
### 2.Now delete the statefulset.yaml from cluster.
```
ubuntu@balasenapathi:~$ kubectl delete -f statefulset.yaml
statefulset.apps "mongo" deleted
```
### 3.Now apply changes in the local-cluster.          
```
ubuntu-dsbda@ubuntudsbda-virtual-machine:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo created
```
### 4.Now list down all the pod in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS      AGE
mongo-0   0/1     Running   2 (11s ago)   52s
```
**Note:** We can see that the pod is running but it is not reday yet.
### 5.To get the information about the pod.
```
ubuntu@balasenapathi:~$ kubectl describe pod mongo-0
Name:             mongo-0
Namespace:        default
Priority:         0
Service Account:  default
Node:             local-cluster/192.168.49.2
Start Time:       Tue, 26 Sep 2023 23:37:51 +0530
Labels:           app=mongo
                  controller-revision-hash=mongo-59b9566db4
                  statefulset.kubernetes.io/pod-name=mongo-0
Annotations:      <none>
Status:           Running
IP:               10.244.0.35
IPs:
  IP:           10.244.0.35
Controlled By:  StatefulSet/mongo
Containers:
  mongo:
    Container ID:  docker://4632a3f7ef1828ebb16ee31054ad79eedcc06cf3743c8e71825d7278c6c867ae
    Image:         mongo:4.0.8
    Image ID:      docker-pullable://mongo@sha256:1dd59670959c3f774c5056e5179145ee9cc06f9003663e14fa44326426a4e6f7
    Port:          <none>
    Host Port:     <none>
    Command:
      mongod
      --bind_ip_all
      --config=/etc/mongo/mongodb.conf
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 26 Sep 2023 23:39:13 +0530
      Finished:     Tue, 26 Sep 2023 23:39:32 +0530
    Ready:          False
    Restart Count:  4
    Liveness:       exec [mongo --eval db.adminCommand('ping')] delay=1s timeout=5s period=10s #success=1 #failure=2
    Readiness:      exec [mongo --eval db.adminCommand('ping')] delay=1s timeout=5s period=10s #success=1 #failure=2
    Startup:        exec [mongo --eval db1.adminCommand('ping')] delay=1s timeout=5s period=10s #success=1 #failure=2
    Environment:
      MONGO_INITDB_ROOT_USERNAME:  <set to the key 'username' of config map 'mongodb-config'>  Optional: false
      MONGO_INITDB_ROOT_PASSWORD:  <set to the key 'password' in secret 'mongodb-secret'>      Optional: false
    Mounts:
      /data/db from mongo-volume (rw)
      /etc/mongo from mongodb-config (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-69qfd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  mongo-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mongo-volume-mongo-0
    ReadOnly:   false
  mongodb-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      mongodb-config
    Optional:  false
  kube-api-access-69qfd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age    From               Message
  ----     ------     ----   ----               -------
  Normal   Scheduled  2m23s  default-scheduler  Successfully assigned default/mongo-0 to local-cluster
  Warning  Unhealthy  2m13s  kubelet            Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("25a50384-9713-4d8e-8eb5-525e58a795e3") }
MongoDB server version: 4.0.8
2023-09-26T18:08:01.817+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  2m3s  kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("1c9e19be-35e3-47bf-9ed1-f19eebac7857") }
MongoDB server version: 4.0.8
2023-09-26T18:08:11.780+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  113s  kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("044f0682-b69b-4610-b546-50a409b87c9f") }
MongoDB server version: 4.0.8
2023-09-26T18:08:21.770+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  103s  kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("96ebe9fa-d981-425b-995b-174d5e86e373") }
MongoDB server version: 4.0.8
2023-09-26T18:08:31.756+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  93s  kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("729019d0-c39e-4c03-b0a3-639a58174b46") }
MongoDB server version: 4.0.8
2023-09-26T18:08:41.744+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Warning  Unhealthy  83s  kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("cf83d37b-53c5-4a3b-95ac-25dbe5e176c4") }
MongoDB server version: 4.0.8
2023-09-26T18:08:51.757+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Normal   Started    82s (x4 over 2m21s)  kubelet  Started container mongo
  Normal   Created    82s (x4 over 2m21s)  kubelet  Created container mongo
  Warning  Unhealthy  73s                  kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("d5224c19-1cb1-42a1-bc12-1d5ec3721e98") }
MongoDB server version: 4.0.8
2023-09-26T18:09:01.774+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Normal   Killing    63s (x4 over 2m3s)  kubelet  Container mongo failed startup probe, will be restarted
  Warning  Unhealthy  63s                 kubelet  Startup probe failed: MongoDB shell version v4.0.8
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("dfd7c003-e209-4e03-935e-e0bf47866cab") }
MongoDB server version: 4.0.8
2023-09-26T18:09:11.755+0000 E QUERY    [js] ReferenceError: db1 is not defined :
@(shell eval):1:1
  Normal  Pulled  62s (x5 over 2m22s)  kubelet  Container image "mongo:4.0.8" already present on machine
```
**Note:**We can see that the startup probe failed because db1 is not defined. Since the restartPolicy is set to the 
default value of "Always," the pod is getting restarted.

### 6.Now list down all the pod in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS             RESTARTS      AGE
mongo-0   0/1     CrashLoopBackOff   6 (55s ago)   5m36s
```
**Note:** The pod status has changed to CrashLoopBackOff. This means the pod is repeatedly crashing and entering a state
called CrashLoopBackOff. This status doesn’t mean the kubelet has given up; rather, it indicates that the kubelet is 
increasing the time interval between each restart attempt. After the first crash, the kubelet restarts the pod immediately.
If it crashes again, the kubelet waits 10 seconds before restarting, and if it crashes again, the delay increases to 
20 seconds before the next restart. This delay increases exponentially with each restart attempt, up to a maximum of 
300 seconds. Once the delay reaches the 300-second limit, the kubelet will continue restarting the container indefinitely
every 5 minutes until the pod either stops crashing or is deleted.

- **Now simply correct the error in startup probe:**

### 7.To delete the statefulset in the cluster.
```
ubuntu@balasenapathi:~$ kubectl delete -f statefulset.yaml
statefulset.apps "mongo" deleted
```
### 8.To apply the statefulset in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo created
```
### 9.To get the list of pods running in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          45s
mongo-1   1/1     Running   0          34s
mongo-2   1/1     Running   0          23s
```
**Note:** We can see all the pods are running and the containers are reday.

A startup probe should be used when the application in our container might take a significant amount of time to reach 
its normal operating state. If we don't add this probe, the liveness probe may fail, causing the pod to enter a restart 
loop.

## Probes Workflow:
- **How Startup, Liveness, and Readiness Probes Work Together:**
![Probes Workflow](https://github.com/balusena/kubernetes-for-devops/blob/main/12-Kubernetes%20Probes/probes_workflow.png)

When a pod is scheduled, Kubernetes initially waits for the initialDelaySeconds specified and then runs the startup probe.
If the startup probe fails, Kubernetes will retry the number of times specified by the failureThreshold. If it still fails
on the last retry, the pod is killed and follows the pod's restart policy. If the startup probe succeeds, both the liveness
and readiness probes are executed after their respective initial delays.

If the liveness probe fails, Kubernetes will retry according to the failureThreshold. If the probe still fails after the
last retry, the pod is restarted. If the probe succeeds, the pod continues running, and the liveness probe is executed 
periodically as defined by the periodSeconds.

If the readiness probe fails, Kubernetes will also retry based on the failureThreshold. If it continues to fail after 
the last retry, the pod is removed from the service's load balancer. If the probe succeeds, the pod resumes receiving 
traffic, and the readiness probe is executed periodically as defined by the periodSeconds.

While probes are very helpful, it is important to be extra cautious when defining them by following best practices.

## Best Practices:
- **1.Ideal Frequency:**
A probe that runs too frequently can waste resources and impact application performance, while a probe that runs too 
infrequently may allow containers to remain in an unhealthy state for too long.

- **2.Light Weight:**
Probes should be as lightweight as possible to ensure checks are executed quickly. Avoid using expensive operations 
within probes, as a probe that does heavy lifting can slow down the containers.

- **3.Correct Restart Policy:**
Probes are influenced by restart policies. Using the "Never" policy will keep the container in a failed state indefinitely.

- **4.Use Only When Needed:**
Simple containers that always terminate on failure don’t need a probe. Use probes only when necessary.

- **5.Keep An Eye On Probes Regularly:**
Whenever new features or fixes are added to the application, they can impact probe performance. Regularly monitor your 
probes and make necessary adjustments.




          
