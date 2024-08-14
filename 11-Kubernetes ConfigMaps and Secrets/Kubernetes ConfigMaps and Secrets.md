# kubernetes ConfigMaps and Secrets.
ConfigMaps and Secrets in Kubernetes are resources for managing configuration and sensitive data,respectively. 

**ConfigMaps** is used to store non-sensitive data like environment variables, command-line arguments, and 
configuration files, separating configuration from application code.

**Secrets** are designed for securely storing confidential information like passwords, API tokens, and certificates.
Secrets provide an extra layer of security compared to plain text, making them suitable for sensitive data. 

## 1.ConfigMaps
- ConfigMaps in Kubernetes is an API object that allows you to store and manage configuration data separately 
  from your application code.
- They provide a way to supply configuration information to containers, pods, or even entire deployments, 
  without requiring changes to the application itself.
- ConfigMaps play a crucial role in maintaining flexibility, portability, and separation of concerns within a
  Kubernetes environment.

**The primary purpose of ConfigMaps is to decouple configuration settings from application logic. This 
separation offers several benefits:**

- **1.Flexibility:** ConfigMaps enable easy modification of configuration parameters without the need to modify and redeploy your application. This is particularly useful when transitioning between different environments (e.g., development, testing, production) or when tuning application behavior.

- **2.Collaboration:** Teams can collaborate effectively by modifying ConfigMaps without interfering with application code, promoting smoother DevOps practices.

- **3.Kubernetes-native:** ConfigMaps integrates seamlessly with Kubernetes, making them a natural fit for configuration management within the container orchestration ecosystem.

ConfigMaps can contain simple key-value pairs or even entire configuration files, which are mounted as 
volumes or accessed as environment variables by pods. They can be created declaratively using YAML manifests
or through imperative commands.

In summary, ConfigMaps empowers Kubernetes users to manage configuration data in a flexible, modular, and 
scalable manner, ensuring that applications are easily adaptable to varying environments and operational 
requirements.

![Kubernetes ConfigMap](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/configmap.png)

## 2.Creating and Managing ConfigMaps.
**There are two ways of creating a config map:**

- 1.The imperative way – without using a ConfigMap definition file.
- 2.Declarative way by using a Configmap definition file.

**1.Imperative Way:Creating a ConfigMap Without Using a Definition File.**
The imperative way involves using kubectl commands to create a ConfigMap directly from the command line 
without defining a ConfigMap YAML file.

### 1.Creating a ConfigMap from Literal Values.
You can create a ConfigMap from literal values using the kubectl create configmap command. 

- This example creates a ConfigMap named my-config with two key-value pairs.
```
ubuntu@balasenapathi:~$ kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
configmap/my-config created
```
### 2.Creating a ConfigMap from a File.
You can create a ConfigMap from a file using the --from-file option. 

- This example creates a ConfigMap named my-config from a file called config.txt.
```
ubuntu@balasenapathi:~$ kubectl create configmap my-config --from-file=config.txt
configmap/my-config created
```
### 3.Creating a ConfigMap from a Directory.
If you have multiple files in a directory and want to include all of them in a ConfigMap,you can use the --from-file option with a directory. 

- This example creates a ConfigMap named my-config from all files in the ./config-dir directory.
```
ubuntu@balasenapathi:~$ kubectl create configmap my-config --from-file=config-dir/
configmap/my-config created
```
**2.Declarative Way:Creating a ConfigMap Using a Definition File.**
The declarative way involves defining a ConfigMap in a YAML file and then applying it using kubectl. This
method is more suited for managing configurations as code and versioning.

### 1.creating a ConfigMap named app-config with key-value pairs using YAML.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "mongodb://mongo.example.com:27017"
  API_KEY: "your-api-key"
