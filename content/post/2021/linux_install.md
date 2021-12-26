---
title: 'Linux install command'
date: '2021-12-26'
categories: ['ops']
tags: ['Linux']
---

I did not know the Linux `install` command before. 

> This install program copies files (often just compiled) into destination locations you choose. 

Let's see how it can be useful.

<!--more-->

## The classic way

It's a common pattern to create a directory or to copy a file while setting the permissions and the ownership--I will keep only the directory example but it goes the same with the file.
The common pattern to do that is to run two commands

- Create the directory with `mkdir` and set the permissions
- Set ownership of the directory with `chown`.

Here is a typical example. 
Say we are `root` and we want to create a directory (`test`) with a specific mode (`0700`) and give the ownership to `vagrant:vagrant`.
Note I'm using the verbose flag (`v`) which is very convenient to check what is going on without looking to the directory.

```bash
# create a directoryn with the 
mkdir -v -m 0700 /tmp/test
# mkdir: created directory 'test'
chown -v vagrant:vagrant /tmp/test
# changed ownership of '/tmp/test' from root:root to vagrant:vagrant
```

## Using the `install` command

The same result can be obtained in a single `install` command.

```bash
install -v -d -m 0600 -o vagrant -g vagrant /tmp/test
# install: creating directory '/tmp/test'
# checking
stat -c "%n %U:%G %a" /tmp/test/
# /tmp/test/ vagrant:vagrant 600
```

Using `install` is not revolutionary however it's good to know and it could be convenient to use it for example in `Dockerfile`s.

## References / Further reading

- [Stackoverflow answer](https://stackoverflow.com/a/25873952/4413446)
- [`install` man page](https://linux.die.net/man/1/install)