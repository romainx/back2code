---
title: 'Docker Cookbook'
type: page
---

# Images

## Remove all images

```bash
# Remove all dangling images (tagged None)
$ docker rmi $(docker images -f "dangling=true" -q)

# Remove all
$ docker rmi $(docker images -q)

# ---- Remove images with multiple tags
# docker images first to get the image id
# $(docker images --filter=reference=spark-base --format "{{.ID}}")
$ docker images | grep <img_id> | awk '{print $1 ":" $2}' | xargs docker rmi

# To remove all images which are not referenced by any existing container
$ docker image prune -a
```

## List image layers

Detail of all the layers of an image.

```bash
$ docker history hello-world:latest
```

## Image size

```bash
# Get the size of a specific image
$ docker images --filter=reference='hello-world'

# Get the size of all the images.
$ docker system df -v
```

## Labels

https://blog.container-solutions.com/docker-inspect-template-magic

# Containers

## Get logs

```sh
$ docker logs $CONT_ID
```

## Running processes

Display the running processes of a container.

```bash
$ docker top $CONT_ID

# PID                 USER                TIME                COMMAND
# 50477               root                0:00                bash
```

## Stop all containers

```bash
# -a is for all containers default is runnning only, -q is for quiet
docker stop $(docker ps -a -q)
```

## Remove all containers

```bash
# Remove all containers
docker rm $(docker ps -a -q)
# all in one
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)
```

# Build

## Remove containers and force pull

```bash
# Build force pull
docker build --pull -t image:tag .
# Remove intermediate containers
docker build --force-rm -t image:tag .
```

Refs:
- https://stackoverflow.com/questions/45057282/jenkins-docker-how-to-force-pull-base-image-before-build
- https://www.databasesandlife.com/docker-build-pull-option/

## Debug

When a build fails, it's possible to run the last layer by using it's ID.

```sh
$ docker build -t echotest .

# Sending build context to Docker daemon 2.048 kB Step 0 : FROM busybox:latest
#     ---> 4986bf8c1536
#    Step 1 : RUN echo "This should work"
#     ---> Running in f63045cc086b
#    This should work
#     ---> 85b49a851fcc

$ docker run -it 85b49a851fcc
```

## Copy

```dockerfile
# will copy the contents of your local go directory in the /usr/local/ directory of your docker image.
ADD go /usr/local/

# To copy the go directory itself in /usr/local/ use:
ADD go /usr/local/go

# or
COPY go /usr/local/go
```

# Run

## Interactive mode

```bash
#docker run -it <image> /bin/bash
$ docker run -it busibox:latest /bin/bash
```

## Copy

```sh
# Get the container ID
$ docker ps
# To ->
$ docker cp foo.txt container_id:/foo.txt
# From <-
$ docker cp container_id:/foo.txt foo.txt
```

## Volume

```bash
docker run -v /host/directory:/container/directory
```

Refs:

- [Copying files from host to Docker container - Stack Overflow](https://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container)
    
# References

- [Top 10 Docker CLI commands you can’t live without – The Code Review – Medium](https://medium.com/the-code-review/top-10-docker-commands-you-cant-live-without-54fb6377f481)
- [docker cleanup guide: containers, images, volumes, networks · GitHub](https://gist.github.com/bastman/5b57ddb3c11942094f8d0a97d461b430)