```
```
ubuntu@balasenapathi:~$ kubectl apply -f configmap.yaml
configmap/app-config configured
```
**1.Using ConfigMaps in Pods:**

ConfigMaps can be consumed in Kubernetes Pods as environment variables or mounted volumes.

### 2.Using ConfigMaps as Environment Variables.
Here's how you can use the app-config ConfigMap as environment variables in a Pod:
```
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app-container
      image: your-app-image
      env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_URL
        - name: API_KEY
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: API_KEY
```
```
ubuntu@balasenapathi:~$ kubectl apply -f pod.yaml
pod/app-pod created
```
**2.Using ConfigMaps as Mounted Volumes:**

You can also mount a ConfigMap as a volume in a Pod to access its data as files.
```
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app-container
      image: your-app-image
      volumeMounts:
        - name: config-volume
          mountPath: /app/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```
```
ubuntu@balasenapathi:~$ kubectl apply -f pod.yaml
pod/app-pod created
```
**Note:** In this example, the app-config ConfigMap will be available in the /app/config directory within the container.

These approaches demonstrate how ConfigMaps can be consumed in Kubernetes Pods, providing configuration 
data to your applications either as environment variables or as mounted files. This separation of 
configuration from application code enhances flexibility, manageability, and reusability in a Kubernetes 
environment.

## 2.Secrets
Secrets in Kubernetes is an object that is designed for securely managing sensitive information, such as 
passwords, tokens, and certificates. With the help of secrets you don't need to put the confidential data
in the application code. It also helps to ensure that sensitive information remains secure within a 
Kubernetes cluster, following best practices for data handling and access control.

Secrets provides an abstraction layer that allows you to separate sensitive data from your application 
code, similar to ConfigMaps. However, Secrets are specifically designed to handle confidential information
more securely.

![Kubernetes Secrets](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/secrets.png)

### 1.Types of Secrets
Kubernetes supports several types of Secrets to accommodate different use cases. Each type of Secret has a
specific purpose and is used in different scenarios. Let’s go through some of these types.

- **1.Opaque:**
The Opaque is the default Secret type. You can use it to store arbitrary Secret data. These Secrets don’t 
have any specific constraints imposed by Kubernetes.
```
---
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded 'admin'
  password: MWYyZDFlMmU2N2Rm  # base64 encoded '1f2d1e2e67df'
```

- **2.kubernetes.io/service-account-token:**
These are automatically created by Kubernetes and attached to service accounts. They’re used to authenticate
applications that want to communicate with the Kubernetes API server. The data field for these Secrets 
includes a token and a Kubernetes API server root certificate.

- **3.kubernetes.io/dockercfg and kubernetes.io/dockerconfigjson:**
These types of Secrets are used to hold Docker configuration information. The dockercfg Secret type is used
for Docker’s legacy configfile format, and dockerconfigjson is used for Docker’s newer config.json format.
They both contain the necessary credentials for pulling images from private Docker repositories.

- **4.kubernetes.io/basic-auth:**
This type of Secret can store credentials for basic authentication. It contains two fields: username and 
password. These are often used when interacting with REST APIs that require basic authentication.
```
---
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: kubernetes.io/basic-auth
stringData:
  username: admin
  password: secret
