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

## Some definitions


> - **Login**: Means that the shell is run as part of the login of the user to the system. Typically used to do any configuration that a user needs / wants to establish his work-environment.
> - **Non-login**: Any other shell run by the user after logging on, or which is run by any automated process which is not coupled to a logged in user.
> - **Interactive**: As the term implies: Interactive means that the commands are run with user-interaction from keyboard. E.g. the shell can > prompt the user to enter input.
> - **Non-interactive**: the shell is probably run from an automated process so it can't assume if can request input or that someone will see the > output. E.g Maybe it is best to write output to a log-file. [^2]

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

## Login shell

> When `bash` is invoked as an interactive login shell, or as a non-interactive shell with the `--login` option, it first reads and executes commands from the file `/etc/profile`, if that file exists. After reading that file, it looks for `~/.bash_profile`, `~/.bash_login`, and `~/.profile`, in that order, and reads and executes commands from the first one that exists and is readable. The `--noprofile` option may be used when the shell is started to inhibit this behavior.[^1]

### Interactive

In this case both startup scripts are invoked, both variables are set.

```bash
$ docker run -it --rm startup bash -l

$ echo "PROFILE -> $TEST_PROFILE, BASHRC -> $TEST_BASHRC"

# PROFILE -> 1603348756283, BASHRC -> 1603348756277
```

### Non-interactive

In this case only `/etc/profile` is invoked, only `TEST_PROFILE` is set.
Login shell is obtained by using the `-l` (for login) option.

```bash
$ docker run --rm startup bash -lc 'echo "PROFILE -> $TEST_PROFILE, BASHRC -> $TEST_BASHRC"' 

# PROFILE -> 1603372796585, BASHRC -> 
```

## Non-login shell

> When an interactive shell that is not a login shell is started, `bash` reads and executes commands from `~/.bashrc`, if that file exists. This may be inhibited by using the `--norc` option. The `--rcfile` file option will force `bash` to read and execute commands from file instead of `~/.bashrc`.

> When bash is started non-interactively, to run a shell script, for example, it looks for the variable `BASH_ENV` in the environment, expands its value if it appears there, and uses the expanded value as the name of a file to read and execute.

### Interactive

In this case only `/etc/bash.bashrc` is invoked, only `TEST_BASHRC` is set.

```bash
$ docker run -it --rm startup bash

$ echo "PROFILE -> $TEST_PROFILE, BASHRC -> $TEST_BASHRC"

# PROFILE -> , BASHRC -> 1603348681395
```

### Non-interactive

In this case nothing happen, both variables are empty.

```bash
$ docker run --rm startup

# PROFILE -> , BASHRC -> 
```

## Wrap up

What we've learned in short.

- **Login shell**: `/etc/profile` is invoked
- **Interactive shell**: `/etc/bash.bashrc` is invoked

See also my previous article on a very close topic [Skeletal profiles (`/etc/skel`)]({{< ref "skel.md" >}}).

## References / Further reading

- [What do you mean by interactive shell?](https://unix.stackexchange.com/questions/43385/what-do-you-mean-by-interactive-shell)
- [Why is `/etc/profile` not invoked for non-login shells?](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells)
- [What's the difference between `.bashrc` and `/etc/bash.bashrc`?](https://askubuntu.com/questions/815066/whats-the-difference-between-bashrc-and-etc-bash-bashrc)

[^1]: [bash - Linux man page](https://linux.die.net/man/1/bash)
[^2]: [What is the difference between interactive shells, login shells, non-login shell and their use cases?](https://unix.stackexchange.com/questions/50665/what-is-the-difference-between-interactive-shells-login-shells-non-login-shell)