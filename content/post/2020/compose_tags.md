---
title: 'Multiple Image Tags with Docker Compose'
date: '2020-01-26'
categories: ['ops']
tags: ['docker']
---

Recently I answered to [a question on Stack Overflow](https://stackoverflow.com/questions/47327979/how-to-use-multiple-image-tags-with-docker-compose):

> How to use multiple image tags with docker-compose?

The validated answer was based on `extends`, but it cannot by used anymore in Compose file format `3.x`. As suggested by a user, the [Extension fields](https://docs.docker.com/compose/compose-file/#extension-fields) capability added in the version `3.4` of Docker Compose can replace it to achieve the same goal: **reuse a single definition to set several tags**.

# Use YAML extension to define multiple tags

Here is how to use `YAML` extensions in this case.

```YAML
version: "3.4"
# Define common behavior
x-ubi-httpd:
  &default-ubi-httpd
  build: ubi-httpd
  # Other settings can also be shared
  image: ubi-httpd:latest

# Define one service by wanted tag
services:
  # Use the extension as is
  ubi-httpd:
    *default-ubi-httpd
  # Override the image tag
  ubi-httpd_major:
    << : *default-ubi-httpd
    image: ubi-httpd:1
  ubi-httpd_minor:
    << : *default-ubi-httpd
    image: ubi-httpd:1.0
  # Using an environment variable defined in a .env file for e.g.
  ubi-httpd_patch:
    << : *default-ubi-httpd
    image: "ubi-httpd:${UBI_HTTPD_PATCH}"
```

It uses an `.env` file to define environment variables.

```bash
$ cat .env
# ---- UBI HTTP ----- #
UBI_HTTPD_PATCH=1.0.1
```

# Build images

Images can be built now with all the defined tags.

```bash
$ docker-compose build
# ...

$ docker images | grep ubi-httpd
# ubi-httpd  1       8cc412411805  3 minutes ago  268MB
# ubi-httpd  1.0     8cc412411805  3 minutes ago  268MB
# ubi-httpd  1.0.1   8cc412411805  3 minutes ago  268MB
# ubi-httpd  latest  8cc412411805  3 minutes ago  268MB
```

Images could also be pushed in the same way.