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

In all these situations the pod will look like its running from the outside but internal functionality is
broken because of these bugs