```

- **5.kubernetes.io/ssh-auth:**
These Secrets are used to hold SSH authentication credentials. They contain a single ssh-privatekey field
for the private SSH key. This can be useful when you need to interact with a service that requires SSH 
authentication.

- **6.kubernetes.io/tls:**
These are used to store a certificate and its corresponding private key. These are typically used for 
TLS-enabled services. This type of Secret expects two fields: tls.crt and tls.key.
```
---
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: kubernetes.io/tls
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
```
**Note:** Remember that all secret data should be base64 encoded before being included in the Secret 
definition. Kubernetes will decode this data when it is used.

![Kubernetes Secrets Types](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/secret_types.png)

### 2.Secret vs ConfigMap
Although Secrets and ConfigMaps serve different purposes in Kubernetes, they share several similarities. 
Both are stored in etcd and have a size limitation of 1MB. They have a similar lifecycle, from creation 
to deletion, and they both use the data field to store information.

However, the differences between Secrets and ConfigMaps are vital: 
- Secrets are meant to store sensitive data and are encrypted in transit and at rest.
- ConfigMaps are used for non-sensitive configuration data and are not encrypted.

![Kubernetes ConfigMaps Secrets](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/configmaps_secrets.png)

### 3.Security and Best Practices for ConfigMaps and Secrets in Kubernetes.

#### 1.ConfigMaps.

- **1.Sensitive Data**: Avoid storing sensitive information like passwords, API keys, or tokens in ConfigMaps. Use Secrets for confidential data.

- **2.Access Control**: Apply Role-Based Access Control (RBAC) to restrict who can create, update, or access ConfigMaps.

- **3.Encryption**: Encrypt sensitive data before storing it in ConfigMaps, especially when dealing with temporary configurations.

- **4.Limited Use**: Use ConfigMaps for non-sensitive configuration settings, environment variables, and other data that's safe to expose.

- **5.Immutability**: Treat ConfigMaps as immutable; create new ConfigMaps for updates instead of modifying existing ones.

#### 2.Secrets.

- **1.Data Sensitivity**: Use Secrets specifically for sensitive data like passwords, API keys, certificates, and tokens.

- **2.Encryption at Rest**: Enable encryption at rest for Secrets to protect data stored in etcd, Kubernetes' data store.

- **3.API Access Control**: Limit access to Secrets using RBAC, ensuring only authorized users or applications can retrieve or modify them.

- **4.Avoid Direct Access**: Avoid directly accessing Secrets from applications or containers; instead, use them as environment variables or mounted volumes.

- **5.Secret Rotation**: Regularly rotate Secrets, such as changing passwords or renewing certificates, and ensure proper disposal of old Secrets.

### 4.General Practices.

- **1.Secure Delivery**: When creating ConfigMaps or Secrets from files, use secure delivery mechanisms to prevent exposing data during transmission.

- **2.Minimize Exposure**: Use Kubernetes pod security policies to restrict pod capabilities and limit potential attack vectors.

- **3.Namespace Isolation**: Use separate namespaces for different applications, teams, or environments to enforce isolation and access control.

- **4.Audit and Monitoring**: Implement auditing and monitoring of ConfigMap and Secret access to detect unauthorized activities.

- **5.Least Privilege**: Apply the principle of least privilege by granting only the necessary permissions to users and applications.

- **6.Automation**: Utilize Infrastructure as Code (IaC) tools to create and manage ConfigMaps and Secrets consistently.

- **7.Regular Review**: Periodically review and assess ConfigMaps and Secrets for compliance with security policies and best practices.

- **8.Educate and Train**: Educate your team about secure handling and management of ConfigMaps and Secrets to maintain consistent practices.

By adhering to these security best practices, you can ensure that your ConfigMaps and Secrets are properly
managed, and protected, and contribute to a more secure Kubernetes environment.

### Do Not Hradcode
If we look at the MongoDB deployment or StatefulSet we've created, we configured properties like "username"
and "password." This approach allows us to easily change these configurations without rebuilding the actual
image. Now, we will learn how to pass the same configuration data to containers using ConfigMaps and Secrets.

Whenever we deploy any application, we shouldn't hardcode properties that change for each environment. 
Instead, we should configure those properties so that we don't need to rebuild the image whenever these 
values change. All we need to do is pass those values from outside. This way, we can reuse the same image
for different environments like DEV, QA, UAT, PRE-PROD, and PROD.

![Do Not Hradcode](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/dont_hardcode.png)

### Different Ways to Configure Data in YAML

In Kubernetes deployments, configuration data can be provided in several ways using YAML files:

1. **Passing Arguments**: Configuration data can be specified as command-line arguments within the YAML file for a container. This method involves adding `args` under the container spec, allowing dynamic values to be passed to the application at runtime.

2. **Environment Variables**: Configuration data can be set as environment variables in the YAML file. By defining `env` under the container spec, you can pass environment-specific settings to the application, making it easy to adapt the configuration without modifying the application code or image.

3. **Configuration Files**: Data can be provided through configuration files mounted as volumes in the YAML file. By defining `volumes` and `volumeMounts`, you can inject configuration files into the container, which the application can then read at startup.

![Passing Arguments and Environment Variables](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/passing_arguments_env_variables.png)

We have already explored a couple of methods (Passing Arguments and Environment Variables) for configuring
data. However, hardcoding this data directly into the pod definition is not ideal, as it requires creating
different pod definitions for each environment.To reuse the same pod definition across multiple environments,
it is better to define configuration data in a common place and reference that data from the pod description.
Kubernetes provides two special types of volumes for this purpose:

- **ConfigMap**
- **Secrets**

![Configuration Files](https://github.com/balusena/kubernetes-for-devops/blob/main/11-Kubernetes%20ConfigMaps%20and%20Secrets/configuration_files.png)

### 1.To get the api-resources of ConfigMap.
```
ubuntu@balasenapathi:~$ kubectl api-resources | grep configmap
configmaps                cm           v1            true         ConfigMap
```
### 2.Creating a ConfigMap in local-cluster.
```
ubuntu@balasenapathi:~$ nano mongo-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: mongodb-config
immutable: false
data:
  username: admin
  password: password
  mongodb.conf: |
    storage:
      dbPath: /data/configdb
    replication:
        replSetName: "rs0" 
