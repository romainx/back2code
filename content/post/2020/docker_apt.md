---
title: 'APT usage in Docker images'
date: '2020-09-29'
categories: ['ops']
tags: ['docker', 'linux']
---

When packages are installed in a `Dockerfile` through **APT** (Debian, Ubuntu) you should comply with some **best practices**.
One of the main reason is to try to keep the image as as small as possible, but it's not the only one. 
Let's examine a typical snippet of package's installation.

<!--more-->

## Snippet

Here is a typical snippet of package installation in a `Dockerfile`.

```Dockerfile
FROM ubuntu:focal-20200916

# Use apt-get instead of apt that meant to be a end-user tool
# Downloads the package lists from the repositories and get information 
# about the newest versions of packages and their dependencies
RUN apt-get update && \ 
    # To avoid the installation of recommended packages (not essential)
    apt-get install -yq --no-install-recommends \
    # Put the list of packages in alphabetical order it's easier to maintain
    nano-tiny \
    # Use tiny version of packages when possible to reduce image's size
    vim-tiny && \
    # Clean the packages and install script in /var/cache/apt/archives/
    apt-get clean && \
    # Remove the package lists (created by apt-get update)
    rm -rf /var/lib/apt/lists/*
```

## Linter

These best practices can be enforced by the use of a `linter`.
[Hadolint][hadolint] is a good choice for that, I appreciate the clarity of the rules that are well documented and the simplicity of the tool.

It checks several rules related to APT.

- `DL3005`: Do not use apt-get upgrade or dist-upgrade.
- `DL3008`: Pin versions in apt-get install.
- `DL3009`: Delete the apt-get lists after installing something (`rm -rf /var/lib/apt/lists/*`)
- `DL3027`: Do not use apt as it is meant to be a end-user tool, use apt-get or apt-cache instead

If we run it on our example, it detects that the packages's versions are not pinned.
So builds done at different times could produce different images and break reproducibility.

```bash
$ hadolint Dockerfile

# Dockerfile:12 DL3008 Pin versions in apt get install. 
# Instead of `apt-get install <package>` use `apt-get install <package>=<version>`
```

## References / Further reading

- [What does `sudo apt-get update` do?](https://askubuntu.com/questions/222348/what-does-sudo-apt-get-update-do)
- [We reduced our Docker images by 60% with `--no-install-recommends`](https://ubuntu.com/blog/we-reduced-our-docker-images-by-60-with-no-install-recommends)
- [Clean, autoclean, and autoremove — combining them is a good step?](https://askubuntu.com/questions/984797/clean-autoclean-and-autoremove-combining-them-is-a-good-step)
- [Clear `apt-get list`](https://unix.stackexchange.com/questions/217369/clear-apt-get-list)

[hadolint]: https://github.com/hadolint/hadolint