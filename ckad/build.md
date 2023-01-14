Create a Dockerfile that has the base image of alpine and runs the command "echo I have to get some sleep && sleep 8h"
```
vim Dockerfile
FROM alpine
CMD /bin/sh -c "echo I have to get some sleep && sleep 8h"
```

Build a container image using the Dockerfile and name the image sleeper.
```
podman image build -t sleeper -f Dockerfile .
# or
docker image build -t sleeper -f Dockerfile .
```

Retag the image to sleeper:v1
```
podman tag sleeper sleeper:v1
# or
docker tag sleeper sleeper:v1
```

List all available images
```
podman images
# or
docker images
```

Remove the sleeper:latest image from the local registry
```
podman rmi sleeper:latest
# or
docker rmi sleeper:latest
```

Run the sleeper:v1 image in detached mode with name sleepy.
```
podman run --name sleepy -d sleeper:v1
# or
docker run --name sleepy -d sleeper:v1
```

Redirect the logs of the sleepy container to sleepy.log
```
podman logs sleepy > sleepy.log
# or
docker logs sleepy > sleepy.log
```

List all running containers and redirect the output of the command to container.txt
```
podman ps > containers.txt
# or
docker ps > containers.txt
```

Remove the sleepy container. Make sure to use the -f option.
```
podman rm -f sleepy
# or
docker rm -f sleepy
```

Delete the sleeper:v1 image.
```
podman rmi sleeper:v1
# or
docker rmi sleeper:v1
```

Download the nginx:alpine image from Docker Hub.
```
podman pull docker.io/library/nginx:alpine
# or
docker pull nginx:alpine
```

Run the nginx container using the nginx:alpine image mapping port 8081 on the host to port 80 in the container.
Make sure the container runs in detached mode.
```
podman run --name nginx -d -p 8081:80 nginx:alpine
# or
docker run --name nginx -d -p 8081:80 nginx:alpine

Test that the container returns the expected html page
```
curl http://localhost:8081
```

Delete the container.
```
podman rm -f nginx
# or
docker rm -f nginx
```

Create a file index.html with contents "This is my page." . Ignore the fact that it is not valid html.
```
echo "This is my page." > index.html
```

Create a Dockerfile that copies the index.html file to /usr/share/nginx/html folder in the container filesystem.
```
vim Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html
```

Build the nginx:alpine-modified image using the new Dockerfile.
```
podman build -t nginx:alpine-modified -f Dockerfile .
# or
docker image build -t nginx:alpine-modified -f Dockerfile .
```

Run the nginx:alpine-modified image in detached mode with name nginx-modified and mapping port 8082 of the host to port 80 on the container.
```
podman run --name nginx-modified -d -p 8082:80 nginx:alpine-modified
# or

```

Test that the page returns the expected result.
```
curl http://localhost:8082
```

Retag the image so that it is ready to be pushed to registry my-registry-url.com and repository nginx-images.
```
podman tag nginx:alpine-modified my-registry-url.com/nginx-images/nginx:alpine-modified
# or
docker tag nginx:alpine-modified my-registry-url.com/nginx-images/nginx:alpine-modified
```

Push the image to the registry ( the command will fail )
```
podman push my-registry-url.com/nginx-images/nginx:alpine-modified
# or
docker push my-registry-url.com/nginx-images/nginx:alpine-modified
```
