---
title: 'Zombie Processes Back in K8S'
date: '2020-02-06'
categories: [ops]
tags: ['docker', 'kubernetes']
---

After my [previous article][LK2] on zombie processes I was curious to see if and how they can affect containers running in a Kubernetes (K8S) cluster.

<!--more-->

## First try

A simple descriptor to deploy the container.

```yaml
# zombie-init.yml
apiVersion: v1
kind: Pod
metadata:
  name: zombie-init
spec:
  containers:
  - name: zombie-init
    image: zombie-init:latest
    imagePullPolicy: Never
```

```bash
$ k create -f zombie-init.yml

# pod/zombie-init created

$ k logs zombie-init         

# The zombie pid will be: 7
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.5  13164 10676 ?        Ss   20:09   0:00 python3 /root/test.py
# root         7  0.0  0.0      0     0 ?        Z    20:09   0:00 [python3] <defunct>
# root         8  0.0  0.1   9392  2952 ?        R    20:09   0:00 ps xawuf
```

We can observe the same behavior a already observed in the previous article. A zombie process (`defunct`) has not been reaped since the `PID 1` process (it's parent) has not played this role.

## Reaping zombies

To benefit from a proper `init` process that will take care of reaping zombies, we have to tell Kubernetes to enable **PID sharing between the containers that are running in the POD**. This is done by specifying `shareProcessNamespace: true`. So the deployment looks now like this.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: zombie-init
spec:
  shareProcessNamespace: true
  containers:
  - name: zombie-init
    image: zombie-init:latest
    imagePullPolicy: Never
```

Here is the result obtained with this new configuration.

```bash
$ k logs zombie-init

# The zombie pid will be: 21
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root        15  0.0  0.5  13156 10596 ?        Ss   20:11   0:00 python3 /root/test.py
# root        22  0.0  0.1   9392  3080 ?        R    20:11   0:00  \_ ps xawuf
# root         1  1.0  0.0   1024     4 ?        Ss   20:11   0:00 /pause
```

In the log, we can see that the zombie process has been reaped without specifying an `init` process or adding one in the container. In this case the `init` process, which plays the parent role, is the `pause` process (in fact it's a container). We can observe that it holds the `PID 1`.

> In Kubernetes, the pause container serves as the "parent container" for all of the containers in your pod. The pause container has two core responsibilities. First, it serves as the basis of Linux namespace sharing in the pod. And second, with PID (process ID) namespace sharing enabled, it serves as PID 1 for each pod and reaps zombie processes.
> 
> ---<cite>[The Almighty Pause Container][LK1]</cite>

If namespace sharing is not enabled each container has to do it's own housekeeping---it could not be a problem at all if the process running in the container does not spawn other processes. The problem can also be solved by using an `init` process inside each container like it has been demonstrated in the [previous article][LK2].

## References / Further reading

- [The Almighty Pause Container][LK1]
- [POD-PID-NAMESPACE / SHARED PID NAMESPACE][LK3]

[LK1]: https://www.ianlewis.org/en/almighty-pause-container
[LK2]: {{< ref "zombie_init.md" >}}
[LK3]: https://stupefied-goodall-e282f7.netlify.com/contributors/design-proposals/node/pod-pid-namespace/