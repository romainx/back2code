---
title: Functional Sequences
date: '2016-11-29'
categories: [data]
---

This feature blew my mind. R permits to define **functional sequences**.

> You can use `%>%` to not only produce *values* but also to produce *functions* (or *functional sequences*)! It’s really all the same, except sometimes the function is applied instantly and produces a result, and sometimes it is not, in which case the function itself is returned.[^fn-mag]

Here is an example.

[^fn-mag]: [Blog entry for magrittr 1.5](https://blog.rstudio.com/2014/12/01/magrittr-1-5/)

```R
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(babynames))

# Defining the functional sequence 
prepare <- . %>%
  group_by(year, sex) %>% 
  summarise(n = sum(n))
```

It is another, more readable way, to define a unary function--or to define a pipeline without applying it to the data. It is possible to get back its description.


```R 
prepare
# Functional sequence with the following components:

#  1. group_by(., year, sex)
#  2. summarise(., n = sum(n))

# Use 'functions' to extract the individual functions.
```

It can be included in a pipeline say to apply it to different data or to make the pipeline shorter and split it in simple, understandable operations.

```R
babynames %>% 
  prepare %>%
  ggplot() +
  geom_bar(aes(x = year, y = n, fill = sex), 
           stat = 'identity')
```

![plot](/post/functional-sequences_files/func.png)