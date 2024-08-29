# Kubernetes Role Based Access Control(RBAC):
Kubernetes Role-Based Access Control (RBAC) is a security mechanism that manages access to resources in a Kubernetes 
cluster. RBAC defines roles with specific permissions, and these roles are assigned to users or groups, controlling 
what actions they can perform. There are two main types of roles: ClusterRoles, which apply cluster-wide, and Roles, 
which apply within a specific namespace. RBAC uses RoleBindings and ClusterRoleBindings to associate roles with users 
or service accounts. By configuring RBAC, administrators can enforce the principle of least privilege, ensuring that 
users only have the permissions they need to perform their tasks.

So far, we have accessed everything in our Kubernetes cluster without any restrictions.However, in a real-time environment,
we typically have multiple nodes, namespaces, deployments, ReplicaSets, pods, services, and many other Kubernetes resources.
Additionally,there will be many users accessing these cluster resources.Without any restrictions,there's a risk of accidentally
deleting critical resources. Therefore, it's essential to impose restrictions on the creation, modification, and deletion of 
resources based on specific roles.

For example, we should ensure that developers can only deploy certain applications to a given namespace, our infrastructure
management teams have read-only access for monitoring tasks, and administrators have full access to perform all actions.

![Kubernetes RBAC Introduction](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_intro.png)

**Note:** We can restrict access to Kubernetes resources using the Role-Based Access Control (RBAC) framework.

### 1.Authentication and Authorization:
In general, to access any resource—whether it's a Kubernetes resource, REST API, AWS service, or any other service we 
must first prove that we are valid users by providing our credentials. This process is called **"authentication."** If
we do not have access to the requested resource, we will encounter a 401 error. If we are valid users, we will receive 
a 200 response.

However, proving our authenticity with credentials does not grant us access to everything. There are various actions we
may need to perform in the application, such as creating, updating, or deleting a resource. We must be authorized to 
perform a specific action. Therefore, after authentication, we are checked against our role to see if we are authorized 
to perform the requested action. This process is called **"authorization."** If we are not authorized, we will receive 
a 403 error; if we are authorized, we can complete the requested actions.

**Note:**
1. **Authentication**: We prove that we are valid users.
2. **Authorization**: We are checked to see if we can perform a specific task.

![Kubernetes RBAC Authentication Authorization](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_authentication_authorization.png)

There are different models through which we can achieve authorization, such as **Role-Based Access Control (RBAC)**, 
**Attribute-Based Access Control (ABAC)**, and **Node Authorization (NA)**. However, the most popular authorization 
model is **RBAC**. RBAC determines whether a user is allowed to perform a specific action on a given resource based 
on their role. The level of access varies depending on the role. For example, if you are a developer, you can create, 
read, and update resources; if you are from the monitoring team, you can only read resources; and if you are from the 
admin team, you can do everything.

**Note:** To understand how RBAC works, we first need to create a user.

### 1.Create User:
In Kubernetes, we cannot create users like we create pods or services because Kubernetes does not manage users internally.
Instead, user management should be handled by external identity platforms like Keycloak, AWS IAM, etc. However, Kubernetes
does handle authentication and authorization. When we perform any operation against our cluster, the request goes to the 
API server. The API server first authenticates whether you are a valid user. If you are valid, it then authorizes whether
you are allowed to perform that action. If authorized, you will receive the result of your request.

![Kubernetes RBAC User](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_user.png)

### 1.Create a simple nginx pod in minikube.
```
ubuntu@balasenapathi:~$ nano nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```
### 2.Apply the changes in the cluster.
```      
ubuntu@balasenapathi:~$ kubectl apply -f nginx-pod.yaml
pod/nginx-pod created
```
                            
**Users and Groups:**

### 1.Creating a user:

when we start the minikube cluster the information related to the cluster is stored in kube config file

### 2.Open the config file in nano editor:
```
ubuntu@balasenapathi:~$ nano ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/ubuntu/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Fri, 06 Oct 2023 20:19:36 IST
        provider: minikube.sigs.k8s.io
        version: v1.31.1
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Fri, 06 Oct 2023 20:19:36 IST
        provider: minikube.sigs.k8s.io
        version: v1.31.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/ubuntu/.minikube/profiles/minikube/client.crt
    client-key: /home/ubuntu/.minikube/profiles/minikube/client.key
```

