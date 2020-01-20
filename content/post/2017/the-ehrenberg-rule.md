---
title: The Ehrenberg’s rule
date: '2017-08-21'
tags: ['readability']
categories: [data]
---

While reading *Data Analysis with Open Source Tools*[^1]--one of my favorite book on this topic--, I was surprised to discover that rounding numbers to display was described by a rule bearing the name of Andrew S. C. Ehrenberg.

> It is obviously pointless to report or quote results to more digits than is warranted. In fact, it is misleading or at the very least unhelpful, because it fails to communicate to the reader another important aspect of the result--namely its reliability! A good rule (sometimes known as Ehrenberg’s rule) is to quote all digits up to and including the *first two variable digits*.

As highlighted above, the problem of reporting two much detail is double:

* Decrease readability
* Not relevant since high precision can be not meaningful (in case of inaccuracy for example)

The **readability problem** is also tackled in *Show Me the Numbers: Designing Tables and Graphs to Enlighten*[^2].

> The level of precision should not exceed the level needed to serve your communication objectives and the needs of your readers.

This is often the case if you deal with financial data. Often decimal digits are not required when presenting data. Hence, a best practice is to use the good unit--e.g. k€ or M€ as highlighted by **Stephen Few** in his book[^2].

> Truncate the display of whole numbers by sets of three digits to the nearest thousand, million, billion, etc., whenever numeric precision can be reduced without the loss of meaningful information, and declare you’ve done so in the title or header.

# In Python

This can be achieved easily in pandas thanks to the **conditional formatting** feature knows as style . Here is the raw data.

```python
# Sample data
np.random.seed(0)
df = pd.DataFrame({'categ': pd.Categorical(np.random.choice(('foo', 'bar'), 100)),
                   'number': np.random.randint(1, 100, 100) * 100})
# Summarizing
grouped = df.groupby('categ').sum()

# categ      number
# -------  --------
# bar        294700
# foo        19770
```

And the data formatted--much clear.

```python
# Format definition
keur = lambda x: '{:.3g} k€'.format(x / 1000)

# The old way
style = grouped.applymap(keur)

# The conditional formatting way
style = grouped.style.format(keur)

# categ    number
# -------  --------
# bar      295 k€
# foo      198 k€
```

The **humanize** python package--a JavaScript equivalent also exists--can also help on this topic--I use it mainly for time delta.

```python
import humanize
import datetime

humanize.intword(1234550)
# '1.2 million'

humanize.naturaltime(datetime.datetime.now() - datetime.timedelta(weeks=60))
# '1 year, 1 month ago'
```

[^1]: Philipp K. Janert, *[Data Analysis with Open Source Tools](https://www.goodreads.com/book/show/8360735-data-analysis-with-open-source-tools)*, (O’Reilly, 2010)
[^2]: Stephen Few, *[Show Me the Numbers: Designing Tables and Graphs to Enlighten](https://www.goodreads.com/book/show/15809253-show-me-the-numbers)*, (Analytics Press, 2012).