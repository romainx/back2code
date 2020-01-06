---
title: 'Playing with Haskell'
date: '2019-07-01'
categories: [dev]
tags: ['haskell']
---

While I was reading *Seven Languages in Seven Weeks* [^RF:Tate_2010], I wanted to test **Haskell**.
Just for fun.

# Running Haskell

To run Haskell one solution is to use a **Docker image**. 
It avoids the burden of installing it on your machine.

To make it even easier, a good old `Makefile` can help.

```makefile
# Assuming you have an src folder it will run the image
# and mount the src volume in /root/src
current_dir = $(shell pwd)
image = "haskell"
tag = "8"
image_tag = "${image}:${tag}"

GHCi:
	@echo "Running ${image_tag} in console mode ..."
	@docker run -v ${current_dir}/src:/root/src -w "/root/src" -it --rm ${image_tag} ghci
```

Once defined, to run `GHCi` (stands for Glasgow Haskell Compiler Interactive), just run the `make`. 

```sh
$ make

# Running haskell:8 in console mode ...
# Unable to find image 'haskell:8' locally
# 8: Pulling from library/haskell
# 146bd6a88618: Pull complete
# 27eaafd2ddfb: Pull complete
# Digest: sha256:e6b4675835bce283d610179c3ee67840ce9b702745f1f7709c121e07d41e0c5d
# Status: Downloaded newer image for haskell:8
# GHCi, version 8.8.1: https://www.haskell.org/ghc/  :? for help

Prelude>
```

# Playing with it

Now you can start playing.

```haskell
-- A dumb example
Prelude> double x = x + x
Prelude> double 4
8
```

# Next steps

My advice is to read the great book *Learn You a Haskell for Great Good!* [^RF:Lipovaca_2011].
Even better, it can be read [online](http://learnyouahaskell.com/) for free.

[^RF:Tate_2010]: Bruce A. Tate, *Seven Languages in Seven Weeks*, 1ʳᵉ éd., O′Reilly, 2010, 368 p.
[^RF:Lipovaca_2011]: Miran Lipovaca, *Learn You a Haskell for Great Good!: A Beginner’s Guide*, No Starch Press, 2011, 400 p.
