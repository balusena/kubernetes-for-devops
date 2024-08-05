# Kubernetes ReplicaSets and Deployments

## ReplicaSets:
ReplicaSet object is used to maintain a stable set of replicated pods running within a cluster at any given 
time. Its purpose is to maintain the specified number of pods and prevent the user from loosing the control 
of the application in case of any pod failure or inaccessible. Whenever any pod gets failed replicaset 
immediately launches another pod and replacing it with failed pod.

**Replicaset has following features:**

- A pod template is used to create a new pod whenever an existing pod fails and also replica count is also 
  maintained by defining the desired number of replicas that a controller needs to be running.

- A replica set also ensures that additional pod needs to be created or deleted whenever instance with same
  label is created.

- Replcaset allows to have multiple replicas of pod which means that the traffic is sent to different 
  instances which prevents single instance from being overloaded.

- Replcaset ensures it has multiple replicas of application so that it won't fail because of one pod fails.

![Kubernetes ReplicaSet](https://github.com/balusena/kubernetes-for-devops/blob/main/05-Kubernetes%20ReplicaSets%20and%20Deployments/kubernetes_replicaset.png)

### 1.Creating ReplicaSets with YAML.
```
#1.Clone this repository and move to example folder

git clone https://github.com/balusena/kubernetes-for-devops.git
cd  05-Kubernetes ReplicaSets and Deployments/examples
```
### 2.To get the apiVersion of replicasets in kubernetes.
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep replicaset
replicasets            rs           apps/v1            true         ReplicaSet
```
### 3.Create a replicaset using file nginx-replicaset.yaml
```
ubuntu@balasenapathi:~$ nano nginx-replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```
### 4.To create the replicaset in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-replicaset.yaml
replicaset.apps/nginx-replicaset created
```
### 5.To check whether replicaset is created in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   3         3         3       2m25s

ubuntu@balasenapathi:~$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   3         3         3       2m35s
```
### 6.To get the list of pods running in the kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-h2dqc   1/1     Running   0          4m39s
nginx-replicaset-jsdvj   1/1     Running   0          4m39s
nginx-replicaset-lb77x   1/1     Running   0          4m39s
```
## Self-Healing in Kubernetes

**Note:** Whenever a pod goes down, bringing it back into normal operation is called self-healing.

### 7.To delete a pod from the Kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl delete pod nginx-replicaset-h2dqc
pod "nginx-replicaset-h2dqc" deleted
```
**Note:** We can see that the pod is deleted from the Kubernetes cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-d466v   1/1     Running   0          58s
nginx-replicaset-jsdvj   1/1     Running   0          11m
nginx-replicaset-lb77x   1/1     Running   0          11m
```
**Note:** We can see that the pod is recreated by the ReplicaSet to ensure that the specified number of 
replicas is maintained. This process is called self-healing and is automatically managed by the 
ReplicaSet, which recreates the pod after it has been deleted.

## High Availability in Kubernetes

**Note:** To ensures systems remain operational with minimal downtime, even when some nodes fail.

Adding a Node to the Cluster. First, let's add one more node to the cluster to ensure high availability.

### 8.List of Nodes in the Kubernetes Cluster
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
local-cluster       Ready    control-plane   47h   v1.27.3
local-cluster-m02   Ready    <none>          51m   v1.27.3
```
### 9.Adding a Node to the Cluster
```
ubuntu@balasenapathi:~$ minikube node add --worker -p local-cluster
😄  Adding node m03 to cluster local-cluster
👍  Starting worker node local-cluster-m03 in cluster local-cluster
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
🔎  Verifying Kubernetes components...
🏄  Successfully added m03 to local-cluster!
```
### 10.List of Nodes After Adding the New Node
```
ubuntu@balasenapathi:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE    VERSION
local-cluster       Ready    control-plane   47h    v1.27.3
local-cluster-m02   Ready    <none>          61m    v1.27.3
local-cluster-m03   Ready    <none>          110s   v1.27.3
```
### 11.Deleting a Node and Observing Pod Behavior

**Note:** Before deleting a node, let's see on which nodes the pods are currently running.

List of Pods and Their Nodes
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
nginx-replicaset-d466v   1/1     Running   0          18m   10.244.0.6   local-cluster       <none>           <none>
nginx-replicaset-jsdvj   1/1     Running   0          28m   10.244.1.2   local-cluster-m02   <none>           <none>
nginx-replicaset-lb77x   1/1     Running   0          28m   10.244.1.3   local-cluster-m02   <
```
### 12.Deleting a Node from the Cluster
```
ubuntu@balasenapathi:~$ minikube node delete local-cluster-m02 -p local-cluster
🔥  Deleting node local-cluster-m02 from cluster local-cluster
✋  Stopping node "local-cluster-m02"  ...
🛑  Powering off "local-cluster-m02" via SSH ...
🔥  Deleting "local-cluster-m02" in docker ...
💀  Node local-cluster-m02 was successfully deleted.
```
### 13.List of Pods After Deleting a Node
```
ubuntu@balasenapathi:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
nginx-replicaset-d466v   1/1     Running   0          23m    10.244.0.6   local-cluster       <none>           <none>
nginx-replicaset-szqkn   1/1     Running   0          109s   10.244.2.3   local-cluster-m03   <none>           <none>
nginx-replicaset-v7pb5   1/1     Running   0          109s   10.244.2.2   local-cluster-m03   
```
**Note:** As we can see, the pods are automatically recreated on the available active running node 
"local-cluster-m03" after deleting the node "local-cluster-m02". This demonstrates how Kubernetes 
ensures high availability by redistributing pods to available nodes when one node is removed.

## Deployment
Deployment is used to maintain the versions of your application running in the pod. It helps to efficiently
scale the number of replica pods and enable the rollout and rollback to an earlier deployment version. 
Deployments runs on multiple replicas of the pod and automatically replaces a pod in case of any failure or
unresponsive. In this way Deployments helps to ensure that one or more instances of your application are 
available to serve user requests.

Deployments use a Pod template, which contains a specification for its Pods. The Pod specification 
determines how each Pod should look like: what applications should run inside its containers, which 
volumes the Pods should mount, its labels, and more. When a Deployment's Pod template is changed, new 
Pods are automatically created one at a time.

![Kubernetes Deployment](https://github.com/balusena/kubernetes-for-devops/blob/main/05-Kubernetes%20ReplicaSets%20and%20Deployments/kubernetes_deployment.png)

**Kubernetes ReplicaSet and deployment in a single frame**

![Kubernetes ReplicaSet Deployment](https://github.com/balusena/kubernetes-for-devops/blob/main/05-Kubernetes%20ReplicaSets%20and%20Deployments/kubernetes_replicaset_deployment.png)

The difference between a ReplicaSet and a Deployment is that a ReplicaSet ensures the desired number of 
replicas are always available. In contrast, Deployments offer many more advantages over ReplicaSets, such
as rollouts, rolling updates, and the ability to specify deployment strategies. Additionally, when a 
Deployment is created, a ReplicaSet is automatically created, so there is no need to create ReplicaSets 
manually when using Deployments.

## Rolling Update and Rollback in Deployment 
**1.Rolling Updates:**
Rolling update provides the orderly migration from one version to a newer version. Rolling update is used 
in such situation when a new version of application came and you have to switch to a newer version. In the
rolling update all the running pod will be replaced with the newer version of application by systematically
terminating the older version of application pods.

**2.Rollback:**
Rollback on the other hand is used to roll back to older version of your application. Suppose you have a
new bug in the application that needs to be solved in that case you might need to go to the older version
of the application with rollback you can achieve that. With rollback all the replicas of the pod which are
running on the new version of the application will be rollback again to previous version.

**3.Deployment with Replicaset:**
Deployments are generally used with replicaset as they are used to manage replicsets. With the help of 
deployment You can simply roll back to a previous Deployment revision. When you are managing ReplicaSet 
using Deployment You can also use a Deployment to create a new revision of a ReplicaSet and then migrate 
existing pods from an older revision into the new revision. After that, the Deployment can take care of 
cleaning up old, unused ReplicaSets.

So, Replicaset ensure replicas of pods are available whereas deployment are reponsible for managing 
different versions of the application. Like deployment replicaset cant rollout or rollback to different 
version of application nor maintain any revisions for the same.

**1.Rolling Updates:**

### 1.Deleting All Objects and Services from Kubernetes Cluster
```
ubuntu@balasenapathi:~$ kubectl delete all --all
pod "nginx-replicaset-d466v" deleted
pod "nginx-replicaset-szqkn" deleted
pod "nginx-replicaset-v7pb5" deleted
service "kubernetes" deleted
replicaset.apps "nginx-replicaset" deleted
```
**Note:** All refers to all resource types such as pods, deployments, services, etc. The --all flag deletes
every object instead of specifying each resource name individually.

### 2.Confirming Deletion
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m4s
```

**Note:** The service/kubernetes is a default service created by Kubernetes and cannot be deleted.

### 3.To get the apiVersion of deployments in kubernetes:
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep deployment
deployments         deploy       apps/v1         true         Deployment
```
### 4.Create a deployment using file nginx-deployment.yaml
```
ubuntu@balasenapathi:~$ nano nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```
### 5.To create the deployment in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

### 6.To check whether replicaset is created in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m20s

ubuntu-dsbda@ubuntudsbda-virtual-machine:~$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2m48s
```

### 7.To get the list of all resources in kubernetes cluster:
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-85dd4d599f-5r66l   1/1     Running   0          7m1s
pod/nginx-deployment-85dd4d599f-7w8vm   1/1     Running   0          7m1s
pod/nginx-deployment-85dd4d599f-vr2bb   1/1     Running   0          7m1s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   34m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           7m1s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-85dd4d599f   3         3         3       7m1s
```
**Note:** Whenever a deployment is created, a ReplicaSet is automatically created. This can be confirmed by
the presence of replicaset.apps/nginx-deployment-85dd4d599f.

### 8.Verifying Pod Labels in Kubernetes Cluster
```
ubuntu@balasenapathi:~$ kubectl get pods --show-labels
NAME                                READY   STATUS    RESTARTS   AGE   LABELS
nginx-deployment-85dd4d599f-5r66l   1/1     Running   0          13m   app=nginx,pod-template-hash=85dd4d599f
nginx-deployment-85dd4d599f-7w8vm   1/1     Running   0          13m   app=nginx,pod-template-hash=85dd4d599f
nginx-deployment-85dd4d599f-vr2bb   1/1     Running   0          13m   app=nginx,pod-template-hash=85dd4d599f
```
**Note:** As we can see, the nginx label is added to the pods, as specified in the deployment spec file. 
This label is added by Kubernetes.

### 9.Scaling an Application in Kubernetes 

If we want to scale our application by increasing the number of instances, we can simply adjust the number 
of replicas. For example, setting `replicas: 4` will scale up the application, while setting `replicas: 2`
will scale it down. We can apply these changes in the deployment file.


### 10.Listing All Pods before applying scaling.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85dd4d599f-7w8vm   1/1     Running   0          22m
nginx-deployment-85dd4d599f-vr2bb   1/1     Running   0          22m
```
**Note:** We currently have two pods, as specified in the deployment file. This is automatically managed
by the ReplicaSet created by the deployment.

### 10.Scaling an Application Using `kubectl`
You can scale the application without modifying the deployment file by using the `kubectl scale` command. 
For example, to scale the application to 4 replicas:
```
ubuntu@balasenapathi:~$ kubectl scale --replicas=4 deployment/nginx-deployment
deployment.apps/nginx-deployment scaled
```
**Note:** The output confirms that the deployment has been scaled.

### 11.Listing All Pods after applying scaling using kubectl.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85dd4d599f-7qkg7   1/1     Running   0          5m36s
nginx-deployment-85dd4d599f-7w8vm   1/1     Running   0          34m
nginx-deployment-85dd4d599f-n46kj   1/1     Running   0          5m36s
nginx-deployment-85dd4d599f-vr2bb   1/1     Running   0          34m
```
**Note:** Be cautious when making changes directly with kubectl scale, as it can lead to discrepancies 
between the deployment file and the cluster state. In the example above, we have 4 replicas running in 
the cluster, but the deployment file specifies only 2 replicas. This can cause confusion. It is always 
recommended to make changes in the deployment file and apply them using the kubectl apply command to 
maintain consistency.

### 12.Changing the version from 1.21.3 to 1.21 using kubectl command without using deployment file:
```
ubuntu@balasenapathi:~$ kubectl set image deployment/nginx-deployment nginx-container=nginx:1.21
deployment.apps/nginx-deployment image updated
```
**Note:** "nginx-container" this is conatiner name a pod can have multiple conatiners with multiple images
so we should specify in which container  we want to apply this image.

**Note:** The output confirms that the image is updated.

### 13.Now try to list down the resources:
```
ubuntu@balasenapathi:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-865f6b58b8-42w86   1/1     Running   0          2m10s
pod/nginx-deployment-865f6b58b8-dtfbz   1/1     Running   0          79s
pod/nginx-deployment-865f6b58b8-tjdnb   1/1     Running   0          77s
pod/nginx-deployment-865f6b58b8-zw7v6   1/1     Running   0          2m10s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           16h

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c55946d5c   0         0         0       23m
replicaset.apps/nginx-deployment-85dd4d599f   0         0         0       16h
replicaset.apps/nginx-deployment-865f6b58b8   4         4         4       2m10s
```
**Note:** As we can see 4 replicas are running and we can check if the image is updated or not by using
the below command.

### 14.Inspecting Pod Details
```
ubuntu@balasenapathi:~$ kubectl describe pod/nginx-deployment-865f6b58b8-42w86
Name:             nginx-deployment-865f6b58b8-42w86
Namespace:        default
Priority:         0
Service Account:  default
Node:             local-cluster-m03/192.168.58.4
Start Time:       Tue, 05 Sep 2023 16:53:11 +0530
Labels:           app=nginx
                  pod-template-hash=865f6b58b8
Annotations:      <none>
Status:           Running
IP:               10.244.1.4
IPs:
  IP:           10.244.1.4
Controlled By:  ReplicaSet/nginx-deployment-865f6b58b8
Containers:
  nginx-container:
    Container ID:   docker://394b122b91e237cae7944a7d9db1ed3b773ad19ad98b4847ac908c5e735cc05f
    Image:          nginx:1.21
    Image ID:       docker-pullable://nginx@sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb884165e514
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 05 Sep 2023 16:54:02 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2fglj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-2fglj:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m52s  default-scheduler  Successfully assigned default/nginx-deployment-865f6b58b8-42w86 to local-cluster-m03
  Normal  Pulling    5m50s  kubelet            Pulling image "nginx:1.21"
  Normal  Pulled     5m11s  kubelet            Successfully pulled image "nginx:1.21" in 38.984340097s (38.984350101s including waiting)
  Normal  Created    5m2s   kubelet            Created container nginx-container
  Normal  Started    5m1s   kubelet            Started container nginx-container
```
**Note:** The image is updated to version 1.21. This demonstrates that while you can update the application
either by directly using kubectl commands or by updating the deployment file, it is always safer and better
practice to use the deployment file. This ensures that changes are managed consistently and in a controlled
manner.

### 15.Viewing Rollout History of a Deployment
```
ubuntu@balasenapathi:~$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>  --------> The initial deployment
2         <none>  --------> Updated latest version to version 1.21.3 using deployment file with kubectl apply
3         <none>  --------> Updated version 1.21.3 to version 1.21 using the kubectl set image command
```
**Note:** The output shows three revisions for this deployment. It is best practice to provide a 
"CHANGE-CAUSE" for each revision. This helps in maintaining a clear record of what has changed. 
You can specify a change cause using the --record flag when applying changes with kubectl. This way, 
each revision will have a meaningful description of the changes made.

### 16.Now change the image from version 1.21 to version 1.20 using --record:
```
ubuntu@balasenapathi:~$ kubectl set image deployment/nginx-deployment nginx-container=nginx:1.20 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deployment image updated
```
**Note:** Now the CHANGE-CAUSE will be recorded because of the above command

### 17.To get the history of revisions done by rollouts:
```
ubuntu@balasenapathi:~$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
4         kubectl set image deployment/nginx-deployment nginx-container=nginx:1.20 --record=true
```
**Note:** The CHANGE-CAUSE is recorded as "kubectl set image deployment/nginx-deployment 
nginx-container=nginx:1.20 --record=true". For best practices, it is recommended to specify 
a meaningful CHANGE-CAUSE using annotations. Annotations can be added via kubectl commands 
or directly in the deployment specification file.

- Here we are giving annotations in the deploymenyt specification file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
  annotations:
    kubernetes.io/change-cause: "Updating to alpine version"
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:alpine
          ports:
            - containerPort: 80
```
**The changes made include:**
- **Annotations**: Added `kubernetes.io/change-cause: "Updating to alpine version"`
- **Spec**: Updated the `image` to `nginx:alpine`

### 18.Attempting to Create the Deployment
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-deployment.yaml
Error from server (NotFound): error when creating "nginx-deployment.yaml": namespaces "nginx" not found
```
### 19.To check the available namespaces:
```
ubuntu@balasenapathi:~$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   2d19h
kube-node-lease        Active   2d19h
kube-public            Active   2d19h
kube-system            Active   2d19h
kubernetes-dashboard   Active   2d16h
```
### 20.Creating the Namespace
Create the "nginx" namespace if it does not exist:
```
ubuntu@balasenapathi:~$ kubectl create namespace nginx
namespace/nginx created
```
### 21.Reapply nginx deployment configuration, after creating the namespace.
```
ubuntu@balasenapathi:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
**Note:** In my case, I had not created the namespace before, which resulted in an error.To resolve this
issue, I removed the namespace: nginx line from the deployment file and successfully applied the changes. 
The removal of the namespace resolved the error, and the CHANGE-CAUSE is now reflected in the rollout 
history.

- Here iam removing "namespace: nginx" in the deployment specification file.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    kubernetes.io/change-cause: "Updating to alpine version"
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:alpine
          ports:
            - containerPort: 80
```
### 22.Viewing Rollout History of Revisions
```
ubuntu@balasenapathi:~$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
4         kubectl set image deployment/nginx-deployment nginx-container=nginx:1.20 --record=true
5         Updating to alpine version
```
**Note:** The CHANGE-CAUSE is updated to "Updating to alpine version" as specified in the deployment file.
This is how rollouts manage updates, allowing us to upgrade our application at any point in time. By 
keeping track of changes through CHANGE-CAUSE, we can better understand and manage the evolution of our 
deployment configurations.

**2.Rollback:**

Let's say after this rollout we find some issues and want to roll back to the previous version. Instead of
using `history`, we should use `undo`. This will roll back to the previous version of the deployment, such
as `nginx:1.20`. To do this, use:
```
ubuntu@balasenapathi:~$ kubectl rollout undo deployment/nginx-deployment
```
Alternatively, you can specify a revision number to roll back to a specific version. For example, you can 
roll back to revision 1, which might correspond to the nginx:latest version, using:
```
ubuntu@balasenapathi:~$ kubectl rollout undo deployment/nginx-deployment --to-revision=1
```
### 1.To rollback the deployment to previous versions:
```
ubuntu@balasenapathi:~$ kubectl rollout undo deployment/nginx-deployment --to-revision=1
deployment.apps/nginx-deployment rolled back
```
### 2.To see the rollout deployment status using kubectl rollout status:
```
ubuntu@balasenapathi:~$ kubectl rollout status deployment/nginx-deployment
deployment "nginx-deployment" successfully rolled out
```
### 3.To verify it was rolled back to the nginx:latest version, list all the pods and check the pod image:
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85dd4d599f-62sx9   1/1     Running   0          16m
nginx-deployment-85dd4d599f-jcqr2   1/1     Running   0          16m
nginx-deployment-85dd4d599f-vq66t   1/1     Running   0          16m
nginx-deployment-85dd4d599f-zx4jm   1/1     Running   0          16m

ubuntu@balasenapathi:~$ kubectl describe pod nginx-deployment-85dd4d599f-62sx9 | grep image
Normal  Pulling    19m   kubelet            Pulling image "nginx:latest"
Normal  Pulled     19m   kubelet            Successfully pulled image "nginx:latest" in 1.57
```
**Note:** From the above output, we can see that the image version has been successfully rolled back to 
the newest version, i.e., nginx:latest.

### 4.To get the complete information about a pod:
```
ubuntu@balasenapathi:~$ kubectl describe pod/nginx-deployment-85dd4d599f-62sx9
Name:             nginx-deployment-85dd4d599f-62sx9
Namespace:        default
Priority:         0
Service Account:  default
Node:             local-cluster-m03/192.168.58.4
Start Time:       Tue, 05 Sep 2023 18:56:57 +0530
Labels:           app=nginx
pod-template-hash=85dd4d599f
Annotations:      <none>
Status:           Running
IP:               10.244.1.13
IPs:
IP:           10.244.1.13
Controlled By:  ReplicaSet/nginx-deployment-85dd4d599f
Containers:
nginx-container:
Container ID:   docker://a3da30068eda16b8cbb0629c8521240209ef061ae04144115cea09328682e851
Image:          nginx:latest
Image ID:       docker-pullable://nginx@sha256:104c7c5c54f2685f0f46f3be607ce60da7085da3eaa5ad22d3d9f01594295e9c
Port:           80/TCP
Host Port:      0/TCP
State:          Running
Started:      Tue, 05 Sep 2023 18:57:02 +0530
Ready:          True
Restart Count:  0
Environment:    <none>
Mounts:
/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lgjk8 (ro)
Conditions:
Type              Status
Initialized       True
Ready             True
ContainersReady   True
PodScheduled      True
Volumes:
kube-api-access-lgjk8:
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
Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
Normal  Scheduled  24m   default-scheduler  Successfully assigned default/nginx-deployment-85dd4d599f-62sx9 to local-cluster-m03
Normal  Pulling    24m   kubelet            Pulling image "nginx:latest"
Normal  Pulled     24m   kubelet            Successfully pulled image "nginx:latest" in 1.575317185s (1.575328811s including waiting)
Normal  Created    24m   kubelet            Created container nginx-container
Normal  Started    24m   kubelet            Started container nginx-container
```
**Note:** We can confirm that the image has been rolled back to the previous deployment REVISION:1, i.e., 
Image: nginx:latest.












