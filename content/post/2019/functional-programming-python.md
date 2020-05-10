---
title: 'Functional Programming in Python'
date: '2019-12-27'
categories: [dev]
tags: ['python']
---

Functional programming can also be interesting in **Python**. Here are some useful snippets.

<!--more-->

## Lambda

A lambda expression is an **anonymous function**.

```python
# Simple power function
f = lambda x: x*x
[f(x) for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## Map

`map` is a **higher-order function** that allows to apply a function to every element in an `iterable` object and it returns itself an `iterable`. 

```python
m = map(f, range(10))
list(m)
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## Filter

The filter function **tests** each element in an `iterable` object with a function that returns either `True` or `False`.

```python
# In this case it's an even filter
even = filter(lambda x: x % 2 == 0, range(10))

list(even)
# [0, 2, 4, 6, 8]

# Combine map and filter
even = map(f, filter(lambda x: x % 2 == 0, range(10)))

list(even)
# [0, 4, 16, 36, 64]
```

## Reduce

Apply function of two arguments cumulatively to the items of sequence, from left to right, so as to **reduce** the sequence to a **single value**.

```python
from functools import reduce

reduce(lambda x,y: x+y, range(10))
# 45
```

## Head & Tail

`head` and `tail` are idiomatic of functional programming languages.

```python
head, *tail = range(10)
head
# 0

tail
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## List Comprehension

```python
[n for n in range(10) if n % 2 == 0]
# [0, 2, 4, 6, 8]
```

## Any and All

Check if `any` or `all` conditions are met.
They can also be seen as series of logical `or` and `and` operators, respectively.

```python
any(x > 10 for x in range(10))
# False

all(x >= 0 for x in range(10))
# True
```

## References / Further reading

- [Functional Programming in Python](https://stackabuse.com/functional-programming-in-python/)
- [4. Map, Filter and Reduce - Python Tips 0.1 documentation](http://book.pythontips.com/en/latest/map_filter.html)
- [functools - Higher-order functions and operations on callable objects - Python 3.7.4 documentation](https://docs.python.org/3/library/functools.html)
- [Head and tail in one line](https://stackoverflow.com/questions/10532473/head-and-tail-in-one-line)
