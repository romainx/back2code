---
title: 'radian'
date: '2020-01-08'
categories: [data]
tags: ['r']
---

I'm a big fan of console tools, and I was wondering if there was a way to **color the R console** output.

I came across this tool called [Radian](https://github.com/randy3k/radian) with this attractive tagline.

> A 21 century R console

So let's try it!

# The Dockerfile

I've written a short `dockerfile` with `rocker/tidyverse` as base image in order to start with batteries included. 

```dockerfile
FROM rocker/tidyverse:latest

RUN apt-get update -qq && apt-get -y install \
    python3-pip && \
    pip3 install radian

CMD ["radian"]
```

Yes it must be shocking for R people, `radian` is written in ... Python.

# Build, Run, Try

```bash
# build
$ docker build --rm --pull -t radian .
# run
$ docker run -it --rm radian 
```

Now I can use a awesome looking console!
See it in action [^1].

![radian console](/post/radian/radian.png)

[^1]: Example taken from [magrittr](https://magrittr.tidyverse.org/articles/magrittr.html)