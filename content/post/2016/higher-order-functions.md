---
title: Higher-order Functions
date: '2016-12-16'
categories: ['dev']
tags: ['scala', 'R', 'python']
---

**Martin Odersky**[^1] in his *Functional Programming in Scala* course illustrates **higher-order functions**[^2] with a simple example in Scala. Scala is a modern functional programming language where functions are *first-class citizen* so they can be used, like any other value, as a parameter and returned as a result.

```scala
// Simple sum function using a recursion to perform an operation on integers between a and b
def sum(f: Int => Int, a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sum(f, a + 1, b)

// Using anonymous function -- addict to syntactic sugar -- to define
// The sum of integers between a and b
def sumInts(a: Int, b: Int) = sum(x => x, a, b)
// The sum of the cubes of integers between, a and b
def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)

> sumInts(1, 10)
Int = 55
> sumCubes(1,10)
Int = 3025
```

I love the Scala lean syntax--and its syntactic sugars--, but Python and R--my old friends--are also, among other paradigms, functional languages. So how is it possible to perform the same things?

# In Python

The syntax is very close--and the results are the same :-). Anonymous functions are called *lambda*.

```python
# The way to define this recursive function is almost the same -- just sad to have to write return statements
def sum(f, a, b):
    if (a > b): return 0
    else: return f(a) + sum(f, a + 1, b)

# In Python anonymous functions are called lambda and are defined as follow
def sum_ints(a, b): 
    return sum(lambda x: x, a, b)

def sum_cubes(a, b): 
    return sum(lambda x: x * x * x, a, b)

>>> sum_ints(1, 10)
55
>>> sum_cubes(1, 10)
3025
```

# In R

Mainly the same thing. I’m not an expert in R and maybe there is a way to use the formula notation (`~`) instead of the anonymous function notation, but I don’t know.

```R
# Brackets have been ommited
sum <- function(f, a, b) {
  if (a > b) return(0)
  else return(f(a) + sum(f, a + 1, b))
}

# Anonymous function are also possible
sum_ints <- function(a, b) {
return(sum((function(x) x), a, b))
}

sum_cubes <- function(a, b) {
return(sum((function(x) x * x * x), a, b))
}

> sum_ints(1, 10)
55 
> sum_cubes(1, 10)
3025
```

In the next course, he introduces more compact syntax, but this is another story.

[^1]: [Martin Odersky](http://lampwww.epfl.ch/~odersky/) has designed the Scala programming language.
[^2]: [Lecture 2.1 - Higher-Order Functions - École Polytechnique Fédérale de Lausanne | Coursera](https://www.coursera.org/learn/progfun1/lecture/xuM1M/lecture-2-1-higher-order-functions)