```
Unlike most configuration files that have a spec: section, a ConfigMap in Kubernetes has a data: section. 
Instead of spec, we use the data field in a ConfigMap.

The data: field accepts key-value pairs. For example, to configure MongoDB with a username of "admin" and 
a password of "password", we would use the following key-value pairs:
```
data:
  username: "admin"
  password: "password"
```
In addition to string values, ConfigMaps can also define configuration files. Here, we'll create a simple
ConfigMap that specifies the dbpath and the replica set name for MongoDB. Since this is a multiline string
and we want to preserve the new lines, we use the pipe symbol | to denote a literal block, like this:
```
mongodb.conf: |
    storage:
      dbPath: /data/configdb
    replication:
        replSetName: "rs0" 
```
Important Notes:

**1.Size Limitation:** ConfigMaps are not designed to hold large chunks of data. The data stored in a 
ConfigMap cannot exceed 1MB. If you need to store settings larger than this limit, consider mounting a 
volume or using a separate database or file service.

**2.Immutable ConfigMaps:** For security reasons, if you don't want anyone to update a ConfigMap, you can
use the immutable: true attribute. When set to true, the ConfigMap cannot be updated; it can only be 
deleted or recreated. The default value is false, which means the ConfigMap can be edited.

### 3.Applying the ConfigMap in local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f mongo-configmap.yaml
configmap/mongodb-config created
```
### 4.Verify whether the ConfigMap is created in local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      7h20m
mongodb-config     3      2m3s