- Here we have list of clusters that we have accessed,and this is our minikube information:
```
clusters:
- cluster:
    certificate-authority: /home/ubuntu/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Fri, 06 Oct 2023 20:19:36 IST
        provider: minikube.sigs.k8s.io
        version: v1.31.1
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
``` 
**Note:** We can have any number of clusters that we can connect it may be minikube, AWS EKS, Azure AKS etc...  

- We have multiple users here:
```
users:
- name: minikube
  user:
    client-certificate: /home/ubuntu/.minikube/profiles/minikube/client.crt
    client-key: /home/ubuntu/.minikube/profiles/minikube/client.key
```
**Note:** When we start the Minikube cluster, a Minikube user is automatically created. So far, we've accessed everything
in the cluster using this Minikube user.

- These are the contexts:
```
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Fri, 06 Oct 2023 20:19:36 IST
        provider: minikube.sigs.k8s.io
        version: v1.31.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
```
**Note:**  
A context in Kubernetes contains the cluster, the user, and the default namespace. The certificate authority is specified
below as "certificate-authority: /home/ubuntu/.minikube/ca.crt." Any user that provides a valid certificate signed by this
certificate authority is considered authenticated.

- **To authenticate against our Minikube,let's generate a certificate and sign this certificate with this certificate authority:**

### 3.First we should generate the users private key with openssl and store output file to the current folder:
```
ubuntu@balasenapathi:~$ openssl genrsa -out balu.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
...............................................+++++
.....................+++++
e is 65537 (0x010001)
```
**Note:** We can see that the private key is generated

### 4.Next we must create a certificate signing request for the user "balu" with above private key:
```
ubuntu@balasenapathi:~$ openssl req -new -key balu.key -out balu.csr -subj "/CN=balu/O=dev/O=example.org"
```
**Note:**
```
===> This is the command to generate CertificateSigningRequest (csr):
 
openssl req -new -key balu.key -out balu.csr -subj "/CN=balu/O=dev/O=example.org"

===> This is the private key we generated in the previous step:

balu.key -out balu.csr

===> CN stands for CommonName which acts like the username:

CN=balu

===> O acts like the groupname we can give multiple groups to the current user "balu" so now balu belongs to dev and example.org groups:

O=dev/O=example.org
```
**Note:** We are saving this CertificateSigningRequest (csr) to the current folder see the current folder below.

### 5.To get the details of present working directory.
```
ubuntu@balasenapathi:~$ pwd
/home/ubuntu ===> Current Folder/directory path
```
### 6.Our private key and CertificateSigningRequest(csr) are ready now verify them using:
```
ubuntu@balasenapathi:~$ ls | grep balu
balu.csr
balu.key
```
**Note:** We can see private key and CertificateSigningRequest(csr) are in Current Folder.

- Now this CertificateSigningRequest(csr) must be signed by the certificate authority .

They can get CertificateSigningRequest(csr) details from kube config file : "certificate-authority: /home/ubuntu/.minikube/ca.crt"

### 7.To sign it we should generate this command:
```
openssl x509 -req -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -days 730 -in balu.csr -out balu.crt
```
**Here are the details:** 
```
~/.minikube/ca.crt: This is Certificate Authority

~/.minikube/ca.key: This is Certificate Authority Key details 

balu.csr: This is CertificateSigningRequest(csr) that we generated 

balu.crt: This is the certificate we will generate and store in current folder
```
```
ubuntu@balasenapathi:~$ openssl x509 -req -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -days 730 -in balu.csr -out balu.crt
Signature ok
subject=CN = balu, O = dev, O = example.org
Getting CA Private Key
```
### 8.Next we should add the user "balu" to the cluster:
```
ubuntu@balasenapathi:~$ kubectl config set-credentials balu --client-certificate=balu.crt --client-key=balu.key
User "balu" set.
```
**Here are the details:** 
```
user ====> balu

certficate we generated ====> balu.crt

private key we generated ====> balu.key
```
**Note:** We can see that balu user is set and we can verify that in kube config file:
```
ubuntu@balasenapathi:~$ nano ~/.kube/config
users:
- name: balu
  user:
    client-certificate: /home/ubuntu/balu.crt
    client-key: /home/ubuntu/balu.key
- name: minikube
```
### 9.We should create the context with the same command:
```
ubuntu@balasenapathi:~$ kubectl config set-context balu-minikube --cluster=minikube --user=balu --namespace=default
Context "balu-minikube" created.
```
**Here are the details:**
```
context name ====> balu-minikube

cluster ====> minikube

user ====> balu

namespace ====> default
```
**Note:** We can see that balu context is set and we can verify that in kube config file:
```
ubuntu@balasenapathi:~$ nano ~/.kube/config
contexts:
- context:
    cluster: minikube
    namespace: default
    user: balu
  name: balu-minikube
```
### 10.We can also verify that by using:
```
ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
          balu-minikube   minikube   balu       default
*         minikube        minikube   minikube   default
```
**Note:** These are the two contexts in the cluster, and the * indicates that we are currently working with the default 
Minikube context. To use our newly created context, we need to switch to it, similar to switching between databases.

