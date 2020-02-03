---
title: 'Zombie Processes'
date: '2020-02-02'
categories: [ops]
tags: ['docker', 'kubernetes']
draft: True
---


# In Kubernetes

A simple descriptor to deploy the container.

```YAML
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



```bash
$ k logs zombie-init

# The zombie pid will be: 21
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root        15  0.0  0.5  13156 10596 ?        Ss   20:11   0:00 python3 /root/test.py
# root        22  0.0  0.1   9392  3080 ?        R    20:11   0:00  \_ ps xawuf
# root         1  1.0  0.0   1024     4 ?        Ss   20:11   0:00 /pause
```

# References / Further reading

- https://github.com/kubernetes/kubernetes/blob/release-1.10/build/pause/CHANGELOG.md
- https://www.ianlewis.org/en/almighty-pause-container
- https://stupefied-goodall-e282f7.netlify.com/contributors/design-proposals/node/pod-pid-namespace/