ubuntu@balasenapathi:~$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      7h20m
mongodb-config     3      112s
```
Note: cm is the short form of ConfigMap,as we can see we have ConfigMap with name "mongodb-config" with 3 keys,

Now that we have a ConfigMap, let's see how to use it in our StatefulSet. We can use this config data as 
environment variables. Previously, we hardcoded the "username" and "password". Instead of hardcoding, let's
retrieve the data from the ConfigMap we created. Instead of using value, we should use valueFrom and
specify configMapKeyRef. The key we are referencing from the ConfigMap is "username", and the ConfigMap 
name is "mongodb-config". This is the ConfigMap we created, and the key refers to the "username: admin" 
we defined in the ConfigMap file. We'll do the same for the password.

Now, this container will use the configuration data from the ConfigMap. If we look at the ConfigMap, we 
also defined a custom MongoDB configuration file. How can we use this configuration file in the StatefulSet?
To use a custom configuration file, we should pass that file's path when starting MongoDB. Let's try to
provide the configuration file as an argument in the command section as --config=/etc/mongo/mongodb.conf.

But where is this file located, and how does the StatefulSet pull this file from the ConfigMap? For MongoDB
to load this file, it must exist in the container. As a first step, we should create this file in the 
container using volumes. Note that ConfigMaps and Secrets are special types of volumes. Let's mount the 
special type of ConfigMap volume onto the container. First, we should declare the volumes. This volume 
should be at the container level, specified in the volumes: section. Let's give the volume a name, such as
name: mongodb-config, and the type of volume is configMap:. The name of the ConfigMap is name: mongodb-config.

Since we have three keys in the ConfigMap file, we should specify which key should be used as a file. We 
need mongodb.conf, and we can specify that with items:, which is an array. (Please note that if you don't
declare the items array, all the keys from the ConfigMap will be used as files—in our case, username, 
password, and mongodb.conf.) The key is key: mongodb.conf from the ConfigMap, and this is the name 
path: mongodb.conf with which the file is created.

Now that we have defined the volume, we should mount it into the container. Go to the volumeMounts: section
and mount this with the name name: mongodb-config, and the mountPath is mountPath: /etc/mongo. Since we are
already specifying the replica set name rs0 in the configuration file, we can remove this from the 
StatefulSet command: section. Additionally, we can remove the dbPath from the args: section, i.e., 
args: ["--dbpath", "/data/db"].

### 5.create the statefulset.yaml
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
                configMapKeyRef:
                  name: mongodb-config
                  key: password
          command:
            - mongod
            - "--bind_ip_all"
            - "--replSet"
            - "--config=/etc/mongo/mongodb.conf"
          volumeMounts:
            - name: mongo-volume
              mountPath: /data/configdb
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
### 6.Now apply the statefulset.yaml changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo configured
```
### 7.Now lets try to list the pods that are running in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -w
NAME      READY   STATUS              RESTARTS   AGE
mongo-0   1/1     Running             0          6s
mongo-1   0/1     ContainerCreating   0          1s
mongo-1   1/1     Running             0          2s
mongo-2   0/1     Pending             0          0s
mongo-2   0/1     Pending             0          0s
mongo-2   0/1     ContainerCreating   0          0s
mongo-2   1/1     Running             0          2s

ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          3m31s
mongo-1   1/1     Running   0          3m26s
mongo-2   1/1     Running   0          3m24s
```
**Note:** The configmap and the pod that uses this configmap must be in the same namespace.

#Now let us get into the pod and see if the environmental variables from our configmap and also the configuration from the configmap are
mounted correctly or not.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/#
```
### 8.Now list down the environmental variables.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# env
HOSTNAME=mongo-0
MONGO_VERSION=4.0.8
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
TERM=xterm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
MONGO_PACKAGE=mongodb-org
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.
7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
MONGO_REPO=repo.mongodb.org
MONGO_INITDB_ROOT_PASSWORD=password
JSYAML_VERSION=3.13.0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
GPG_KEYS=9DA31620334BD75D9DCB49F368818C72E52529D4
PWD=/
SHLVL=1
HOME=/root
MONGO_MAJOR=4.0
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
MONGO_INITDB_ROOT_USERNAME=admin
GOSU_VERSION=1.11
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
_=/usr/bin/env
```
**Note:**
We can see the username and password enviromental variables "MONGO_INITDB_ROOT_USERNAME=admin","MONGO_INITDB_ROOT_PASSWORD=password"

### 9.Now lets see the configuation file is mounted or not
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# cat /etc/mongo/mongodb.conf
storage:
dbPath: /data/configdb
replication:
replSetName: "rs0"
```
**Note:** As we can see, this is the configuration file that we provided in the ConfigMap and mounted into
the container. This way, we can use the ConfigMap data in a pod. Please note that a ConfigMap doesn't 
differentiate between single-line property values and multi-line file-like values. What matters is how 
pods and other objects consume those values. We consume these values as environment variables, e.g.,
```
data:
  username: "admin"
  password: "password"
```
- And we consume these values as a file.
```
yaml
Copy code
storage:
  dbPath: /data/db
replication:
  replSetName: "rs0"
```
In Kubernetes, ConfigMaps are used to manage configuration data that can be consumed by pods. ConfigMaps 
can store data in two formats: as individual key-value pairs (single-line property values) or as file-like
data (multi-line values).

Now that we have decoupled the configuration from the pod manifest, this ConfigMap can be used by any 
number of pods.If you want to change the data,all you need to do is update the ConfigMap,and it will be 
reflected in all the pods that use this ConfigMap.

### 10.Now lets update the configmap and see by Creating a ConfigMap in local-cluster.
```
ubuntu@balasenapathi:~$ nano mongo-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: mongodb-config
immutable: false
data:
  username: admin1
  password: password
  mongodb.conf: |
    storage:
      dbPath: /data/db
    replication:
        replSetName: "rs0" 
```
### 11.Now apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f mongo-configmap.yaml
configmap/mongodb-config configured
```
### 12.Now get into the pod and see if the values are updated automatically.

Let's list the file contents of mongodb.cong.
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# cat /etc/mongo/mongodb.conf
storage:
  dbPath: /data/db
replication:
    replSetName: "rs0"
```
**Note:** They have updated, we can see "dbPath: /data/db" from "dbPath: /data/configdb"

