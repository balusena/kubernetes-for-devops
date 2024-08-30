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

