---
title: "Stack Overflow: reading data"
date: "2016-11-05"
categories: [dev]
---

One of the tedious thing in [Stack Overflow](http://stackoverflow.com/) is to **grab example data provided by users** in order to be able to use it to reproduce the case and try to solve it. Here is how to do it efficiently in **R** and **Python**, hope this will help you being the first to answer!

# In R

```r
library(tibble)


# Paste the text in a String, R allows multiline strings.
zz <- "Sepal.Length Sepal.Width Petal.Length Petal.Width Species
1          5.1         3.5          1.4         0.2  setosa
2          4.9         3.0          1.4         0.2  setosa
3          4.7         3.2          1.3         0.2  setosa
"

# I have not found the way to do the same thing with one of the readr function :-(
df <- read.table(text = zz, header = TRUE) %>% as_data_frame()
df %>% head()

## # A tibble: 3 × 5
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
##          <dbl>       <dbl>        <dbl>       <dbl>  <fctr>
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
## 3          4.7         3.2          1.3         0.2  setosa
```

And this is how to read data directly from the clipboard thanks to the 
`psych` package.

`df <- read.clipboard()`

# In Python

```python
import pandas as pd
import io

# Paste the text by using of triple-quotes to span String literals on multiple lines
zz = """Sepal.Length Sepal.Width Petal.Length Petal.Width Species
1          5.1         3.5          1.4         0.2  setosa
2          4.9         3.0          1.4         0.2  setosa
3          4.7         3.2          1.3         0.2  setosa
"""

df = pd.read_table(io.StringIO(zz), delim_whitespace=True)
df

##    Sepal.Length  Sepal.Width  Petal.Length  Petal.Width Species
## 1           5.1          3.5           1.4          0.2  setosa
## 2           4.9          3.0           1.4          0.2  setosa
## 3           4.7          3.2           1.3          0.2  setosa
```

It is also possible to read directly the text from the clipboard--under the hood this function calls `read_table`.

`df = pd.read_clipboard()`

# References / Further reading

* [Source code](https://gist.github.com/romainx/07a5a5dfeb663f29abebb92b14a64f05)
* [How to make a great R reproducible example](https://stackoverflow.com/questions/5963269/how-to-make-a-great-r-reproducible-example).
* [Pandas IO Tools](https://pandas.pydata.org/pandas-docs/stable/io.html)