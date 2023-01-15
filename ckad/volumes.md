Create the PersistentVolume my-pv with access mode that enables it to be mounted on many nodes for read operation.
It should have 10Gi capacity and hostPath on /tmp/data. The reclaim policy should be set to Retain.
```
vim my-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadOnlyMany
  hostPath:
    path: /tmp/data
  persistentVolumeReclaimPolicy: Retain
kubectl create -f my-pv.yml
```

Create a PersistentVolumeClaim my-pvc with read only many access mode. The pvc should request 5Gi storage.
Bind it to the my-pv PersistentVolume.
```
vim my-pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  volumeName: my-pv
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 5Gi
kubectl create -f my-pvc.yml
```

Make sure that my-pvc is bound to my-pv
```
kubectl get pvc my-pvc
```

Populate /tmp/data directory with files file1, file2 and file3.
```
mkdir -p /tmp/data && touch /tmp/data/file1 /tmp/data/file2 /tmp/data/file3
```

Create a Deployment with name web that uses the httpd:alpine image has 2 replicas.
Mount the my-pvc PersistentVolumeClaim on /tmp/data in the containers.
```
kubectl create deploy web --image httpd:alpine --replicas=2 --dry-run=client -o yaml > web.yml
vim web.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      volumes: # add this line
      - name: data # add this line
        persistentVolumeClaim: # add this line
          claimName: my-pvc # add this line
      containers:
      - image: httpd:alpine
        name: httpd
        resources: {}
        volumeMounts: # add this line
        - name: data # add this line
          mountPath: /tmp/data # add this line
status: {}
kubectl create -f web.yml
```

Check if the files are present in the Pods.
```
# your Pods will have different names due to pod-template-hash and ReplicaSet
kubectl exec web-7d5845694-dnm4r -- ls /tmp/data
kubectl exec web-7d5845694-547q4 -- ls /tmp/data
```

Delete the Deployment, PersistentVolume and PersistentVolumeClaim.
```
kubectl delete deploy web
kubectl delete -f my-pvc.yml -f my-pv.yml
```

Create the StorageClass my-sc with provisioner custom-provisioner , Retain reclaim policy, volume expansion should be disabled.
The binding mode should be set to WaitForFirstConsumer.
```
vim my-sc.yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-sc
provisioner: custom-provisioner
reclaimPolicy: Retain
allowVolumeExpansion: false
volumeBindingMode: WaitForFirstConsumer
kubectl create -f my-sc.yml
```

Create a PersistentVolumeClaim my-pvc-with-sc with storage class my-sc . The pvc should request 500Mi of storage and should have
ReadWriteOnce access mode.
```
vim my-pvc-with-sc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-with-sc
spec:
  storageClassName: my-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
kubectl create -f vim my-pvc-with-sc.yml
```

Check the status of the newly created pvc.
```
kubectl get pvc my-pvc-with-sc
```

Delete the PersistentVolumeClaim and StorageClass.
```
kubectl delete -f my-pvc-with-sc.yml  -f my-sc.yml
```