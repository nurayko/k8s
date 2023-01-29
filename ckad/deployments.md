Create a Deployment webserver running the nginx:alpine image with 3 replicas.
<details>
  <summary>Answer</summary>

```
kubectl create deploy webserver --image=nginx:alpine --replicas=3
```
</details>
<p>&nbsp;</p>

Change the image of webserver Deployment to nginx:latest and record the changes.
<details>
  <summary>Answer</summary>

```
kubectl set image deploy webserver nginx=nginx:latest --record
```
</details>
<p>&nbsp;</p>

Rollback the webserver Deployment to it's previous version
<details>
  <summary>Answer</summary>

```
kubectl rollout undo deploy webserver
```
</details>
<p>&nbsp;</p>

Create a Deployment busy with image busybox. The Deployment should execute the command touch /tmp/ready && sleep 1d and implement readiness probe which executes the command cat /tmp/ready every 15 seconds.
The readiness probe should have initial delay of 10 seconds.
<details>
  <summary>Answer</summary>

```
kubectl create deploy busy --image=busybox --dry-run=client -o yaml -- /bin/sh -c "touch /tmp/ready && sleep 1d" > busy.yml
vim busy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: busy
  name: busy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: busy
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - touch /tmp/ready && sleep 1d
        image: busybox
        name: busybox
        resources: {}
        readinessProbe: # add this line
          exec: # add this line
            command: # add this line
            - cat # add this line
            - /tmp/ready # add this line
          initialDelaySeconds: 10 # add this line
          periodSeconds: 15 # add this line
status: {}
kubectl create -f busy.yml
# after ~25 seconds the Deployment should be 1/1 Ready
kubectl get deploy busy
```
</details>
<p>&nbsp;</p>

Create a Deployment with name redis using the redis:latest image. Change the Deployment strategy to Recreate. Make sure the Pods are not running as root.
<details>
  <summary>Answer</summary>

```
kubectl create deploy redis --image=redis:latest --replicas=2 --dry-run=client -o yaml > rd.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: redis
  name: redis
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
  strategy: # edit this line
    type: Recreate # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: redis
    spec:
      securityContext: # add this line
        runAsNonRoot: false # add this line
      containers:
      - image: redis:latest
        name: redis
        resources: {}
status: {}
kubectl create -f rd.yml
```
</details>
<p>&nbsp;</p>

Scale the redis Deployment to have 4 replicas.
<details>
  <summary>Answer</summary>

```
kubectl scale deploy redis --replicas=4
```
</details>
<p>&nbsp;</p>

Create a ConfigMap with name petnames and key-value pairs:
dog=Rock
cat=Smith
Mount the ConfigMap as a volume at /tmp/pets in the new Deployment pets which should run busybox image with 1 replica and the following command:
while true ; do test -d /tmp/pets && cat /tmp/pets/* ; sleep 30 ; done
<details>
  <summary>Answer</summary>

```
kubectl create configmap petnames --from-literal=dog=Rock --from-literal=cat=Smith
kubectl create deploy pets --image=busybox --replicas=1 --dry-run=client -o yaml -- /bin/sh -c "while true ; do test -d /tmp/pets && cat /tmp/pets/* ; sleep 30 ; done" > pets.yml
vim pets.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: pets
  name: pets
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pets
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pets
    spec:
      volumes: # add this line
      - name: pets # add this line
        configMap: # add this line
          name: petnames # add this line
      containers:
      - command:
        - /bin/sh
        - -c
        - while true ; do test -d /tmp/pets && cat /tmp/pets/* ; sleep 30 ; done
        image: busybox
        name: busybox
        resources: {}
        volumeMounts: # add this line
        - name: pets # add this line
          mountPath: /tmp/pets # add this line
status: {}
kubectl create -f pets.yml
```
</details>
<p>&nbsp;</p>