### 11.To switch the context.
```
ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
          balu-minikube   minikube   balu       default
*         minikube        minikube   minikube   default

ubuntu@balasenapathi:~$ kubectl config use-context balu-minikube
Switched to context "balu-minikube".

ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
*         balu-minikube   minikube   balu       default
          minikube        minikube   minikube   default
```
**Note:** The context has been switched from `minikube` to `balu-minikube`, as indicated by the *. Now, any request we 
send will be made on behalf of the `balu` user instead of the `minikube` user.

### 12.Now try to list down the pods in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "balu" cannot list resource "pods" in API group "" in the namespace "default"
```
**Note:** As we can see, the `balu` user is forbidden from listing the pods. This is because we just created the user,
and by default, they do not have any permissions to access cluster resources—they are just a skeleton user. This means 
`balu` is a valid user but is not authorized to perform any actions. As an administrator, we need to grant this user 
certain permissions. Let's switch back to the `minikube` user, who has access to everything, and then assign permissions
 to the `balu` user.

### 13.Now switch back to the minikube user:
```
ubuntu@balasenapathi:~$ kubectl config use-context minikube
Switched to context "minikube".

ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
          balu-minikube   minikube   balu       default
*         minikube        minikube   minikube   default
```

**Role and RoleBinding**

### 2.Role and RoleBinding:
In Kubernetes RBAC (Role-Based Access Control), a Role defines a set of permissions within a namespace, specifying which
actions (e.g., create, update, delete) can be performed on specific resources (e.g., pods, services). RoleBinding is used
to assign these roles to users, groups, or service accounts, effectively binding the permissions to the specified entities.
This means that a RoleBinding associates a Role with a user or group, allowing them to perform the actions defined in that
Role within the namespace. Together, Role and RoleBinding control access to resources in Kubernetes.

**Note:** Now we should give permissions to the balu user:

In kubernetes we can give permissions to a user with Roles and RoleBindings.

#### Creating a Role.

### 1.Create a simple role manifest file.
```
ubuntu@balasenapathi:~$ nano role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  verbs: ["get", "watch", "list"]
  resources: ["pods", "pods/log"]
  # resourceNames: ["nginx"]
```
**Note:** The `apiGroups` field identifies which API groups to target. This is necessary because different API groups 
can have the same resource names. You can compare this to packages in Java, where a package can have multiple classes,
and different packages can have classes with the same name. The empty string `[""]` in the list represents the core 
Kubernetes API.The `verbs` field `["get", "watch", "list"]`specifies the actions that can be performed on the specified
resources `["pods", "pods/log"]`.

### 2.To see what kind of verbs we can give for a particular user:
```
ubuntu@balasenapathi:~$ kubectl api-resources -o wide | grep pod
pods                               po              v1                                     true         Pod                               
create,delete,deletecollection,get,list,patch,update,watch   all
podtemplates                                       v1                                     true         PodTemplate                       
create,delete,deletecollection,get,list,patch,update,watch   
horizontalpodautoscalers           hpa             autoscaling/v2                         true         HorizontalPodAutoscaler           
create,delete,deletecollection,get,list,patch,update,watch   all
verticalpodautoscalercheckpoints   vpacheckpoint   autoscaling.k8s.io/v1                  true         VerticalPodAutoscalerCheckpoint   
delete,deletecollection,get,list,patch,create,update,watch   
verticalpodautoscalers             vpa             autoscaling.k8s.io/v1                  true         VerticalPodAutoscaler             
delete,deletecollection,get,list,patch,create,update,watch   
pods                                               metrics.k8s.io/v1beta1                 true         PodMetrics                        
get,list                                                     
poddisruptionbudgets               pdb             policy/v1                              true         PodDisruptionBudget               
create,delete,deletecollection,get,list,patch,update,watch   
```
**Note:** For pod we can give all these kind of verbs.

**Note:** Let's assign some of these verbs to a role. In our role manifest, we’ve specified the verbs `"get"`, `"watch"`,
and `"list"`. This means that anyone with this role can retrieve pod information, watch for pod changes, and list all 
the pods.

### 3.To switch the context.
```
ubuntu@balasenapathi:~$ kubectl config use-context minikube
Switched to context "minikube".

ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
          balu-minikube   minikube   balu       default
