---
title: Setup a local registry for Kubernetes
date: '2018-12-26'
tags: [ops]
---

For **testing purpose**  I'm using the Kubernetes cluster shipped with the Docker Desktop client (`docker-for-desktop`).
I want to avoid to go through [Docker Hub][lk-1] each time I want to build an image to run it in my Kubernetes local cluster.
To solve this problem I've setup a local registry that runs in the docker on my machine. I can push images to this local registry that are directly available from the Kubernetes cluster.

# Declare the insecure registry

To be able to use a local non secure registry you have first to declare it.

```bash
# Getting your IP on a mac
$ MY_IP="$(ipconfig getifaddr en0)"
$ echo $MY_IP
xxx.x
```
You have to declare the `MY_IP:5000` as an insecure registry in Docker client and apply and restart to  change.

![](/post/2018/docker-local-registry_files/docker-registry.png)

# Start the registry

The registry documentation is available [here][lk-2].

```bash
# Start the registry
$ docker run -d -p 5000:5000 --name registry registry:2
# Test
$ curl -i $MY_IP:5000

HTTP/1.1 200 OK
```

# Push an image in the registry

To be able to use it from Kubernetes you have first to push an image to it. For the example I'm reusing the image `luksa/kubia` from the great book *Kubernetes in Action*[^1].

```bash
# Tag the image with the registry
$ docker image tag luksa/kubia $MY_IP:5000/kubia
# Push
$ docker push $MY_IP:5000/kubia
# List the content
$ curl -X GET http://$MY_IP:5000/v2/_catalog

{"repositories":["kubia"]}
```

# Use the image from the Kubernetes cluster

Now that the image is available in the local registry, I can use it from the Kubernetes cluster.

```bash
# Run the image
$ k run kubia --image=$MY_IP:5000/kubia --port=8080 --generator=run/v1
# Check what's going on
$ k describe pod
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  Scheduled              44s   default-scheduler            Successfully assigned kubia-brwcb to docker-for-desktop
  Normal  SuccessfulMountVolume  43s   kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-t5n42"
  Normal  Pulling                43s   kubelet, docker-for-desktop  pulling image "xxx.x:5000/kubia"
  Normal  Pulled                 43s   kubelet, docker-for-desktop  Successfully pulled image "xxx.x:5000/kubia"
  Normal  Created                42s   kubelet, docker-for-desktop  Created container
  Normal  Started                42s   kubelet, docker-for-desktop  Started container

# Testing if it works by exposing a http service
$ k expose rc kubia --type=LoadBalancer --name kubia-http
service "kubia-http" exposed
$ curl localhost:8080
You\`ve hit kubia-brwcb

# Cleaning things up
$ k delete rc kubia
replicationcontroller "kubia" deleted
$ k delete svc kubia-http
service "kubia-http" deleted
```

When you're done, you can simply stop the local registry.

```bash
# How to clean things up at the end
$ docker container stop registry && docker container rm -v registry
```
[lk-1]: https://hub.docker.com/
[lk-2]: https://docs.docker.com/registry/#basic-commands

[^1]: Marko Luksa, *[Kubernetes in Action](https://www.goodreads.com/book/show/34013922-kubernetes-in-action)*, (Manning, 2017)