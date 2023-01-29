Expose webserver Pod running on port 80 via NodePort Service. The nodePort should be 31555, the Service name should be webserver-svc.
<details>
  <summary>Answer</summary>

```
kubectl expose pod webserver --name=webserver-svc --type=NodePort --target-port=80 --dry-run=client -o yaml > ws.yml
vim ws.yml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: webserver
  name: webserver-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 31555 # add this line
  selector:
    run: webserver
  type: NodePort
status:
  loadBalancer: {}
kubectl create -f ws.yml
```
</details>
<p>&nbsp;</p>

Edit the type of the webserver-svc Service to ClusterIP. Use the kubectl patch command.
<details>
  <summary>Answer</summary>

```
kubectl patch svc webserver-svc -p '{"spec":{"type":"ClusterIP"}}'
```
</details>
<p>&nbsp;</p>

Expose the frontend Deployment internally. The Service should be named frontend-svc, serve on port 8080 and should target port 80 of the Pods of the Deployment.
<details>
  <summary>Answer</summary>

```
kubectl expose deploy frontend --name=frontend-svc --type=ClusterIP --port=8080 --target-port=80
```
</details>
<p>&nbsp;</p>

List the addresses that the Service frontend-svc load balances.
<details>
  <summary>Answer</summary>

```
kubectl get ep frontend-svc -o jsonpath='{.subsets[*].addresses[*].ip}'
```
</details>
<p>&nbsp;</p>

Create a Service that exposes external IP. It should select pods with label app=frontend that serve traffic on port 80.
The Service should serve on port 8082.
<details>
  <summary>Answer</summary>

```
kubectl create svc loadbalancer frontend --tcp=8082:80
```
</details>
<p>&nbsp;</p>

Test that the service returns correct response using curl from a temporary Pod.
<details>
  <summary>Answer</summary>

```
kubectl run tmp --restart=Never --rm -i --image=nginx -- curl http://frontend:8082
# you can do this cross-namespace, default in this case
kubectl run tmp --restart=Never --rm -i --image=nginx -- curl http://frontend.default:8082
# or you can specify the ip address
kubectl run tmp --restart=Never --rm -i --image=nginx -- curl http://10.99.170.32:8082
```
</details>
<p>&nbsp;</p>