*         minikube        minikube   minikube   default
```
### 4.Now apply this manifest in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f role.yaml
role.rbac.authorization.k8s.io/pod-reader created
```
**Note:** The role is get created.

### 5.To verify the roles in the cluster:
```
ubuntu@balasenapathi:~$ kubectl get roles
NAME         CREATED AT
pod-reader   2023-10-08T18:35:37Z
```
**Note:** As we can see, the `pod-reader` role has been created. Now that we have both the user and the role ready, we 
need to connect them. This connection is made through a `RoleBinding`, which links the subject (user) to the role. This
ensures that the user has the specified permissions defined in the role.

#### Creating a RoleBinding and Attaching the Role and Subject to User "balu".

### Granting Permissions to the User "balu" Using RoleBinding.

In this RoleBinding, we connect both the Subject and the Role. The Subject can be a user, user group, or even a service 
account, and the Role defines what resources and actions the Subject can access or perform. When we attach the Subject 
and Role using RoleBinding, we are granting permissions to the user. In this case, we are giving the "balu" user the 
permission to read pods.

![Kubernetes RBAC Role RoleBinding](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_role_rolebinding.png)

### 1.Create a rolebinding in the cluster.
```
ubuntu@balasenapathi:~$ nano role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows user "balu" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
subjects:
# You can specify more than one "subject"
- kind: User
  name: balu # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
# roleRef:
#   kind: ClusterRole
#   name: secret-reader
#   apiGroup: rbac.authorization.k8s.io
```
**Note:** We are specifying `kind: RoleBinding` and assigning the user "balu" that we created earlier. The 
`apiGroup: rbac.authorization.k8s.io` identifies the RBAC group for the user, in this case, "balu." We are 
attaching this user to the role named "pod-reader" that we created previously. Essentially, a RoleBinding 
connects the user "balu" to the Role named "pod-reader."

### 2.Now apply this rolebinding in the cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f role-binding.yaml
rolebinding.rbac.authorization.k8s.io/read-pods created
```
### 3.Now lets witch back to the user balu-minikube:
```
ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
          balu-minikube   minikube   balu       default
*         minikube        minikube   minikube   default

ubuntu@balasenapathi:~$ kubectl config use-context balu-minikube
Switched to context "balu-minikube".

ubuntu@balasenapathi:~$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
*         balu-minikube   minikube   balu       default
          minikube        minikube   minikube   default
```
### 4.Now list down the pods to see whether balu-minikube is having permissions to list down the pods:          
```      
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
nginx                          1/1     Running   0             77s
traffic-generator              1/1     Running   3 (18h ago)   4d20h
utility-api-5b4b9b5776-srjfv   1/1     Running   0             66m
```
**Note:** We can now list down the pods because we attached the `pod-reader` role to the `balu` user.

### 5.Now lets create a pod in balu-minikube with user balu and see we can create a pod here:
```
ubuntu@balasenapathi:~$ kubectl run nginx --image=nginx
Error from server (Forbidden): pods is forbidden: User "balu" cannot create resource "pods" in API group "" in the namespace "default"
```
**Note:** As you can see, the user `balu` is forbidden to create pods. In the RoleBinding manifest file, we granted the 
`balu` user the verbs `"get"`, `"watch"`, and `"list"`, but not the `"create"` permission.

### 6.Now lets switch back to the minikube user:
```
ubuntu@balasenapathi:~$ kubectl config use-context minikube
Switched to context "minikube".
```
### 7.Now create a namespace "ns" and with namespace name "test" in minikube:
```
ubuntu@balasenapathi:~$ kubectl create ns test
namespace/test created
```
### 8.Now create a new pod in this namespace:
```
ubuntu@balasenapathi:~$ kubectl get ns
NAME              STATUS   AGE
default           Active   4d20h
kube-node-lease   Active   4d20h
kube-public       Active   4d20h
kube-system       Active   4d20h
test              Active   65s =====> newly created namespace with name test

