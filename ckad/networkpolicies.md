Run these commands in order to generate Deployments and Services as well as Namespace to test Network Policies.
```
kubectl create deploy nginx --image=nginx:alpine
kubectl create deploy httpd --image=httpd:alpine
kubectl expose deploy nginx --port 80
kubectl expose deploy httpd --port 80
kubectl create ns testnetpol
```

Test that the Deployments are accessible through the Services.
<details>
  <summary>Answer</summary>

```
kubectl run tmp --restart=Never --image=nginx -i --rm -- curl --max-time 10 http://nginx:80
kubectl run tmp --restart=Never --image=nginx -i --rm -- curl --max-time 10 http://httpd:80
```
</details>
<p>&nbsp;</p>

Create a NetworkPolicy netpol1 that allows access to Pods with label app=nginx only from pods with label app=httpd and only on port 80.
<details>
  <summary>Answer</summary>

```
vim netpol1.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol1
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx # apply to Pods with this label
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: httpd # accept traffic from Pods with this label
      ports:
        - protocol: TCP
          port: 80 # and only on this port
```
</details>
<p>&nbsp;</p>

Test the access from a temporary Pod.
<details>
  <summary>Answer</summary>

```
kubectl run tmp --restart=Never --image=nginx -i --rm -- curl --max-time 10 http://nginx:80
```
</details>
<p>&nbsp;</p>

Test the access from a temporary Pod with label app=httpd.
<details>
  <summary>Answer</summary>

```
kubectl run tmp --labels=app=httpd --restart=Never --image=nginx -i --rm -- curl --max-time 10 http://nginx:80
# should work this time
```
</details>
<p>&nbsp;</p>

Create a NetworkPolicy netpol2 in default namespace that allows access to Pods with label app=httpd only from Pods with label run=tmp only on port 80 and from
Namespace testnetpol.
<details>
  <summary>Answer</summary>

```
vim netpol2.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol2
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: httpd # Pods to which policy applies
  policyTypes:
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: testnetpol # matches the name of the Namespace
          podSelector: # mind that there is no "-" , which means logical AND in the policy
            matchLabels:
              run: tmp # Pods from which to receive traffic
      ports:
        - protocol: TCP
          port: 80
kubectl create -f netpol2.yml
```
</details>
<p>&nbsp;</p>

Test that the httpd Pod is not accessible from default Namespace Pods.
<details>
  <summary>Answer</summary>

```
kubectl run tmp --restart=Never --image=nginx -i --rm -- curl --max-time 10 http://httpd:80
```
</details>
<p>&nbsp;</p>

Test from Pod from Namespace testnetpol with label run=tmp.
<details>
  <summary>Answer</summary>

```
kubectl run tmp -n testnetpol --restart=Never --image=nginx -i --rm -- curl --max-time 10 http://httpd.default:80
# Mind that we should specify the Namespace when we refer to Service from another Namespace
```
</details>
<p>&nbsp;</p>


Create a NetworkPolicy netpol3 for Pods with label httpd to only make outbound traffic on port 80 and to Pods with label app=nginx.
<details>
  <summary>Answer</summary>

```
vim netpol3.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol3
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: httpd
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: nginx
      ports:
        - protocol: TCP
          port: 80
kubectl create -f netpol3.yml
```
</details>
<p>&nbsp;</p>

Create a test Pod to test the egress policy
<details>
  <summary>Answer</summary>

```
kubectl run testdeploy --image=nginx:alpine
```
</details>
<p>&nbsp;</p>

Test the Network Policy running curl in the Pod with label httpd ( from the Deployment httpd).
<details>
  <summary>Answer</summary>

```
kubectl exec -it httpd-78f8b485db-j28b6 -- /bin/sh
wget -T 5 192.168.167.134 # the Pod ip of the nginx Deployment , should be successful
wget -T 5 192.168.167.183 # the IP of the testdeploy Pod, should fail
```
</details>
<p>&nbsp;</p>

Cleanup the resources.
```
kubectl delete ns testnetpol
kubectl delete -f netpol1.yml -f netpol2.yml  -f netpol3.yml
kubectl delete deploy,svc nginx httpd
kubectl delete pod testdeploy
```