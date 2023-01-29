Create a CronJob mycj with image busybox and schedule it to run every 7 minutes.
The container should execute the command "echo Hello from mycj".
The CronJob should keep history for no more than 5 successful runs and 4 failures.
<details>
  <summary>Answer</summary>

```
kubectl create cj mycj --image=busybox --schedule="*/7 * * * *" --dry-run=client -o yaml > mycj.yml -- echo "Hello from mycj"
vim mycj.yml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: mycj
spec:
  successfulJobsHistoryLimit: 5 # add this line
  failedJobsHistoryLimit: 4 # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: mycj
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - echo
            - Hello from mycj
            image: busybox
            name: mycj
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/7 * * * *'
status: {}
kubectl create -f mycj.yml
```
</details>
<p>&nbsp;</p>

Edit the CronJob and set the schedule to execute every 2 minutes.
<details>
  <summary>Answer</summary>

```
kubectl edit cj mycj
...
  schedule: '*/2 * * * *'
...
```
</details>
<p>&nbsp;</p>

Check the logs of the Pods to confirm that the expected message is displayed.
<details>
  <summary>Answer</summary>

```
kubectl mycj-27898027-fnzcz
```
</details>
<p>&nbsp;</p>

Create a CronJob named parallel. Allow the cronjob to run Jobs in parallel. The image should be busybox with command "echo Running in parallel && sleep 15m".
The CronJob should be scheduled to run every 3 minutes.
<details>
  <summary>Answer</summary>

```
kubectl create cj parallel --image busybox --schedule="*/3 * * * *" --dry-run=client -o yaml > parallel.yml -- /bin/sh -c "echo Running in parallel && sleep 15m"
vim parallel.yml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: parallel
spec:
  concurrencyPolicy: Allow # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: parallel
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - echo Running in parallel && sleep 15m
            image: busybox
            name: parallel
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/3 * * * *'
status: {}
kubectl create -f parallel.yml
```
</details>
<p>&nbsp;</p>

After 10 minutes there should be Jobs running in parallel created by parallel CronJob.
<details>
  <summary>Answer</summary>

```
kubectl get jobs
```
</details>
<p>&nbsp;</p>

Suspend the parallel CronJob.
<details>
  <summary>Answer</summary>

```
kubectl edit cj parallel
...
  suspend: true
...
```
</details>
<p>&nbsp;</p>

Check the status of the CronJob and delete it.
<details>
  <summary>Answer</summary>

```
kubectl get cj parallel
kubectl delete cj parallel
```
</details>
<p>&nbsp;</p>