ubuntu@balasenapathi:~$ kubectl run nginx --image=nginx -n test
pod/nginx created
```
**Note:** Now the same nginx pod is created in the test namespace also:

### 9.To verify this list down the pods in test namespace:
```
ubuntu@balasenapathi:~$ kubectl get pods -n test
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          4m49s
```
### 10.Now switch back to balu-minikube of user balu and try to access this pod in the test namespace:
```
ubuntu@balasenapathi:~$ kubectl config use-context balu-minikube
Switched to context "balu-minikube".

ubuntu@balasenapathi:~$ kubectl get pods -n test
Error from server (Forbidden): pods is forbidden: User "balu" cannot list resource "pods" in API group "" in the namespace "test"
``` 
**Note:** We received the same forbidden error because `Role` and `RoleBinding` are namespaced.This means a user will only
have permissions for the namespace where the `RoleBinding` is defined. If the user tries to access resources in another 
namespace where a `RoleBinding` does not exist, they will encounter an error. Previously, we defined the `RoleBinding` in 
the default namespace, but we attempted to access resources in the test namespace, where the `RoleBinding` was not created. 

![Kubernetes RBAC Role RoleBinding Issues](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_role_rolebinding_issues.png)

Managing `RoleBinding` for each namespace can be tedious, and some resources, such as `PersistentVolumes`, are not namespaced
and cannot have `RoleBindings`. To address this, Kubernetes provides `ClusterRole` and `ClusterRoleBinding`, which operate at
the cluster level rather than the namespace level. Unlike `RoleBinding`, which is restricted to a specific namespace, 
`ClusterRoleBinding` grants permissions across all namespaces in the cluster.

**ClusterRole and ClusterRoleBinding**

### 3.ClusterRole and ClusterRoleBinding:

**ClusterRole** and **ClusterRoleBinding** are Kubernetes resources used for managing access across the entire cluster.

**ClusterRole** defines a set of permissions, including actions (verbs) on resources (like pods, services) across all 
namespaces or at the cluster level. Unlike `Role`, which is namespace-scoped, `ClusterRole` is not limited to a specific
namespace.

**ClusterRoleBinding** associates a `ClusterRole` with users, groups, or service accounts, granting them the defined 
permissions across the entire cluster. It provides a way to bind roles to subjects at the cluster level, ensuring consistent
access control across all namespaces.

![Kubernetes RBAC Cluster Role Cluster RoleBinding](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_cluster_role_cluster_rolebinding.png)

### 1.Create a clusterrole in the cluster.
```
ubuntu@balasenapathi:~$ nano cluster-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  verbs: ["get", "watch", "list"]
  resources: ["pods", "pods/log"]
```
**Note:** As we can see, the `ClusterRole` is similar to the `Role` where we define the verbs (e.g., ["get", "watch", "list"])
and resources (e.g., ["pods", "pods/log"]). However, `ClusterRole` operates at the cluster level rather than being confined to
a specific namespace. Similarly to how we attached a `Role` to the user `balu`, a `ClusterRole` can be attached to users or 
service accounts to grant permissions across the entire cluster.

### 2.Create a clusterrolebinding in the cluster.
```
ubuntu@balasenapathi:~$ nano cluster-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "dev" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: pod-reader-global
subjects:
- kind: User
  name: balu # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
**Note:** Now we need to attach the `ClusterRole` to the user `balu`. For that, we use a `ClusterRoleBinding`, which 
references both the `ClusterRole` and the user `balu`. Apply these configurations to the cluster to grant the specified
permissions to the user.

