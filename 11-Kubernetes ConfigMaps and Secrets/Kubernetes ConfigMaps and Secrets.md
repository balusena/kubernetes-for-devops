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

## 2.Creating and Managing ConfigMaps:
**There are two ways of creating a config map:**

- 1.The imperative way – without using a ConfigMap definition file.
- 2.Declarative way by using a Configmap definition file.

**1.Imperative Way:Creating a ConfigMap Without Using a Definition File.**
The imperative way involves using kubectl commands to create a ConfigMap directly from the command line 
without defining a ConfigMap YAML file.

### 1.Creating a ConfigMap from Literal Values
You can create a ConfigMap from literal values using the kubectl create configmap command. 

- This example creates a ConfigMap named my-config with two key-value pairs.
```
ubuntu@balasenapathi:~$ kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
configmap/my-config created
```
### 2.Creating a ConfigMap from a File
You can create a ConfigMap from a file using the --from-file option. 

- This example creates a ConfigMap named my-config from a file called config.txt.
```
ubuntu@balasenapathi:~$ kubectl create configmap my-config --from-file=config.txt
configmap/my-config created
```
### 3.Creating a ConfigMap from a Directory
If you have multiple files in a directory and want to include all of them in a ConfigMap,you can use the --from-file option with a directory. 

- This example creates a ConfigMap named my-config from all files in the ./config-dir directory.
```
ubuntu@balasenapathi:~$ kubectl create configmap my-config --from-file=config-dir/
configmap/my-config created
```
**2.Declarative Way:Creating a ConfigMap Using a Definition File.**
The declarative way involves defining a ConfigMap in a YAML file and then applying it using kubectl. This
method is more suited for managing configurations as code and versioning.

### 1. creating a ConfigMap named app-config with key-value pairs using YAML.
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
**Using ConfigMaps in Pods:**

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
2. Using ConfigMaps as Mounted Volumes:

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

