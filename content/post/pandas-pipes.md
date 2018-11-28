---
title: Pandas pipes
date: '2016-11-13'
tags: [data]
---

![Magrittr](/post/pandas-pipes_files/magritrr.jpeg)

I love the ability of using **pipes** (with the dedicated operator `%>%`) in R introduced by the [magrittr](https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html) package -- we are using them for many years in *nix systems, the old good `|`. They let write data wrangling sequences in a very readable way -- very close to a natural language. See them in action.

```R
babynames %>% 
  filter(name == "Eva") %>%
  group_by(year) %>%
  summarise(n = sum(n)) %>% 
  ggplot(aes(x = year, y = n)) + geom_line()
```

Reading that and knowing the content of the [babynames](https://github.com/hadley/babynames) package, we can guess without effort that we are trying to plot the evolution of the number of babies (born in the USA) named "Eva" along the years.

![In R](/post/pandas-pipes_files/pipe-ggplot.png)

With **python pandas** there is no dedicated operator but calls to `DataFrame` methods can be chained by using the standard `.` operator that becomes in this case a kind of pipeline operator. It's great for built-in `DataFrame` methods but it is also possible to use the `pipe`[^pandas_pipe] method to wrap any function call. 

One additional drawback is the impossibility to indent the different calls properly. Fortunately a [trick](https://stackoverflow.com/questions/4768941/how-to-break-a-line-of-chained-methods-in-python) consisting in surrounding the pipeline by parenthesis permits to bypass this limitation. Here is what it looks like.

```python
(
babynames
  .query('name == "Eva"')
  .groupby('year')
  .agg({'n': 'sum'})
  .plot(kind = 'line')
)
```

![In Python](/post/pandas-pipes_files/pipe-pandas.png)

Not so bad!

# Pipe everything

The `pipe` method can be used to call any function and integrate this call in the pipeline.  Imagine you want to create a function (called `number_by_year`) to gather all your preparation code.

```python
def number_by_year(df, b_name):
    """From the babynames dataset, 
    count the number of babies by year bearing the given name.
    """
    return (df
            .query('name == @b_name')
            .groupby('year')
            .agg({'n': 'sum'})
           )

# This is how to use the function just defined
# in the pipeline thanks to the pipe method
(
babynames
    .pipe(number_by_year, b_name="Eva")
    .plot(kind = 'line')
)
```

And yes, it is possible to do the same thing in R.

```R
# From the babynames dataset, 
# count the number of babies by year bearing the given name.
number_by_year <- function(.data, b_name) {
  .data %>% 
  filter(name == b_name) %>%
  group_by(year) %>%
  summarise(n = sum(n))
}

babynames %>% 
  number_by_year(b_name = "Eva") %>% 
  ggplot(aes(year, n)) + geom_line()
```

[^pandas_pipe]: [Pipe method](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.pipe.html)