### 13.Now lets check the environmental variables :
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# cat /etc/mongo/mongodb.conf
storage:
dbPath: /data/db
replication:
replSetName: "rs0"
root@mongo-0:/# env
HOSTNAME=mongo-0
MONGO_VERSION=4.0.8
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
TERM=xterm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
MONGO_PACKAGE=mongodb-org
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.
7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
MONGO_REPO=repo.mongodb.org
MONGO_INITDB_ROOT_PASSWORD=password
JSYAML_VERSION=3.13.0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
GPG_KEYS=9DA31620334BD75D9DCB49F368818C72E52529D4
PWD=/
SHLVL=1
HOME=/root
MONGO_MAJOR=4.0
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
MONGO_INITDB_ROOT_USERNAME=admin
GOSU_VERSION=1.11
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
_=/usr/bin/env
```
**Note:** The username has not been updated from admin to admin1; it is still MONGO_INITDB_ROOT_USERNAME=admin.

This is because ConfigMaps consumed as volumes are updated automatically, but ConfigMaps consumed as 
environment variables are not updated automatically and require a pod restart.

### 14.Now lets restart a pod and see whether the environmental variables are updated after pod restart or not:

- Now delete the mongo-0 pod.
```
ubuntu@balasenapathi:~$ kubectl delete pod mongo-0
pod "mongo-0" deleted
```
- The pods gets automatically created bcz we are using statefulset.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          25s
mongo-1   1/1     Running   0          70m
mongo-2   1/1     Running   0          70m
```
### 15.Now list into the same mongo-0 pod and list down the environmental variables:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# env
HOSTNAME=mongo-0
MONGO_VERSION=4.0.8
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
TERM=xterm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
MONGO_PACKAGE=mongodb-org
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.
7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
MONGO_REPO=repo.mongodb.org
MONGO_INITDB_ROOT_PASSWORD=password
JSYAML_VERSION=3.13.0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
GPG_KEYS=9DA31620334BD75D9DCB49F368818C72E52529D4
PWD=/
SHLVL=1
HOME=/root
MONGO_MAJOR=4.0
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
MONGO_INITDB_ROOT_USERNAME=admin1
GOSU_VERSION=1.11
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
_=/usr/bin/env
```
**Note:** Its updated and the username has changed from admin to admin1 "MONGO_INITDB_ROOT_USERNAME=admin1" after mongo-0 pod restarted.

### 16.To see if the data is stored in the new path i.e /data/db in the container:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bash
root@mongo-0:/# ls /data/db
WiredTiger                            collection-15-8387574925976539953.wt  diagnostic.data                  
index-23-8387574925976539953.wt
WiredTiger.lock                       collection-17-8387574925976539953.wt  index-1-8387574925976539953.wt   
index-3-8387574925976539953.wt
WiredTiger.turtle                     collection-19-8387574925976539953.wt  index-10-8387574925976539953.wt  
index-5-8387574925976539953.wt
WiredTiger.wt                         collection-2-8387574925976539953.wt   index-12-8387574925976539953.wt  
index-7-8387574925976539953.wt
WiredTigerLAS.wt                      collection-22-8387574925976539953.wt  index-14-8387574925976539953.wt  journal
_mdb_catalog.wt                       collection-4-8387574925976539953.wt   index-16-8387574925976539953.wt  mongod.lock
collection-0-8387574925976539953.wt   collection-6-8387574925976539953.wt   index-18-8387574925976539953.wt  sizeStorer.wt
collection-11-8387574925976539953.wt  collection-8-8387574925976539953.wt   index-20-8387574925976539953.wt  storage.bson
collection-13-8387574925976539953.wt  collection-9-8387574925976539953.wt   index-21-8387574925976539953.wt
```
**Note:** We can see that the data is loaded in the newly updated location i.e, "/data/db"

As we know, the password is confidential data, and there might be other sensitive information like API 
keys and certificates that we don’t want everyone to see. To configure such confidential data, Kubernetes 
provides another type of volume called Secrets. ConfigMaps and Secrets are almost the same in terms of how
we create and use them; the key difference is that Secrets are secured, and the data in Secrets can be 
encrypted in the etcd database.

### 17.Creating a secret file in local-cluster.
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
**Note:** Please note that the data in the secret must be given as base64 encoded data.

### 19.To encode a text from a command line
```
ubuntu@balasenapathi:~$ echo -n password | base64
cGFzc3dvcmQ=
```
**Note:** we can give any number of keys we have given in configmap,kubernetes provides different types of secrets.

You can refer this here ===> https://kubernetes.io/docs/concepts/configuration/secret/

---> For arbitary user defined data we should use Opaque secret type
---> For storing basic authentication credentials we should use  kubernetes.io/basic-auth

### 20.To refer to this secret data from the pod we need to do some changes in statefulset.
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
**Note:** In the env: section, make the following changes:

