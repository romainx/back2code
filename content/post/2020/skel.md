---
title: 'Skeletal profiles'
date: '2020-09-26'
categories: ['ops']
tags: ['linux']
---

> Sample startup files are traditionally kept in `/etc/skel`. If you customize your systems' startup file examples, `/usr/local/etc/skel` is a reasonable place to put the modified copies.[^1]

<!--more-->

## What is inside?

Default startup files are stored in `/etc/skel`.

```bash
$ ls -al /etc/skel

# total 20
# drwxr-xr-x 2 root root 4096 Sep 16 01:22 .
# drwxr-xr-x 1 root root 4096 Sep 25 19:20 ..
# -rw-r--r-- 1 root root  220 Feb 25  2020 .bash_logout
# -rw-r--r-- 1 root root 3771 Feb 25  2020 .bashrc
# -rw-r--r-- 1 root root  807 Feb 25  2020 .profile
```

This directory is used during the addition of users.

```bash
$ cat /etc/default/useradd

# [...]
# The SKEL variable specifies the directory containing "skeletal" user
# files; in other words, files such as a sample .profile that will be
# copied to the new user's home directory when it is created.
# SKEL=/etc/skel
```

## Customizing the startup file

In [Jupyter Docker Stacks][jupyter_stacks], we are customizing the startup `.bashrc` file[^2].
It is done before the creation of the default user (`jovyan`), so it will benefit for this customization. 
And the same goes for any other user that will be created---it's the case when images are spawned by [JupyterHub][jupyter-hub] for example.

Let's illustrate it.

```bash
# setting the color prompt by removing comment at the beginning of the line
$ sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc
# creating the new user
$ useradd -m -s /bin/bash -N -u 1000 jovyan
# login will use the customized .bashrc file
$ su jovyan
$ cat ~/.bashrc | grep force_color_prompt=yes

# force_color_prompt=yes
```

Here is the result, beautiful!

{{< figure src="/images/skel/jovyan.png" title="Color prompt" >}}

{{< admonition type=note >}}
The file is modified in place instead of being done in a copy in `/usr/local/etc/skel` like advised in the book because we are working on an image and not on a system that will be updated.
{{< /admonition >}}

[^1]: Nemeth, Evi, et al. *UNIX and Linux System Administration Handbook*. Addison Wesley, 2017.
[^2]: According to the *UNIX and Linux System Administration Handbook*, startup files traditionally begin with a dot and end with the letters `rc`, short for "run command", a relic of the CTSS operating system.

[jupyter_stacks]: https://github.com/jupyter/docker-stacks
[jupyter-hub]: https://jupyter.org/hub