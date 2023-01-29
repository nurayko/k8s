Create a new ServiceAccount called secretviewer.
<details>
  <summary>Answer</summary>

```
kubectl create serviceaccount secretviewer
```
</details>
<p>&nbsp;</p>

Create a new Role called viewsecrets that allows performing get and list on secrets.
<details>
  <summary>Answer</summary>

```
kubectl create role viewsecrets --verb=get,list --resource=secret
```
</details>
<p>&nbsp;</p>

Create a new RoleBinding secretviewer that binds the Role viewsecrets to the ServiceAccount secretviewer.
<details>
  <summary>Answer</summary>

```
kubectl create rolebinding secretview --role=viewsecrets --serviceaccount=default:secretviewer
```
</details>
<p>&nbsp;</p>

Test the permission to get and list secrets of the secretviewer ServiceAccount.
<details>
  <summary>Answer</summary>

```
kubectl auth can-i get secrets --as system:serviceaccount:default:secretviewer
yes # output
kubectl auth can-i list secrets --as system:serviceaccount:default:secretviewer
yes # output
```
</details>
<p>&nbsp;</p>

Create a Pod secretviewer that uses the ServiceAccount secretviewer that runs the nginx:alpine image.
<details>
  <summary>Answer</summary>

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
</details>
<p>&nbsp;</p>

Create a token for the secretviewer ServiceAccount.
<details>
  <summary>Answer</summary>

```
kubectl create token secretviewer
```
</details>
<p>&nbsp;</p>

Create a secret secretviewer-secret with the token for the secretviewer ServiceAccount.
<details>
  <summary>Answer</summary>

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
</details>
<p>&nbsp;</p>

Get the token from the secretviewer-secret Secret.
<details>
  <summary>Answer</summary>

```
kubectl get secret secretviewer-secret -o jsonpath={.data.token} | base64 -d
```
</details>
<p>&nbsp;</p>

Create a Role viewer in the dev namespace that can get, watch and list every resource.
<details>
  <summary>Answer</summary>

```
kubectl create role viewer -n dev --verb=get,watch,list --resource="*"
```
</details>
<p>&nbsp;</p>

Create a RoleBinding viewer-john which binds the viewer Role to user with name john in the dev namespace.
<details>
  <summary>Answer</summary>

```
kubectl create rolebinding viewer-john -n dev --role=viewer --user=john
```
</details>
<p>&nbsp;</p>