- Change valueFrom: to valueFrom: secretKeyRef:
- Change the key: from "password" to "password"
- Change the name: from "mongodb-config" to "mongodb-secret"

Now, this password will be read from the Secret.

### 21.Now lets delete the password from the mongo-congigmap.yaml
```
ubuntu@balasenapathi:~$ nano mongo-configgmap.yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: mongodb-config
immutable: false
data:
  username: admin1
  mongodb.conf: |
    storage:
      dbPath: /data/db
    replication:
        replSetName: "rs0"
```
### 22.Now apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f mongo-configmap.yaml
configmap/mongodb-config configured
```
```
ubuntu@balasenapathi:~$ kubectl apply -f mongo-secret.yaml
secret/mongodb-secret created
```

### 23.Now delete the statefulset.yaml and reapply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl delete -f statefulset.yaml
statefulset.apps "mongo" deleted
```
```
ubuntu@balasenapathi:~$ kubectl apply -f statefulset.yaml
statefulset.apps/mongo created
```
### 24.Now get inside the mongo-0 pod in the local-cluster.
**Note:**Ctrl+r is used for searching
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- bin/sh
# env
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=mongo-0
HOME=/root
GPG_KEYS=9DA31620334BD75D9DCB49F368818C72E52529D4
TERM=xterm
MONGO_INITDB_ROOT_PASSWORD=password
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
MONGO_PACKAGE=mongodb-org
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MONGO_MAJOR=4.0
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
JSYAML_VERSION=3.13.0
GOSU_VERSION=1.11
MONGO_REPO=repo.mongodb.org
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
MONGO_INITDB_ROOT_USERNAME=admin1
MONGO_VERSION=4.0.8
```
**Note:** As we can see that the data is getting from the secret "MONGO_INITDB_ROOT_PASSWORD=password".

### 25.Just like we updated the configmap we can update the secret too this can be done by.

- To encode a text from a command line and encoding password with password123
```
ubuntu@balasenapathi:~$ echo -n password123 | base64
cGFzc3dvcmQxMjM=
```
### 26.Changing a secret file in local-cluster.
```
ubuntu@balasenapathi:~$ nano mongo-secret.yaml
apiVersion: v1
kind: Secret
metadata: 
  name: mongodb-secret
immutable: false
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=
```
### 27.Now apply the changes in the local-cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f mongo-secret.yaml
secret/mongodb-secret configured
```
### 28.Now delte the mongo-0 pod:
```
ubuntu@balasenapathi:~$ kubectl delete pod mongo-0
pod "mongo-0" deleted
```
### 29.The pods gets automatically created bcz we are using statefulset.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   1/1     Running   0          25s
mongo-1   1/1     Running   0          70m
mongo-2   1/1     Running   0          70m
```
### 30Now get into the mongo-0 pod and see the changes are reflected:
```
ubuntu@balasenapathi:~$ kubectl exec -it mongo-0 -- /bin/sh
# env
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=mongo-0
HOME=/root
GPG_KEYS=9DA31620334BD75D9DCB49F368818C72E52529D4
TERM=xterm
MONGO_INITDB_ROOT_PASSWORD=password123
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
MONGO_PACKAGE=mongodb-org
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MONGO_MAJOR=4.0
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
JSYAML_VERSION=3.13.0
GOSU_VERSION=1.11
MONGO_REPO=repo.mongodb.org
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
MONGO_INITDB_ROOT_USERNAME=admin1
MONGO_VERSION=4.0.8
```
**Note:** We can see that the password has been changed from password to password123. Please note that we
are encoding the data, not encrypting it. This means it can be easily decoded with echo -n. For example,
this is base64 encoded data: cGFzc3dvcmQxMjM=. To decode it, you would use the command:
```
ubuntu@balasenapathi:~$ echo -n cGFzc3dvcmQxMjM= | base64 --decode
password123
```
**Note:** password123 is the decoded data from the base64 encoded string cGFzc3dvcmQxMjM=.

So we should make sure to restrict access to Secrets, as they can be easily read by getting into the pods,
just like we read environment variables. We will discuss more about restricting access to Kubernetes 
resources later.

**Choosing between ConfigMap and Secret is very simple:**

- 1.If you want to store non-sensitive, plain configuration data, use a ConfigMap.
- 2.If you want to store any sensitive data, use a Secret.
- 3.If a configuration includes both sensitive and non-sensitive data, you should use Secrets.
























