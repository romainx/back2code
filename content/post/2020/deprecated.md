---
title: 'Deprecation in Python'
date: '2020-01-17'
categories: [dev]
tags: ['python']
---

It's always a good practice to **deprecate** functions, methods or classes before removing or changing something.

<!--more-->

> Features are deprecated rather than immediately removed, to provide backward compatibility, and to give programmers time to bring affected code into compliance with the new standard.
>
> --- <cite>[Deprecation](https://en.wikipedia.org/wiki/Deprecation)</cite>

It permits to perform changes and to clean code---because it's always necessary to make things evolve---while avoiding breaking things.

In Java there is a built-in annotation for that `@Deprecated`. However in **Python** there is no such a feature.

But as you may no, if something is missing, someone with the same concerns has already built it.
There is several implementations, I've chosen [Deprecated](https://github.com/tantale/deprecated) since 

- it's very simple and raises a simple **warning**---more on that later---that is printed at runtime. 
  In fact my first need was to print this warning clearly in Jupyter notebooks,
- it provides a very convenient way to deprecate things through a **decorator**. 

## Install

```bash
$ pip install deprecated
```

## Deprecate things

```python
from deprecated import deprecated

@deprecated(reason="use new_function")
def old_function():
    print("I'm old now I want to retire...")

def new_function():
    print("I'm ready to conquer the world!")

>>> old_function()

# __main__:1: DeprecationWarning: Call to deprecated function 
# (or staticmethod) old_function. (use new_function)
# I'm old now I want to retire...
```

## How it works

It uses a Python standard library called `warnings`[^1]. 

> Warning messages are typically issued in situations where it is useful to alert the user of some condition in a program, where that condition (normally) doesn’t warrant raising an exception and terminating the program. For example, one might want to issue a warning when a program uses an obsolete module.

They are managed as `Exception` and a more precisely by raising a `DeprecationWarning` printed to the `sys.stderr`---and so deprecation messages appear clearly in red in Jupyter notebooks.

> Base class for warnings about deprecated features when those warnings are intended for other Python developers.

[^1]: [warnings — Warning control](https://docs.python.org/3/library/warnings.html)