### 3.Now apply the ClusterRole changes in the cluster.
```
ubuntu@balasenapathi:~$ kubectl apply -f  cluster-role.yaml
Error from server (Forbidden): error when retrieving current configuration of:
Resource: "rbac.authorization.k8s.io/v1, Resource=clusterroles", GroupVersionKind: "rbac.authorization.k8s.io/v1, Kind=ClusterRole"
Name: "pod-reader", Namespace: ""
from server for: "cluster-role.yaml": clusterroles.rbac.authorization.k8s.io "pod-reader" is forbidden: User "balu" cannot get resource 
"clusterroles" in API group "rbac.authorization.k8s.io" at the cluster scope
```
**Note:** The error occurs because the `balu-minikube` context does not have the necessary permissions to create a `ClusterRole`.
Ensure that you are using a context with sufficient privileges, such as an admin context, to create cluster-wide resources like
`ClusterRole`.

### 4.Now switch back to the minikube context.
```
ubuntu@balasenapathi:~$ kubectl config use-context minikube
Switched to context "minikube".
```
### 5.Now apply the ClusterRole changes in the minikube context.
```
ubuntu@balasenapathi:~$ kubectl apply -f cluster-role.yaml
clusterrole.rbac.authorization.k8s.io/pod-reader created
```
**Note:** We can see that the ClusterRole is created.

### 6.Now apply the ClusterRoleBinding changes in the minikube context.
```
ubuntu@balasenapathi:~$ kubectl apply -f cluster-role-binding.yaml
clusterrolebinding.rbac.authorization.k8s.io/pod-reader-global created
```
**Note:** We can see that the ClusterRoleBinding is also created.

### 7.Now switch back to the balu-minikube context.
```
ubuntu@balasenapathi:~$ kubectl config use-context balu-minikube
Switched to context "balu-minikube".
```
### 8.Now try to access or get the pod from test namespace we created.
```
ubuntu@balasenapathi:~$ kubectl get pod -n test
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          152m
```
**Note:** As we can see, we are now able to list pods from different namespaces, not just the "test" namespace. The 
`ClusterRole` and `ClusterRoleBinding` allow us to access pods across all namespaces.

### 9.To get the pods from all the namespaces in the cluster.
```
ubuntu@balasenapathi:~$ kubectl get pods -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS        AGE
default       nginx                                       1/1     Running   0               170m
default       traffic-generator                           1/1     Running   3 (20h ago)     4d23h
default       utility-api-5b4b9b5776-srjfv                1/1     Running   0               3h55m
kube-system   coredns-5d78c9869d-k9glg                    1/1     Running   3 (20h ago)     4d23h
kube-system   etcd-minikube                               1/1     Running   3 (20h ago)     4d23h
kube-system   kube-apiserver-minikube                     1/1     Running   3 (20h ago)     4d23h
kube-system   kube-controller-manager-minikube            1/1     Running   4 (20h ago)     4d23h
kube-system   kube-proxy-m7mpq                            1/1     Running   3 (20h ago)     4d23h
kube-system   kube-scheduler-minikube                     1/1     Running   3 (20h ago)     4d23h
kube-system   metrics-server-844d8db974-j67w4             1/1     Running   6 (3h56m ago)   4d23h
kube-system   storage-provisioner                         1/1     Running   7 (3h57m ago)   4d23h
kube-system   vpa-admission-controller-754ccfdf99-2fd6t   1/1     Running   3 (20h ago)     4d23h
kube-system   vpa-recommender-667f9769fb-lsfkd            1/1     Running   3 (20h ago)     4d23h
kube-system   vpa-updater-696b8787f9-rt95f                1/1     Running   3 (20h ago)     4d23h
test          nginx                                       1/1     Running   0               155m
```
**Note:** This is because we have defined the `ClusterRoleBinding` for the "balu" user. Now, we know how to grant access
to a user across the cluster.

**Groups**

### 4.Groups:
RBAC groups in Kubernetes allow you to define permissions for multiple users collectively. By creating a group and assigning
roles to it, you can manage access for all members of that group at once, simplifying the administration of permissions and 
ensuring consistency across users with similar responsibilities.

**Note:**
Handling hundreds of users individually in a Kubernetes cluster can be cumbersome, especially when assigning permissions.
Instead of managing each user separately, you can use Groups. A Group is essentially a collection of users. By assigning
users to a Group, you can manage their permissions collectively.

