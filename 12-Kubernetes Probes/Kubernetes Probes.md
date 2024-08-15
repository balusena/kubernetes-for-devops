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
- 1.Liveness Probe
- 2.Readiness Probe
- 3.Startup Probe

balu you are great!


