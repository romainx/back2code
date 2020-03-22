---
title: Feather
date: '2016-11-11'
categories: ['data']
tags: ['python', 'r']
---

We often read articles on the topic **Python vs. R, which language should I pick?** My opinion is that each language, and its corresponding ecosystem, has its pros and cons and can be used efficiently to solve different problems.

**Wes McKinney** and **Hadley Wickham** seem to agree on this point and have recently developed in strong collaboration the [Feather](https://github.com/wesm/feather) packages (one in Python and one in R at this time, but it could / will be extended to other languages).

> It is designed to make reading and writing data frames efficient, and to make sharing data across data analysis languages easy.

Stop talking, it’s time to experiment. Here is a “hello world” of writing a data frame in R and reading it back in Python (pandas).

```r
library(feather)

data("mtcars")
path <- "/tmp/mtcars.feather"
write_feather(mtcars, path)
```

Now we can read it in Python and start playing with `mtcars` data ;-)

```python
import feather

path = '/tmp/mtcars.feather'
df = feather.read_dataframe(path)

df.head(3)
##     mpg  cyl   disp     hp  drat     wt   qsec   vs   am  gear  carb
## 0  21.0  6.0  160.0  110.0  3.90  2.620  16.46  0.0  1.0   4.0   4.0
## 1  21.0  6.0  160.0  110.0  3.90  2.875  17.02  0.0  1.0   4.0   4.0
## 2  22.8  4.0  108.0   93.0  3.85  2.320  18.61  1.0  1.0   4.0   1.0
```

A last world, Feather uses the [Apache Arrow](https://arrow.apache.org/) columnar memory specification, but at this time it should not be used for long-term storage since it is likely to change.

> Note to users: Feather should be treated as alpha software. In particular, the file format is likely to evolve over the coming year. Do not use Feather for long-term data storage.

# References / Further reading

* [Project on GitHub](https://github.com/wesm/feather)
* [Post on Cloudera blog](http://blog.cloudera.com/blog/2016/03/feather-a-fast-on-disk-format-for-data-frames-for-r-and-python-powered-by-apache-arrow/)
* [Post on RStudio blog](https://blog.rstudio.org/2016/03/29/feather/)