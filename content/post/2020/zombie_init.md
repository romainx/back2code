---
title: 'Zombie Processes'
date: '2020-02-02'
categories: [ops]
tags: ['docker', 'kubernetes']
draft: True
---

# Zombie processes, a short definition

# Producing a zombie process

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

Ready to test.

```bash
# Build
$ docker build --rm -t zombie-init ./zombie-init

# Without init
$ docker run --rm zombie-init

# The zombie pid will be: 7
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.5  13164 10744 ?        Ss   09:44   0:00 python3 /root/test.py
# root         7  0.0  0.0      0     0 ?        Z    09:44   0:00 [python3] <defunct>
# root         8  0.0  0.1   9392  3068 ?        R    09:44   0:00 ps xawuf
```

We can observe a zombie (`defunct`) process that is orphan, it has the `PID #7`. This process will remain here since there is no process that will care about removing (reaping) it. In this case it's not an issue, but imagine if the main process you are running create a lot of child processes (like in a loop) that become orphans and zombies.

# Killing zombies

Since the Docker `1.13` version there is a special `--init` flag that can be used to reap zombie processes.

> You can use the --init flag to indicate that an init process should be used as the PID 1 in the container. Specifying an init process ensures the usual responsibilities of an init system, such as reaping zombie processes, are performed inside the created container.

--- [source](https://docs.docker.com/engine/reference/#specify-an-init-process)

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

Yes it works the `defunct` process (`#8`) is gone.

# With Kubernetes


# Using an `init` for containers

There are several `init` solutions for containers and mainly

- [tini](https://github.com/krallin/tini)
- [dumb-init](https://github.com/Yelp/dumb-init)

I've chosen here to use **tini** since it's included in Docker (it's in fact the `--init` flag seen just before).

> The default init process used is the first docker-init executable found in the system path of the Docker daemon process. This docker-init binary, included in the default installation, is backed by tini.

--- [source](https://docs.docker.com/engine/reference/#specify-an-init-process)

To use it can be used like that.

TODO: check if it can be installed through conda also available in pre-built docker images.

```dockerfile
# ---- TINI ----
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]
# [...]
CMD ["/root/test.py"]
```

```bash
$ k logs zombie-init                        

The zombie pid will be: 8
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   2280   688 ?        Ss   16:29   0:00 /tini -- /root/test.py
root         6  0.0  0.5  13120 10760 ?        S    16:29   0:00 python3 /root/test.py
root         9  0.0  0.1   9392  3036 ?        R    16:29   0:00  \_ ps xawuf
```