For example, instead of assigning permissions to the user "balu" directly, you can assign them to a Group, such as "dev".
When you create the user "balu", you associate them with the "dev" Group. This way, any permissions granted to the "dev"
Group automatically apply to "balu" and any other users in that group.

### 1.Create cluster-role-binding in the cluster.
```
ubuntu@balasenapathi:~$ nano cluster-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "dev" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: pod-reader-global
subjects:
- kind: User
  name: balu # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: dev # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
**Note:**
As a homework assignment, try creating multiple users and attaching them to the same group, such as "dev". After setting
this up, check if these users can access the resources assigned to the "dev" group. This will help you understand how 
Group-based permissions work in Kubernetes RBAC and how they can simplify managing access for multiple users.

### 2.Now apply the cluster-role-binding changes in the cluster
```
ubuntu@balasenapathi:~$ kubectl apply -f cluster-role-binding.yaml
clusterrolebinding.rbac.authorization.k8s.io/pod-reader-global configured
```
**Note:** We have seen how a physical user like "balu" can access Kubernetes resources, but what if a Spring Boot or 
Python application wants to access the same resources? Providing our credentials directly in the application isn't safe.
To securely manage access for applications, Kubernetes provides **Service Accounts**.

**Service Accounts**

### 5.Service Accounts:
In Kubernetes, **Service Accounts** are used to grant specific permissions to applications or processes running inside pods.
Unlike user accounts, which are for human users, service accounts are designed for system components and applications. With
RBAC (Role-Based Access Control), you can assign roles to service accounts, controlling what resources and actions the 
applications can access within the cluster. This ensures secure, fine-grained control over how applications interact with
Kubernetes resources.

![Kubernetes RBAC Service Accounts](https://github.com/balusena/kubernetes-for-devops/blob/main/16-Kubernetes%20Role%20Based%20Access%20Control/rbac_service_accounts.png)

Service Accounts are special types of users in Kubernetes, often referred to as application users. When a namespace is 
created, a default service account is automatically generated within it. Pods, where our applications run, use these 
default service accounts to authenticate with the API server. If no specific service account is mentioned, all pods will
use this default account. However, you can also create custom service accounts within these namespaces to provide more 
tailored access control for different applications.

### 1.To get the service account in cluster.
```
ubuntu@balasenapathi:~$ kubectl get sa
Error from server (Forbidden): serviceaccounts is forbidden: User "balu" cannot list resource "serviceaccounts" in API group "" in the 
namespace "default"
```
### 2.Now switch back to the minikube context.
```
ubuntu@balasenapathi:~$ kubectl config use-context minikube
Switched to context "minikube".
```
```
ubuntu@balasenapathi:~$ kubectl get sa
NAME      SECRETS   AGE
default   0         5d1h
``` 
**Note:** We can see that the default service account is there in default namespace in minikube.

### 3.We can also find this namespace in test namespace.
```
ubuntu@balasenapathi:~$ kubectl get sa -n test
NAME      SECRETS   AGE
default   0         4h52m
```
### 4.To create a service account in minikube.
```
ubuntu@balasenapathi:~$ kubectl create sa test-sa
serviceaccount/test-sa created
```
### 5.To get the service account in cluster.
```
ubuntu@balasenapathi:~$ kubectl get sa
NAME      SECRETS   AGE
default   0         5d1h
test-sa   0         9s
```
**Note:** A new service account "test-sa" is created in the minikube.

### 6.Now create a simple kubectl-pod and try to get the list of pods from this pod:
```
ubuntu@balasenapathi:~$ nano kubectl-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-pod
spec:  
  containers:
  - name: kubectl
    image: bitnami/kubectl
    command: ["sleep", "20000"]
