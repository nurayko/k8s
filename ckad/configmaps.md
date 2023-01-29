Create the other-app-config ConfigMap with key-value pairs other-app-address=other-app-svc and other-app-port=8082 .
<details>
  <summary>Answer</summary>

```
kubectl create cm other-app-config --from-literal other-app-address=other-app-svc --from-literal other-app-port=8082
```
</details>
<p>&nbsp;</p>
Run the Pod mypod with image nginx:alpine and add the data of other-app-config as environment variables .

<details>
  <summary>Answer</summary>

```
kubectl run mypod --image nginx:alpine --dry-run=client -o yaml > mypod.yml
vim mypod.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypod
  name: mypod
spec:
  containers:
  - image: nginx:alpine
    name: mypod
    resources: {}
    envFrom: # add this line
    - configMapRef: # add this line
        name: other-app-config # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
kubectl create -f mypod.yml
```

</details>
<p>&nbsp;</p>
Check if the environment variables are present in the container of the Pod.
<details>
  <summary>Answer</summary>

```
kubectl exec -it mypod -- /bin/sh
env | grep other
```

</details>
<p>&nbsp;</p>
Delete the Pod and the ConfigMap.
<details>
  <summary>Answer</summary>

```
kubectl delete -f mypod.yml
kubectl delete cm other-app-config
```

</details>
<p>&nbsp;</p>
Create the file log.properties with content log.directory=/var/log , rotation=7 , size=8192 .
Create the log-properties ConfigMap using the file.
Create the Pod web with image httpd:alpine and mount log-properties as Volume under /tmp/config.
<details>
  <summary>Answer</summary>

```
printf "%s\n" log.directory=/var/log rotation=7 size=8192 > log.properties
kubectl create cm log-properties --from-file=log.properties
kubectl run web --image httpd:alpine --dry-run=client -o yaml > web.yml
vim web.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: web
spec:
  volumes: # add this line
  - name: properties # add this line
    configMap: # add this line
      name: log-properties # add this line
  containers:
  - image: httpd:alpine
    name: web
    resources: {}
    volumeMounts: # add this line
    - name: properties # add this line
      mountPath: /tmp/config # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
kubectl create -f web.yml
```

</details>
<p>&nbsp;</p>
Check if the file has been mounted in the Pod.
<details>
  <summary>Answer</summary>

```
kubectl exec web -- /bin/sh -c "ls /tmp/config && cat /tmp/config/log.properties"
```

</details>
<p>&nbsp;</p>
Edit the ConfigMap and change the rotation property to 10.
<details>
  <summary>Answer</summary>

```
kubectl edit cm log-properties
data:
  log.properties: |
    log.directory=/var/log
    rotation=10 # change this line
    size=8192

```

</details>
<p>&nbsp;</p>
Check again if the changes have taken effect in the Pod.
<details>
  <summary>Answer</summary>

```
# wait a little bit for the changes to take effect and then execute
kubectl exec web -- /bin/sh -c "ls /tmp/config && cat /tmp/config/log.properties"
```

</details>
<p>&nbsp;</p>
Delete the Pod and the ConfigMap.
<details>
  <summary>Answer</summary>

```
kubectl delete -f web.yml
kubectl delete cm log-properties
```

</details>
<p>&nbsp;</p>
Create the fruits ConfigMap with values favorite1=banana, favorite2=apple, favorite3=kiwi.
<details>
  <summary>Answer</summary>

```
kubectl create cm fruits --from-literal favorite1=banana --from-literal favorite2=apple --from-literal favorite3=kiwi
```

</details>
<p>&nbsp;</p>
Create the Deployment fruity with image httpd:alpine and 1 replica.
Add the contents of the fruits ConfigMap as environment variables and make sure the keys are uppercased in the Pod of the Deployment.
<details>
  <summary>Answer</summary>

```
kubectl create deploy fruity --image httpd:alpine --replicas=1 --dry-run=client -o yaml > fruity.yml
vim fruity.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: fruity
  name: fruity
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fruity
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: fruity
    spec:
      containers:
      - image: httpd:alpine
        name: httpd
        resources: {}
        env: # add this line
        - name: FAVORITE1 # add this line
          valueFrom: # add this line
            configMapKeyRef: # add this line
              name: fruits # add this line
              key: favorite1 # add this line
        - name: FAVORITE2 # add this line
          valueFrom: # add this line
            configMapKeyRef: # add this line
              name: fruits # add this line
              key: favorite2 # add this line
        - name: FAVORITE3 # add this line
          valueFrom: # add this line
            configMapKeyRef: # add this line
              name: fruits # add this line
              key: favorite3 # add this line
status: {}
kubectl create -f fruity.yml
```

</details>
<p>&nbsp;</p>
Check if the environment variables are present in the Pod of the Deployment.
<details>
  <summary>Answer</summary>

```
# your Pod will have different name because of the pod-template-hash
kubectl exec fruity-6d5f4484dc-5j7p9 -- env | grep FAVORITE
```

</details>
<p>&nbsp;</p>
Delete the Deployment and the ConfigMap.
<details>
  <summary>Answer</summary>

```
kubectl delete -f fruity.yml
kubectl delete cm fruits
```

</details>