---
title: 'Littler'
date: '2020-01-06'
categories: [data]
tags: ['R']
---

`R` packages have often funny names. [littler](http://dirk.eddelbuettel.com/code/littler.html) stands for little `R`, this means lower case `r`.

<!--more-->

> littler provides the `r` program, a simplified command-line interface for GNU R. This allows direct execution of commands, use in piping where the output of one program supplies the input of the next, as well as adding the ability for writing *hash-bang* scripts, i.e. creating executable files starting with, say, `#!/usr/bin/r`.

I have discovered this tool while exploring a `dockerfile` provided by the [Rocker project](https://www.rocker-project.org/).

```dockerfile
RUN install2.r --error \
    --deps TRUE \
    tidyverse
```

I was wondering what was `install2.r`?

In fact it's one of the recipe -- and maybe one of the most interesting -- provided by this package.

It's installed by default in the rocker images, so let's test it.

```bash
# Running a bash with the r-base image
$ docker run --rm -ti rocker/r-base /bin/bash
# Now to install a packge (crayon is a tool to color terminal outputs) simply call
$ install.r crayon
# Then littler can also be used to run R from the command line
$ r -e 'library(crayon); cat(blue("Blue Text\n"))'
# Blue Text
```

I was using the following command to script the installation of packages, so I will give it a try.

```bash
# More verbose and error-prone syntax 
R --slave -e 'install.packages("crayon")'
```

It also brings other features like executing `RMarkdown`, check the [littler vignette](https://cran.r-project.org/web/packages/littler/vignettes/littler-examples.html) for more examples.
