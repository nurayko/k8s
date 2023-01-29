Create the Job lifter with image busybox that executes the command "echo Heavy Lifting && sleep 10m".
The Job should execute up to 6 times successfully.
<details>
  <summary>Answer</summary>

```
kubectl create job lifter --image=busybox --dry-run=client -o yaml > lifter.yml -- /bin/sh -c "echo Heavy Lifting && sleep 10m"
vim lifter.yml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: lifter
spec:
  completions: 6 # add this line
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - echo Heavy Lifting && sleep 10m
        image: busybox
        name: lifter
        resources: {}
      restartPolicy: Never
status: {}
kubectl create -f lifter.yml
```
</details>
<p>&nbsp;</p>

Check the logs of the running Pod and make sure the message is displayed.
<details>
  <summary>Answer</summary>

```
kubectl logs lifter-5slfw
```
</details>
<p>&nbsp;</p>

Edit the Job to have 3 Pods run in parallel.
<details>
  <summary>Answer</summary>

```
kubectl edit job lifter
# .....
spec:
  backoffLimit: 6
  completionMode: NonIndexed
  completions: 6
  parallelism: 3 # edit this line
  selector:
# .....
```
</details>
<p>&nbsp;</p>

Check if the new Pods have started.
<details>
  <summary>Answer</summary>

```
kubectl get pods
```
</details>
<p>&nbsp;</p>

Delete the Job.
<details>
  <summary>Answer</summary>

```
kubectl delete job lifter
```
</details>
<p>&nbsp;</p>

Create a Job nap with image busybox that runs the command "echo Taking a nap && sleep 30".
The job should run 4 times successfully having 2 Pods run in parallel. The job should delete itself 30 seconds after it is finished.
<details>
  <summary>Answer</summary>

```
kubectl create job nap --image busybox --dry-run=client -o yaml > nap.yml -- /bin/sh -c "echo Taking a nap && sleep 30"
vim nap.yml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: nap
spec:
  completions: 4 # add this line
  parallelism: 2 # add this line
  ttlSecondsAfterFinished: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - echo Taking a nap && sleep 30
        image: busybox
        name: nap
        resources: {}
      restartPolicy: Never
status: {}
kubectl apply -f nap.yml
```
</details>
<p>&nbsp;</p>

Check if the 2 Pods if the Job are running.
<details>
  <summary>Answer</summary>

```
kubectl get po
```
</details>
<p>&nbsp;</p>

Check the logs of the Pods.
<details>
  <summary>Answer</summary>

```
kubectl log nap-cbw9f
```
</details>
<p>&nbsp;</p>

After around 90 seconds the Job should complete, and 30 seconds after that it should be deleted. Check the Job.
<details>
  <summary>Answer</summary>

```
kubectl get job nap
```
</details>
<p>&nbsp;</p>

Create a Job myjob with image busybox that runs the command "echo This is my job. && sleep 10m".
The Job should run until 3 completions, and it should retry failures 2 times at most.
The Job should be active for 90 seconds, if it isn't completed until then it should be considered failed.
<details>
  <summary>Answer</summary>

```
kubectl create job myjob --image busybox --dry-run=client -o yaml > myjob.yml -- /bin/sh -c "echo This is my job. && sleep 10m"
vim myjob.yml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: myjob
spec:
  completions: 3 # add this line
  backoffLimit: 2 # add this line
  activeDeadlineSeconds: 90 # add this line
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - echo This is my job. && sleep 10m
        image: busybox
        name: myjob
        resources: {}
      restartPolicy: Never
status: {}
kubectl apply -f myjob.yml
```
</details>
<p>&nbsp;</p>

Check the logs of the Pod for the message
<details>
  <summary>Answer</summary>

```
kubectl logs myjob-cnsp6
```
</details>
<p>&nbsp;</p>

After 90 seconds of the Job it should considered failed. The activeDeadlineSeconds overrides the backoffLimit and the job won't execute and start Pods anymore.
Check the reason and the message.
<details>
  <summary>Answer</summary>

```
kubectl describe job myjob
#   Warning  DeadlineExceeded  2m47s  job-controller          Job was active longer than specified deadline
```
</details>
<p>&nbsp;</p>

Delete the Job.
<details>
  <summary>Answer</summary>

```
kubectl delete -f myjob.yml
```
</details>
<p>&nbsp;</p>