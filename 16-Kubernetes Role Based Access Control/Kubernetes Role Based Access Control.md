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






