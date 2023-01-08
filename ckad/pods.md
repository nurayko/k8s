Create a Pod with name nginx-app that is running the nginx:1.23.1-alpine image with label app=nginx.
```
kubectl run nginx-app --image=nginx:1.23.1-alpine --labels=app=nginx
```

Create a Pod with name sleeper that is running the busybox:latest image. It should execute command to print envinronment variables and sleep for 10 minutes.
Check the logs of the container to verify it's working/
```
kubectl run sleeper --image=busybox:latest -- /bin/sh -c "env && sleep 600"
kubectl logs sleeper
```

Create a Pod with name limited that is running the nginx:1.23.1-alpine image. The pod should request 200Mi memory and 200m cpu. The limits should be twice the amount of the request.
```
kubectl run limited --image=nginx:1.23.1-alpine --dry-run=client -o yaml > tmp.yml
vim tmp.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: limited
  name: limited
spec:
  containers:
  - image: nginx:1.23.1-alpine
    name: limited
    resources:
      requests: # add this line
        memory: "200Mi" # add this line
        cpu: "200m" # add this line
      limits: # add this line
        memory: "400Mi" # add this line
        cpu: "400m" # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl apply -f tmp.yml
```

Create a Pod with name secured that is running the redis image. The container should run as user 1000, run as group 2000 and it should not allow privilege escalation.
```
kubectl run secured --image=redis --dry-run=client -o yaml > redis.yml
vim tmp.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secured
  name: secured
spec:
  containers:
  - image: redis
    name: secured
    resources: {}
    securityContext: # add this line
      runAsUser: 1000 # add this line
      runAsGroup: 2000 # add this line
      allowPrivilegeEscalation: false # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl apply -f redis.yml
```


Create a generic secret named credentials with data username=admin and password=admin.
Create a Pod with name nginx running the nginx:latest image that mounts this secret as a volume at /tmp .
Make sure the secret is mounted properly.
```
kubectl create secret generic credentials --from-literal=username=admin --from-literal=password=admin
kubectl run nginx --image=nginx:latest --dry-run=client -o yaml > nginx.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add this line
  - name: secret # add this line
    secret: # add this line
      secretName: credentials # add this line
  containers:
  - image: nginx:latest
    name: nginx
    resources: {}
    volumeMounts: # add this line
    - name: secret # add this line
      mountPath: "/tmp" # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl apply -f nginx.yml
kubectl get po nginx
kubectl exec nginx -- ls /tmp
```

List all running Pods and their labels in the dev namespace.
```
kubectl get po --show-labels -n dev
```

List all running Pods sorted by their age.
```
kubectl get po --sort-by={.metadata.creationTimestamp}
```

Label all Pods that have label app: fe or app: web with label ui: true in the dev namespace.
```
kubectl label po ui=true -l "app in (fe,web)" -n dev
```

Show only the status of the running Pod pod15 in the default namespace.
```
kubectl get po pod15 -o jsonpath="{.status.phase}"
```
