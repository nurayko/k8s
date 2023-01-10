Create a new ServiceAccount called secretviewer.
```
kubectl create serviceaccount secretviewer
```

Create a new Role called viewsecrets that allows performing get and list on secrets.
```
kubectl create role viewsecrets --verb=get,list --resource=secret
```

Create a new RoleBinding secretviewer that binds the Role viewsecrets to the ServiceAccount secretviewer.
```
kubectl create rolebinding secretview --role=viewsecrets --serviceaccount=default:secretviewer
```

Test the permission to get and list secrets of the secretviewer ServiceAccount.
```
kubectl auth can-i get secrets --as system:serviceaccount:default:secretviewer
yes # output
kubectl auth can-i list secrets --as system:serviceaccount:default:secretviewer
yes # output
```

Create a Pod secretviewer that uses the ServiceAccount secretviewer that runs the nginx:alpine image.
```
kubectl run secretviewer --image=nginx:alpine --dry-run=client -o yaml > secretviewer.yml
vim secretviewer.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secretviewer
  name: secretviewer
spec:
  serviceAccountName: secretviewer # add this line
  containers:
  - image: nginx:alpine
    name: secretviewer
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
kubectl create -f secretviewer.yml
```

Create a token for the secretviewer ServiceAccount.
```
kubectl create token secretviewer
```

Create a secret secretviewer-secret with the token for the secretviewer ServiceAccount.
```
kubectl create secret generic secretviewer-secret --type="kubernetes.io/service-account-token" --dry-run=client -o yaml > secret.yml
vim secret.yml
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: null
  name: secretviewer-secret
  annotations: # add this line
    kubernetes.io/service-account.name: secretviewer # add this line
type: kubernetes.io/service-account-token
kubectl create -f secret.yml
```

Get the token from the secretviewer-secret Secret.
```
kubectl get secret secretviewer-secret -o jsonpath={.data.token} | base64 -d
```

Create a Role viewer in the dev namespace that can get, watch and list every resource.
```
kubectl create role viewer -n dev --verb=get,watch,list --resource="*"
```

Create a RoleBinding viewer-john which binds the viewer Role to user with name john in the dev namespace.
```
kubectl create rolebinding viewer-john -n dev --role=viewer --user=john
```
