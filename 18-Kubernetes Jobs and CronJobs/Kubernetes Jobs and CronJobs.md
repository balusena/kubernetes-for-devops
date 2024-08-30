# Kubernetes Jobs and CronJobs:
Kubernetes Jobs are used to manage tasks that need to run to completion. They ensure that a specified number of pods 
execute successfully. This makes Jobs ideal for batch processing tasks, running scripts, or any task that needs to run
once until completion. Kubernetes will create the necessary pods and manage their execution to ensure the job completes
successfully, even restarting pods if needed.

CronJobs are an extension of Jobs and allow you to schedule tasks at specific times or intervals, similar to cron jobs 
in Unix-like systems. They are useful for automating repetitive tasks such as backups, report generation, or cleanup 
operations. CronJobs ensure that these tasks run automatically at the scheduled times, without the need for manual 
intervention. This makes them powerful tools for maintaining the regular operations of a Kubernetes cluster.

In Kubernetes, we have different controllers for managing pods, such as ReplicaSets, Deployments, StatefulSets, and DaemonSets.
These controllers ensure that their pods are always running. If a pod fails, the controller restarts it or reschedules it to 
another node to maintain the desired number of running pods. However, there are scenarios where you may want to run your pods 
only once, such as taking database backups or sending emails in a batch. These processes should not be running continuously; 
they should run for a specific amount of time and at particular intervals.

Using controllers like Deployments for such tasks is not ideal, as they ensure that the pod runs continuously. Instead, for these
kinds of batch jobs, you can use Jobs and CronJobs in Kubernetes. Jobs allow you to run processes only once, while CronJobs enable
you to schedule them to run at specific intervals.

![Kubernetes Jobs CronJobs](https://github.com/balusena/kubernetes-for-devops/blob/main/18-Kubernetes%20Jobs%20and%20CronJobs/jobs_cronjobs_intro.png)

# Kubernetes Jobs Lifecycle:
When a Kubernetes job is created, it launches a pod with the image specified in the manifest. This image can be any application
that contains the logic for our use case. If there are errors in running the application in the pod due to reasons such as memory
or CPU issues, the job retries a given number of times by recreating the pod. The number of retries can be specified using the 
`backoffLimit`. For example, if we set the `backoffLimit` to three (3) and the pod fails, Kubernetes will create a new pod to 
retry the task. If the new pod also fails, Kubernetes will create another pod, and so on, until three (3) attempts have been made.
If the job still fails after three (3) attempts, Kubernetes will mark the job as failed with a "BackoffLimitExceeded" error.

The `backoffLimit` field is very useful to ensure that our job is not retried indefinitely, which could waste resources and cause
unnecessary load on the system. If we don’t set anything, the `backoffLimit` defaults to six (6).

Another important property in the job manifest file is `activeDeadlineSeconds`. This property defines the maximum duration
for which a job should run. If the job runs longer than this period, it will be marked as incomplete with a "DeadlineExceeded"
error, regardless of how many pods are created. Note that `activeDeadlineSeconds` takes precedence over `backoffLimit`.

If there are no errors in the pod, the job controller checks the number of pod completions. This can be specified in the job
manifest, indicating the minimum number of successful runs required. This completion count in the job is similar to replicas
in deployments. The job will continue to spin up new pods until the required completions are met. Once the specified number
of successful completions is reached, the job will be marked as completed.

When we set `completions` > 1, by default, pods are created one by one sequentially. However, we can create pods in parallel
by setting the `parallelism` value. For example, if `parallelism` is set to two (2) and `completions` to three (3), two (2)
pods will be created initially. Once they are completed, another one (1) pod will be created. This is the lifecycle of a job.
These jobs are executed only once, meaning when a job is created, the pod is created, the execution runs in the pod, and the 
pod is then completed.

However, sometimes we want to run a job on a scheduled basis, such as for database backups where we want to back up our database
every day at midnight. For this, Kubernetes provides another resource called a `CronJob`. In a `CronJob`, we can specify a cron
expression to define the schedule. When the `CronJob` is created, Kubernetes creates a job according to the specified schedule,
and from there, the same job flow applies.

In short, Kubernetes Jobs and CronJobs are useful for automating and managing a wide variety of tasks in a containerized environment.
Jobs and CronJobs can be used in multiple use cases.

![Kubernetes Jobs Lifecycle](https://github.com/balusena/kubernetes-for-devops/blob/main/18-Kubernetes%20Jobs%20and%20CronJobs/kubernetes_jobs_lifecycle.png)













