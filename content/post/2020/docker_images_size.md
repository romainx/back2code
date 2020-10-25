---
title: 'Docker images size'
date: '2020-10-25'
categories: [ops]
tags: ['docker']
---

> How does one get the size of a Docker image **before** they pull it to their machine?

<!--more-->

## Introduction

The size given by the `docker images` command is **the uncompressed size of the image installed locally**, you can only get **from the registry the size of the compressed images** (for each architecture and if you want for each layer).

## Docker Hub

Get the **compressed size in bytes** of an image specific tag.

```bash
# for an official image the namespace is called library
curl -s https://hub.docker.com/v2/repositories/library/alpine/tags | \
    jq '.results[] | select(.name=="latest") | .full_size'

# 2796860

# here the project namespace is used
curl -s https://hub.docker.com/v2/repositories/jupyter/base-notebook/tags | \
    jq '.results[] | select(.name=="latest") | .full_size'

# 187647701
```

Get the compressed size in bytes of for an image tag for a specific architecture / os.

```bash
# selecting an architecture
curl -s https://hub.docker.com/v2/repositories/library/alpine/tags | \
    jq '.results[] | select(.name=="latest") | .images[] | select (.architecture=="amd64") | .size'

# 2796860

# selecting an architecture and a specific os
curl -s https://hub.docker.com/v2/repositories/library/hello-world/tags | \                                                                                         
    jq '.results[] | select(.name=="latest") | .images[] | select (.architecture=="amd64" and .os=="linux") | .size'

# 2529
```

## Alternative

An alternative is to use the experimental [`docker manifest inspect`](https://docs.docker.com/engine/reference/commandline/manifest_inspect/) command. The advantage is that it does not rely on Docker Hub, **it works as well with other registries** because it's based on the [Image Manifest specification](https://docs.docker.com/registry/spec/manifest-v2-2/).

```bash
# activate experimental mode
export DOCKER_CLI_EXPERIMENTAL=enabled 

docker manifest inspect -v alpine:latest | \
    jq '.[] | select(.Descriptor.platform.architecture=="amd64") | .SchemaV2Manifest.layers[].size'

# 2796860

# need to sum if multiple layers
docker manifest inspect jupyter/base-notebook:latest | \
    jq '[.layers[].size] | add'

# 187647701
```

## Notes

This post is a mix of two of my Stack Overflow answers 

- [Get the size of a Docker image before a pull?](https://stackoverflow.com/questions/33352901/get-the-size-of-a-docker-image-before-a-pull)
- [How to get total size of a docker image by docker API properly?](https://stackoverflow.com/questions/40377199/how-to-get-total-size-of-a-docker-image-by-docker-api-properly/)

## References / Further reading

- [Retrieving Docker Image Sizes](https://gist.github.com/MichaelSimons/fb588539dcefd9b5fdf45ba04c302db6)