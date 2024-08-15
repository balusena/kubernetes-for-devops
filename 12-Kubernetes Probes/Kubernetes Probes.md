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

### 2.For our mongo deployment lets make a simple change in statefulset.yaml file and check if our conatiner is healthy or not.
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
**Note:** We are intentionally using - "db1.adminCommand('ping')" in the exec section to test the container.

### 3.Now apply the changes in the local-cluster:
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

### 6.Correct the db1 to db in statefulset.yaml and apply the changes in local-cluster:
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


