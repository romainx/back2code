---
title: 'Linux startup scripts'
date: '2020-10-22'
categories: ['ops']
tags: ['docker', 'linux']
---

When it comes to perform customization to startup scripts in order to initialize or tune something, it's not obvious to pick the right **startup file**. The way the system is reading startup files depends on the context and mainly if the shell is

- **Interactive** / **non-interactive** shell
- **Login** / **non-login** shell

<!--more-->

## Setting up test image

The best way to figure out what happens is to perform a simple test.
In this test I build a simple docker image exporting different environment variables each storing the current number of milliseconds since epoch.

- `TEST_PROFILE` is set by `/etc/profile`
- `TEST_BASHRC` is set by `/etc/bash.bashrc`

{{< admonition type=note >}}
Those files are system-wide files, they can be completed by user's own startup files `~/.profile` and `~/.bashrc`.  
`/etc/profile` is the main script but fragment of startup files can be put in `/etc/profile.d` to be executed. 
{{< /admonition >}}

```Dockerfile
FROM ubuntu:focal

# number of ms since epoch
RUN echo 'export TEST_PROFILE="$(date +%s%3N)"' >> /etc/profile && \
    echo 'export TEST_BASHRC="$(date +%s%3N)"' >> /etc/bash.bashrc

CMD echo "PROFILE -> $TEST_PROFILE, BASHRC -> $TEST_BASHRC"
```

We can now build the `startup` image `docker build --rm -t startup`.

## Non-interactive & non-login shell

In this case nothing happen, both variables are empty.

```bash
$ docker run -t --rm startup

# PROFILE -> , BASHRC -> 
```

## Interactive & non-login shell

In this case only `/etc/bash.bashrc` is invoked, only `TEST_BASHRC` is set.

```bash
$ docker run -it --rm startup bash

$ echo "PROFILE -> $TEST_PROFILE, BASHRC -> $TEST_BASHRC"

# PROFILE -> , BASHRC -> 1603348681395
```

## Interactive & login shell

In this case both startup scripts are invoked, both variables are set.

```bash
$ docker run -it --rm startup bash -l

$ echo "PROFILE -> $TEST_PROFILE, BASHRC -> $TEST_BASHRC"

# PROFILE -> 1603348756283, BASHRC -> 1603348756277
```

We can notice that the `TEST_BASHRC` variable is set first.

## Wrap up

What we've learned

- **Non-interactive**: None
- **Interactive**
  - **Non-login** shell: `/etc/bash.bashrc`
  - **Login shell**: `/etc/bash.bashrc` and `/etc/profile`

See also my previous article on a very close topic [Skeletal profiles (`/etc/skel`)]({{< ref "skel.md" >}}).

## References / Further reading

- [What do you mean by interactive shell?](https://unix.stackexchange.com/questions/43385/what-do-you-mean-by-interactive-shell)
- [Why is `/etc/profile` not invoked for non-login shells?](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells)
- [What's the difference between `.bashrc` and `/etc/bash.bashrc`?](https://askubuntu.com/questions/815066/whats-the-difference-between-bashrc-and-etc-bash-bashrc)