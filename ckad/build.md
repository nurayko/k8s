Create a Dockerfile that has the base image of alpine and runs the command "echo I have to get some sleep && sleep 8h" .
<details>
  <summary>Answer</summary>

```
vim Dockerfile
FROM alpine
CMD ["/bin/sh","-c","echo I have to get some sleep && sleep 8h"]
```

</details>
<p>&nbsp;</p>
Build a container image using the Dockerfile and name the image sleeper.
<details>
  <summary>Answer</summary>

```
podman image build -t sleeper -f Dockerfile .
# or
docker image build -t sleeper -f Dockerfile .
```

</details>
<p>&nbsp;</p>
Retag the image to sleeper:v1 .
<details>
  <summary>Answer</summary>

```
podman tag sleeper sleeper:v1
# or
docker tag sleeper sleeper:v1
```

</details>
<p>&nbsp;</p>
List all available images.
<details>
  <summary>Answer</summary>

```
podman images
# or
docker images
```

</details>
<p>&nbsp;</p>
Remove the sleeper:latest image from the local registry.
<details>
  <summary>Answer</summary>

```
podman rmi sleeper:latest
# or
docker rmi sleeper:latest
```

</details>
<p>&nbsp;</p>
Run the sleeper:v1 image in detached mode with name sleepy.
<details>
  <summary>Answer</summary>

```
podman run --name sleepy -d sleeper:v1
# or
docker run --name sleepy -d sleeper:v1
```

</details>
<p>&nbsp;</p>
Redirect the logs of the sleepy container to sleepy.log .
<details>
  <summary>Answer</summary>

```
podman logs sleepy > sleepy.log
# or
docker logs sleepy > sleepy.log
```

</details>
<p>&nbsp;</p>
List all running containers and redirect the output of the command to container.txt .
<details>
  <summary>Answer</summary>

```
podman ps > containers.txt
# or
docker ps > containers.txt
```

</details>
<p>&nbsp;</p>
Remove the sleepy container. Make sure to use the -f option.
<details>
  <summary>Answer</summary>

```
podman rm -f sleepy
# or
docker rm -f sleepy
```

</details>
<p>&nbsp;</p>
Delete the sleeper:v1 image.
<details>
  <summary>Answer</summary>

```
podman rmi sleeper:v1
# or
docker rmi sleeper:v1
```

</details>
<p>&nbsp;</p>
Download the nginx:alpine image from Docker Hub.

<details>
  <summary>Answer</summary>
```
podman pull docker.io/library/nginx:alpine
# or
docker pull nginx:alpine
```

</details>
<p>&nbsp;</p>
Run the nginx container using the nginx:alpine image mapping port 8081 on the host to port 80 in the container.
Make sure the container runs in detached mode.
<details>
  <summary>Answer</summary>

```
podman run --name nginx -d -p 8081:80 nginx:alpine
# or
docker run --name nginx -d -p 8081:80 nginx:alpine
```

</details>
<p>&nbsp;</p>
Test that the container returns the expected html page.
<details>
  <summary>Answer</summary>

```
curl http://localhost:8081
```

</details>
<p>&nbsp;</p>
Delete the container.
<details>
  <summary>Answer</summary>

```
podman rm -f nginx
# or
docker rm -f nginx
```

</details>
<p>&nbsp;</p>
Create a file index.html with contents "This is my page." . Ignore the fact that it is not valid html.
<details>
  <summary>Answer</summary>

```
echo "This is my page." > index.html
```

</details>
<p>&nbsp;</p>
Create a Dockerfile that copies the index.html file to /usr/share/nginx/html folder in the container filesystem.
<details>
  <summary>Answer</summary>

```
vim Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html
```

</details>
<p>&nbsp;</p>
Build the nginx:alpine-modified image using the new Dockerfile.

<details>
  <summary>Answer</summary>

```
podman build -t nginx:alpine-modified -f Dockerfile .
# or
docker image build -t nginx:alpine-modified -f Dockerfile .
```

</details>
<p>&nbsp;</p>
Run the nginx:alpine-modified image in detached mode with name nginx-modified and mapping port 8082 of the host to port 80 on the container.
<details>
  <summary>Answer</summary>

```
podman run --name nginx-modified -d -p 8082:80 nginx:alpine-modified
# or
docker run --name nginx-modified -d -p 8082:80 nginx:alpine-modified
```

</details>
<p>&nbsp;</p>
Test that the page returns the expected result.
<details>
  <summary>Answer</summary>

```
curl http://localhost:8082
```

</details>
<p>&nbsp;</p>
Retag the image so that it is ready to be pushed to registry my-registry-url.com and repository nginx-images.
<details>
  <summary>Answer</summary>

```
podman tag nginx:alpine-modified my-registry-url.com/nginx-images/nginx:alpine-modified
# or
docker tag nginx:alpine-modified my-registry-url.com/nginx-images/nginx:alpine-modified
```

</details>
<p>&nbsp;</p>
Push the image to the registry ( the command will fail ).
<details>
  <summary>Answer</summary>

```
podman push my-registry-url.com/nginx-images/nginx:alpine-modified
# or
docker push my-registry-url.com/nginx-images/nginx:alpine-modified
```

</details>