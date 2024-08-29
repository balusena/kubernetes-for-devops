# Kubernetes Jobs and CronJobs:
Kubernetes Jobs are used to manage tasks that need to run to completion. They ensure that a specified number of pods 
execute successfully. This makes Jobs ideal for batch processing tasks, running scripts, or any task that needs to run
once until completion. Kubernetes will create the necessary pods and manage their execution to ensure the job completes
successfully, even restarting pods if needed.

CronJobs are an extension of Jobs and allow you to schedule tasks at specific times or intervals, similar to cron jobs 
in Unix-like systems. They are useful for automating repetitive tasks such as backups, report generation, or cleanup 
operations. CronJobs ensure that these tasks run automatically at the scheduled times, without the need for manual 
intervention. This makes them powerful tools for maintaining the regular operations of a Kubernetes cluster.





