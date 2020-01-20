---
title: 'Deprecation in R'
date: '2020-01-20'
categories: [dev]
tags: ['R']
---

After my [recent article]({{< ref "deprecated" >}}) about marking **deprecated** code in Python I had to do the same thing in **R**.
The good thing is that it's included in the language (in The R Base Package).

To deprecate an object simply make a call to the `.Deprecated()` function.
It will produce a warning---that can be suppressed by users by calling `suppressWarnings()`.

It's also possible to signal that an object has been removed through a call to `.Defunct()`. 
In this case it will produce an error, in other terms the program will stop working.

```R
old_function <- function() {
    .Deprecated(msg = "Will be removed in the next version")
    cat("\nI'm old now I want to retire...")
}

dead_function <- function() {
    .Defunct(msg = "Function removed :-(")
    cat("\nI'm dead now I will not chat anymore...")
}

old_function()
# Will be removed in the next version
# I'm old now I want to retire...

dead_function()
# Error : Function removed :-(
```

# References / Further reading

- [Package evolution - changing stuff in your package](https://ropensci.org/technotes/2017/01/05/package-evolution/)
- [Documentation](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/Deprecated)