```
### 7.Apply the changes in the minikube.
```
ubuntu@balasenapathi:~$ kubectl apply -f kubectl-pod.yaml
pod/kubectl-pod created
```
### 8.Now get into the kubectl-pod.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
kubectl-pod                    1/1     Running   0             92s
nginx                          1/1     Running   0             5h11m
traffic-generator              1/1     Running   3 (23h ago)   5d1h
utility-api-5b4b9b5776-srjfv   1/1     Running   0             6h16m
```
### 9.Now try to list down the pods.
```
ubuntu@balasenapathi:~$ kubectl exec -it kubectl-pod -- bash
I have no name!@kubectl-pod:/$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group 
"" in the namespace "default"
```
**Note:** The error with the reason "forbidden" occurred because the pod is trying to use the default service account,
which does not have permission to list pods. To resolve this, we should use a custom service account, such as the `test`
service account, which has the necessary permissions for this pod.

### 10.Now make changes in kubectl-pod.
```
ubuntu@balasenapathi:~$ nano kubectl-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-pod
spec:
  serviceAccount: test-sa
  containers:
  - name: kubectl
    image: bitnami/kubectl
    command: ["sleep", "20000"]
```    
**Note:** To use custom service account "test-sa" we need to specify this spec: section in kubectl-pod manifest file.
### 11.Now delete the pod in the cluster.
```
ubuntu@balasenapathi:~$ kubectl delete -f kubectl-pod.yaml
pod "kubectl-pod" deleted
```
### 12.Now apply the changes in the cluster to create the pod.
```
ubuntu@balasenapathi:~$ kubectl apply -f kubectl-pod.yaml
pod/kubectl-pod created
```
### 13.Now try to get into the pods and try to list down the pods.
```
ubuntu@balasenapathi:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
kubectl-pod                    1/1     Running   0             33s
nginx                          1/1     Running   0             5h54m
traffic-generator              1/1     Running   3 (24h ago)   5d2h
utility-api-5b4b9b5776-srjfv   1/1     Running   0             6h59m

ubuntu@balasenapathi:~$ kubectl exec -it kubectl-pod -- bash
I have no name!@kubectl-pod:/$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:test-sa" cannot list resource "pods" in API group 
"" in the namespace "default"
```
**Note:** The error persists, but this time it is using the `test-sa` service account in the default namespace. The issue
is that although we created the service account, we did not attach any roles to it, which means it still lacks the necessary
permissions.

### 14.To add permissions to this service account "test-sa" for this make changes in role-binding.yaml:
```
ubuntu@balasenapathi:~$ nano role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows user "balu" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
subjects:
# You can specify more than one "subject"
- kind: User
  name: balu # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: test-sa
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
# roleRef:
#   kind: ClusterRole
#   name: secret-reader
#   apiGroup: rbac.authorization.k8s.io
```
**Note:** We have added the Kind: ServiceAccount and name: test-sa in role-binding.yaml

### 15.Apply the changes in the cluster:
```
ubuntu@balasenapathi:~$ kubectl apply -f role-binding.yaml
rolebinding.rbac.authorization.k8s.io/read-pods configured
```
### 16.Now try to list down the pods after updating the role-binding.yaml:
```
ubuntu@balasenapathi:~$ kubectl exec -it kubectl-pod -- bash
I have no name!@kubectl-pod:/$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:test-sa" cannot list resource "pods" in API group 
"" in the namespace "default"
I have no name!@kubectl-pod:/$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
kubectl-pod                    1/1     Running   0             12m
nginx                          1/1     Running   0             6h5m
traffic-generator              1/1     Running   3 (24h ago)   5d2h
utility-api-5b4b9b5776-srjfv   1/1     Running   0             7h10m
```
**Note:** Now we are able to list down the pods with the `test-sa` service account. This demonstrates that pods use 
service accounts to access Kubernetes resources, with appropriate permissions configured.

### 17.Now come out of pod and see if we can perform action with a command.
```
ubuntu@balasenapathi:~$ kubectl auth can-i create pods
yes
```
**Note:** It is saying that we can able to create pods because we are in minikube context.

### 18.To test if a service account is having permission or not.
```
ubuntu@balasenapathi:~$ kubectl auth can-i create pods --as="system:serviceaccount:default:test-sa"
no
```
**Note:** Here swe got the response no that means we cannot create pods with this service account "test-sa"

### 19.Now lets try to see if we can get the pods.
```
ubuntu@balasenapathi:~$ kubectl auth can-i get pods --as="system:serviceaccount:default:test-sa"
yes
```
**Note:** We can get the pods with this service account "test-sa".

**So if you want a real quick check if a user or service account is having a permission or not we can give this "auth can-i".**










