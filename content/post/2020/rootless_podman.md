---
title: 'Rootless Podman'
date: '2020-02-09'
categories: [ops]
tags: ['docker']
---

> You might be saying to yourself, "but I have rootless containers right now with Docker - I run my docker commands as a regular user and it works all the time."  Even though you are executing the docker command line tool without root, the docker daemon is executing those requests as root on your behalf [...]
> 
> --- <cite>[A preview of running containers without root in RHEL 7.6][LK2]</cite> 

# Docker the root of all evil?

In fact it's exactly what is mentioned in the quote. The docker daemon itself runs as `root`. 

```bash
# The docker 
$ ps -eaf | grep "[d]ocker"

# root       817     1  0 06:28 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

One of the consequence is that any user has to belong to the `docker` group to be able to use launch a container.
Let's try to run a simple example that is running a container with a standard user on the host (`ubuntu`) and the `root` user in the container.

```bash
# Sleep forever
$ docker run -d --rm --name busy busybox sleep infinity

$ docker ps

# CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
# 3070e49e27bf        busybox             "sleep infinity"    About a minute ago   Up About a minute                       busy

# The process on the host is run by root
$ ps -eaf | grep "[s]leep"

# root      1984  1956  0 06:33 ?        00:00:00 sleep infinity
```

It confirms that the process on the host is run by the `root` user. So if there is any issue a `root` access can be obtained on the host, it's a big risk.

Another consequence is that any pulled image---and also any container---will reside in `/var/lib/docker` that is owned by `root` and shared among all users.

```bash
$ ls -alsh /var/lib/docker

# ls: cannot open directory '/var/lib/docker': Permission denied

$ sudo ls -alsh /var/lib/docker

# total 60K
# 4.0K drwx--x--x 14 root root 4.0K Feb  9 06:28 .
# 4.0K drwxr-xr-x 40 root root 4.0K Feb  8 15:16 ..
# 4.0K drwx------  2 root root 4.0K Feb  8 15:09 builder
# 4.0K drwx------  4 root root 4.0K Feb  8 15:09 buildkit
# 4.0K drwx------  6 root root 4.0K Feb  9 06:41 containers
# 4.0K drwx------  3 root root 4.0K Feb  8 15:09 image
# 4.0K drwxr-x---  3 root root 4.0K Feb  8 15:09 network
# 4.0K -rwxr-xr-x  1 root root 1.5K Feb  8 15:12 nuke-graph-directory.sh
# 4.0K drwx------ 12 root root 4.0K Feb  9 06:41 overlay2
# [...]
```

# Podman to the rescue!

> What is Podman? Podman is a daemonless container engine for developing, managing, and running OCI Containers on your Linux System. Containers can either be run as root or in rootless mode. Simply put: `alias docker=podman`. More details [here](https://podman.io/whatis.html). 
> --- <cite>https://podman.io</cite>

There is several interesting things there:

- **daemonless** means that there is no need to have a daemon running on the host
- **rootless mode** means that there is no need to be `root` to start a container
- **alias of docker** means that podman mimics the `docker` command, so in theory any `docker` command can be used with `podman`

Let's try to do the same thing with `podman`.


```bash
$ podman run -d --rm --name busy busybox sleep infinity

$ podman ps

# CONTAINER ID  IMAGE                             COMMAND         CREATED         STATUS             PORTS  NAMES
# fe7254c0c247  docker.io/library/busybox:latest  sleep infinity  25 seconds ago  Up 24 seconds ago         busy

# The process on the host is run by ubuntu (the unpriviledged user)
$ ps -eaf | grep "[s]leep"

# ubuntu   16626 16614  0 07:12 ?        00:00:00 sleep infinity
```

It's also possible to check through this command what are the `user / group` used on the host (`huser / hgroup`) and in the container (`user / group`).

```bash
$ podman top -l user group huser hgroup

# USER   GROUP   HUSER   HGROUP
# root   root    1000    1000

# ID 10000 correspond to the ubuntu user
$ id ubuntu

# uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),108(lxd),114(netdev),115(docker)

# We can see the difference between docker and podman if we run both at the same time
$ ps -eaf | grep "[s]leep"

# ubuntu    4299  4288  0 15:04 ?        00:00:00 sleep infinity
# root     10694 10669  0 15:18 ?        00:00:00 sleep infinity
```

So far so good and it goes the same with the images---and the other stuffs---that are stored by user and not shared with everyone and owned by `root`.

```bash
# Checking where the podman store is located
$ podman info | tail

#  ContainerStore:
#    number: 3
#  GraphDriverName: vfs
#  GraphOptions: {}
#  GraphRoot: /home/ubuntu/.local/share/containers/storage
#  GraphStatus: {}
#  ImageStore:
#    number: 1
#  RunRoot: /run/user/1000/containers
#  VolumePath: /home/ubuntu/.local/share/containers/storage/volumes

$ ls -alsh /home/ubuntu/.local/share/containers/storage

# total 40K
# 4.0K drwx------ 9 ubuntu ubuntu 4.0K Feb  8 14:47 .
# 4.0K drwx------ 4 ubuntu ubuntu 4.0K Feb  8 14:48 ..
# 4.0K drwx------ 2 ubuntu ubuntu 4.0K Feb  8 14:47 libpod
# 4.0K drwx------ 2 ubuntu ubuntu 4.0K Feb  8 14:47 mounts
# 4.0K -rw------- 1 ubuntu ubuntu   64 Feb  9 07:12 storage.lock
# 4.0K drwx------ 2 ubuntu ubuntu 4.0K Feb  8 14:47 tmp
# 4.0K drwx------ 3 ubuntu ubuntu 4.0K Feb  8 14:48 vfs
# 4.0K drwx------ 5 ubuntu ubuntu 4.0K Feb  9 07:12 vfs-containers
# 4.0K drwx------ 3 ubuntu ubuntu 4.0K Feb  8 14:48 vfs-images
# 4.0K drwx------ 2 ubuntu ubuntu 4.0K Feb  9 07:12 vfs-layers
````

PS: The grep trick `grep "[s]leep"` comes from this [question/answer][LK1].

[LK1]: https://unix.stackexchange.com/questions/74185/how-can-i-prevent-grep-from-showing-up-in-ps-results
[LK2]: https://www.redhat.com/en/blog/preview-running-containers-without-root-rhel-76