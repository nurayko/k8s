Create the Secret credentials with data username=admin and password=admin123.
```
kubectl create secret generic credentials --from-literal username=admin --from-literal password=admin123
```

Print the username key of the credentials Secret.
```
kubectl get secret credentials -o jsonpath='{.data.username}' | base64 -d
```

Create the Pod with name secret-pod running the nginx:alpine image.
Mount the credentials Secret as a Volume under /tmp/ folder in the container.
```
kubectl run secret-pod --image nginx:alpine --dry-run=client -o yaml > secret-pod.yml
vim secret-pod.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
spec:
  volumes: # add this line
  - name: secret # add this line
    secret: # add this line
      secretName: credentials # add this line
  containers:
  - image: nginx:alpine
    name: secret-pod
    resources: {}
    volumeMounts: # add this line
    - name: secret # add this line
      mountPath: /tmp # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
kubectl create -f secret-pod.yml
```

Check if the Secret is mounted in the Pod.
```
kubectl exec secret-pod -- ls /tmp
```

Create the docker Secret of type docker-registry with registry internal-registry:5000, username service-user and password 21312cdsd32e+ .
```
kubectl create secret docker-registry docker \
--docker-server internal-registry:5000 \
--docker-username service-user \
--docker-password 21312cdsd32e+
```

Create the Secret db-creds from the following db.properties file:
db.url=postgres-svc
db.name=store
db.port=5432
db.user=store-user
db.pass=123123.dsadxxxx
Make sure that the key to the data in the Secret is application.properties .
```
kubectl create secret generic db-creds --from-file=application.properties=./db.properties
```

Create the frontend Deployment with image httpd:alpine and 1 replica.
Mount the db-creds Secret at /tmp/config in the container.
```
kubectl create deploy frontend --image httpd:alpine --replicas 1 --dry-run=client -o yaml > fe.yml
vim fe.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: frontend
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend
    spec:
      volumes: # add this line
      - name: db # add this line
        secret: # add this line
          secretName: db-creds # add this line
      containers:
      - image: httpd:alpine
        name: httpd
        resources: {}
        volumeMounts: # add this line
        - name: db # add this line
          mountPath: /tmp/config # add this line
status: {}
```

Check if the file Secret is mounted
```
kubectl exec frontend-57478f668c-lnm6p -- ls /tmp/config
```

Delete the Deployment and the Secret.
```
kubectl delete -f fe.yml
kubectl delete secret db-creds
```

Create the Secret animals with key-value pairs animal1=rabbit animal2=deer.
Run the pod animal with image nginx:alpine.
Add the Secret data in the environment variables, but make sure that animal2 key has value ANIMAL2 in the environment variables of the container.
```
kubectl create secret generic animals --from-literal animal1=rabbit --from-literal animal2=deer
kubectl run animal --image nginx:alpine --dry-run=client -o yaml > animal.yml
vim animal.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: animal
  name: animal
spec:
  containers:
  - image: nginx:alpine
    name: animal
    resources: {}
    env: # add this line
    - name: ANIMAL2 # add this line
      valueFrom: # add this line
        secretKeyRef: # add this line
          name: animals # add this line
          key: animal2 # add this line
    - name: animal1 # add this line
      valueFrom: # add this line
        secretKeyRef: # add this line
          name: animals # add this line
          key: animal1 # add this line
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
kubectl create -f animal.yml
```

Check if the values from the Secret are present as environment variables in the container.
```
kubectl exec animal -- env | grep -i 'ANIMAL[1/2]'
```

Delete the Pod and the Secret
```
kubectl delete -f animal.yml
kubectl delete secret animals
```
