---
title: Pandas apply and map
date: '2016-11-20'
categories: [data]
---

The way to apply a function to pandas  data structures is not always obvious--several methods exist (`apply`, `applymap`, `map`) and their scope is different.

First there is two main structures (fortunately I'm not talking about `Panel` here):

* `Series`: one-dimensional labeled array
* `DataFrame`: 2-dimensional labeled data structure

The `apply` / `map` methods can work on different ways.

# Element-wise

The function is called (mapped) for each individual element (value)--so it takes the element (each distinct value) as parameter.

* `map` for a `Series`: can be used with either a dict, a function, or a `Series`.
* `applymap` for a `DataFrame`: It is equivalent to calling map on all columns of the `DataFrame`.

# By row / column

The function is called (applied) for an entire row or a column--so it takes a row or a column as parameter, in other words a `Series`.

* `apply` for a `DataFrame` that can be called with an axis parameter indicating to apply to column (`0`) or to row (`1`).
* `apply` can also be used with a `Series`: it will only work for the entire array when used with a numpy universal function `ufunc`. So it's not working element-wise, however when used with standard function it will work element-wise.

In short, apply works on **row / column** of a `DataFrame`, `applymap` works **element-wise** on a `DataFrame`, and `map`--and `apply` for most cases--works **element-wise** on a `Series`.

# References / Further reading

1. Wes McKinney, *[Python for Data Analysis](https://www.goodreads.com/book/show/14744694-python-for-data-analysis)* ( O'Reilly, 2012)
2. [Difference between map, applymap and apply methods in Pandas](http://stackoverflow.com/questions/19798153/difference-between-map-applymap-and-apply-methods-in-pandas)