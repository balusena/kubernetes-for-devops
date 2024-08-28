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




