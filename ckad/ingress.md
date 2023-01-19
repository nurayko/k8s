Run these commands in order to generate Deployments and Services.
```
kubectl create deploy nginx --image=nginx:alpine
kubectl create deploy httpd --image=httpd:alpine
kubectl expose deploy nginx --port 80
kubectl expose deploy httpd --port 80
```

Display all Ingress classes in the cluster
```
kubectl get ingressclass
# You should have an Ingress class and Ingress controller so that the Ingress works.
```

Create an Ingress ingress-web that routes traffic to host web.nginx.com to the nginx Service on port 80.
Traffic to host web.httpd.com should be redirected to httpd Service on port 80. Use the nginx Ingress class name.
```
vim ingress-web.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-web
spec:
  ingressClassName: nginx # the ingress class that is installed in the Cluster
  rules:
  - host: "web.nginx.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx
            port:
              number: 80
  - host: "web.httpd.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: httpd
            port:
              number: 80
kubectl create -f ingress-web.yml
```

Describe the ingress and check if it routes traffic to services.
```
kubectl describe ingress ingress-web
...
Rules:
  Host           Path  Backends
  ----           ----  --------
  web.nginx.com
                 /   nginx:80 (192.168.167.188:80)
  web.httpd.com
                 /   httpd:80 (192.168.167.178:80)
...
```

Test that the ingress works by running the curl command.
The command should be executed with the address of the LoadBalancer Service of the ingress-nginx-controller specifying the Host with a Header.
```
curl --max-time 5 -H "Host: web.nginx.com" http://10.111.248.248
# Test to check if the traffic is route to the nginx Service
...
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
...
curl --max-time 5 -H "Host: web.httpd.com" http://10.111.248.248
# Test to check if the traffic is route to the httpd Service
<html><body><h1>It works!</h1></body></html>
```

Edit the ingress and add the "nginx.ingress.kubernetes.io/rewrite-target" annotation with value "/".
```
kubectl annotate ingress ingress-web nginx.ingress.kubernetes.io/rewrite-target=/
```

Test again if the ingress is routing traffic properly.
```
curl --max-time 5 -H "Host: web.nginx.com" http://10.111.248.248
curl --max-time 5 -H "Host: web.httpd.com" http://10.111.248.248
```

Delete all resources created.
```
kubectl delete -f ingress-web.yml
kubectl delete svc,deploy httpd nginx
```