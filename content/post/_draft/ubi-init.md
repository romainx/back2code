---
title: 'First Look at the UBI Init Base Images'
date: '2020-01-19'
categories: [ops]
tags: ['docker']
---

# Introduction

Among its catalog of **Universal Base Images** (UBI), **Red Hat** provides a kind / flavor prefixed `init`.

> The UBI `ubi8-init` images contains the `systemd` initialization system, making them useful for building images in which you want to run `systemd` services, such as a web server or file server. [...]  
> In `ubi8-init`, the `Cmd` is set to `/sbin/init`, instead of `bash`, to start the `systemd` Init service by default. It includes `ps` and `process` related commands (`procps-ng` package), which `ubi8` does not. [...]  
> Also, `ubi8-init` sets `SIGRTMIN+3` as the `StopSignal`, as `systemd` in `ubi8-init` ignores normal signals to exit (`SIGTERM` and `SIGKILL`), but will terminate if it receives `SIGRTMIN+3`. 
>
> --- <cite>[Using Init Red Hat base images][LK-1]</cite>

The point seems to be---quick disclaimer: I'm not a specialist at all of this topic, just curious---that different communities have different opinions:

**Upstream docker**

> Upstream docker says any process can run as `PID 1` in a container.
> And they have proven this by the thousands of docker-formatted container images that are present on their container image registry.

**Systemd developers**

> The systemd developers believe the opposite.
> They say you should always run an init system as `PID 1` in any environment.  
> They state that `PID 1` provides services to the processes inside the container that are part of the Linux API. [...]  
> People building docker-formatted images have to build their own Init command for launching the container.  They can’t simply use the systemd unit file just the way that the OS and packager intends.  This is also part of the Linux Service API.
>
> --- <cite>[Running systemd in a non-privileged container][LK-2]</cite>

# First try

```bash
$ docker run --name ubi-init -d --rm registry.access.redhat.com/ubi8/ubi-init

# Init is running as the PID 1
$ docker exec -it ubi-init ps -eaf

# UID        PID  PPID  C STIME TTY          TIME CMD
# root         1     0  0 08:09 ?        00:00:00 /sbin/init

# However it does not work
$ docker exec -it ubi-init systemctl --type=service

# System has not been booted with systemd as init system (PID 1). Can't operate.
# Failed to connect to bus: Host is down
```

# Privileged mode

To run `systemd` expects a certain number of things, see [Running systemd in a non-privileged container][LK-2] for the detail. A simple way to make it work is to run docker in **privileged** mode.

> When the operator executes docker run --privileged, Docker will enable access to all devices on the host as well as set some configuration in AppArmor or SELinux to allow the container nearly all the same access to the host as processes running outside containers on the host.
> --- <cite>[Docker run reference][LK-3]</cite>

So let's try by installing and running an Apache HTTP server.

```bash
# Run and expose the port 80
docker run --privileged --name ubi-init -p 80:80 -d --rm registry.access.redhat.com/ubi8/ubi-init
# Installation
docker exec ubi-init dnf install -y httpd
# Start
docker exec ubi-init systemctl start httpd
# Checking if it runs
docker exec ubi-init systemctl status httpd

# ● httpd.service - The Apache HTTP Server
#    Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
#    Active: active (running) since Sun 2020-01-19 10:42:52 UTC; 5s ago
```

Now the Apache start page should be displayed in your preferred web browser at [http://localhost](http://localhost).

# From a docker image

By defining a simple docker image for that the advantage in terms of simplicity and standardisation are obvious.

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi-init

# Install httpd
RUN dnf -y install httpd && \
    dnf clean all && \
    # Tell systemd to start httpd
    systemctl enable httpd

# Expose its default port
EXPOSE 80
```

```bash
# Build the image
$ docker build --rm --pull -t httpd-init .
# Run it
$ docker run --privileged --name httpd-init -d --rm -p 80:80 httpd-init
```

# Quick exploration

We can check that the logs are written in the journal.

```bash
# Check that logs are written in the journal
$ docker exec httpd-init journalctl -u httpd

# Jan 19 10:46:38 96829b67cf16 systemd[1]: Starting The Apache HTTP Server...
# Jan 19 10:46:38 96829b67cf16 httpd[28]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress # this message
# Jan 19 10:46:38 96829b67cf16 systemd[1]: Started The Apache HTTP Server.
# Jan 19 10:46:38 96829b67cf16 httpd[28]: Server configured, listening on: port 80

# But nothing in docker logs ...
$ docker logs httpd-init
```

What is the problem?

> The main one is that `systemd`/`journald` controls the output of containers, whereas tools like Kubernetes and OpenShift expect the containers to log directly to `stdout` and `stderr`. So, if you are going to manage your containers via Orchestrator like these, then you should think twice about using systemd-based containers. Additionally, the upstream community of Docker and Moby were often hostile to the use of `systemd` in a container.
> 
> --- <cite>[How to run systemd in a container][LK-4]</cite>

# Next

To go further it could be interesting to have a look at [Podman](https://podman.io/).  
[This article][LK-4] explain how and the advantages of using this setup.
I may give it a try in a next article...

[LK-1]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/using_red_hat_universal_base_images_standard_minimal_and_runtimes
[LK-2]: https://developers.redhat.com/blog/2016/09/13/running-systemd-in-a-non-privileged-container/
[LK-3]: https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities
[LK-4]: https://developers.redhat.com/blog/2019/04/24/how-to-run-systemd-in-a-container/