---
title: 'Zombie Processes'
date: '2020-02-03'
categories: [ops]
tags: ['docker']
---

## Zombie processes, a short definition

The first step is an **orphaned** process, a process that has lost his parent.

> Suppose the parent process terminates, either intentionally (because the program logic has determined that it should exit), or caused by a user action (e.g. the user killed the process). What happens then to its children? They no longer have a parent process, so they become "orphaned" (this is the actual technical term).
>
> --- <cite>[source][LK-1]</cite>

<!--more-->

The init process is responsible of **adopting** orphaned processes and to **reap** them, i.e. to clean it up.

> And this is where the init process kicks in. The init process---`PID 1`---has a special task. Its task is to "adopt" orphaned child processes (again, this is the actual technical term). This means that the init process becomes the parent of such processes, even though those processes were never created directly by the init process

If like in standard docker container launching a command, there is no proper `init` process, nobody will care about orphaned processes and they will stay here as **zombies** also called `defunct`. The problem is not related to the resources used by these zombies (none) but to the number of processes that will increase until system exhaustion.

> As long as a zombie is not removed from the system via a wait, it will consume a slot in the kernel process table, and if this table fills, it will not be possible to create further processes

## Producing a zombie process

To generate a zombie process I have used the Python code displayed in [this issue](https://github.com/Yelp/dumb-init/issues/128).

```python
#!/usr/bin/env python3
import os
import subprocess


pid = os.fork()
if pid == 0:  # child
    pid2 = os.fork()
    if pid2 != 0:  # parent
        print('The zombie pid will be: {}'.format(pid2))
else:  # parent
    os.waitpid(pid, 0)
    subprocess.check_call(('ps', 'xawuf'))
```

Then a simple `Dockerfile` to package and run it.

```dockerfile
FROM python

COPY test.py /root/
RUN chmod +x /root/test.py

CMD ["/root/test.py"]
```

Ready to test!

```bash
# Build
$ docker build --rm -t zombie-init ./zombie-init

# Run
$ docker run --rm zombie-init

# The zombie pid will be: 7
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.5  13164 10744 ?        Ss   09:44   0:00 python3 /root/test.py
# root         7  0.0  0.0      0     0 ?        Z    09:44   0:00 [python3] <defunct>
# root         8  0.0  0.1   9392  3068 ?        R    09:44   0:00 ps xawuf
```

We can observe a zombie (`defunct`) process that is orphan, it has the `PID #7`. This process will remain here since there is no process that will care about reaping (removing) it. In this case it's not an issue, but imagine if the main process you are running creates a lot of child processes (like in a loop) that become orphans and then zombies.

## Killing zombies

How to get rid of zombies?

### Use the docker `--init` flag

Since the Docker `1.13` version there is a special `--init` flag that can be used to tell Docker to use an init system that will reap zombies.

> You can use the `--init` flag to indicate that an init process should be used as the `PID 1` in the container. Specifying an init process ensures the usual responsibilities of an init system, such as reaping zombie processes, are performed inside the created container.
>
> --- <cite>[source](https://docs.docker.com/engine/reference/#specify-an-init-process)</cite>

Let's check if it works.

```bash
# With init flag
$ docker run --init --rm zombie-init

# The zombie pid will be: 8
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  2.0  0.0   1052     4 ?        Ss   09:42   0:00 /sbin/docker-init -- /root/test.py
# root         6  0.0  0.5  13164 10808 ?        S    09:42   0:00 python3 /root/test.py
# root         9  0.0  0.1   9392  3004 ?        R    09:42   0:00  \_ ps xawuf
```

Yes, it works the `defunct` process (`#8`) is gone!

### Using an `init` for containers

There are several `init` solutions for containers and mainly

- [tini](https://github.com/krallin/tini)
- [dumb-init](https://github.com/Yelp/dumb-init)

I've chosen here to use **tini** since it's included in Docker (it's in fact the `--init` flag seen just before).

> The default init process used is the first docker-init executable found in the system path of the Docker daemon process. This docker-init binary, included in the default installation, is backed by tini.

--- [source](https://docs.docker.com/engine/reference/#specify-an-init-process)

To use it several choices are available, one of them is to download and install it directly in the `dockerfile`.

```dockerfile
# ---- TINI ----
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]
# [...]
CMD ["/root/test.py"]
```

It is included in some base images and can also be installed through `conda`.

```bash
$ conda install -c conda-forge tini
```

## References / Further reading

- [Docker and the PID 1 zombie reaping problem][LK-1]
- [Docker - init, zombies - why does it matter?](https://stackoverflow.com/questions/49162358/docker-init-zombies-why-does-it-matter/)
- [Introducing dumb-init, an init system for Docker containers](https://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html)

[LK-1]: https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/