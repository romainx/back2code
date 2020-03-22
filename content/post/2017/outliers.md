---
title: Outliers
date: '2017-08-19'
categories: ['data']
tags: ['stats', 'python']
---

# Definition

> An *outlier* is a data point or observation whose value is quite different from the others in the dataset being analyzed.[^1]

It is an important part of the analysis to identify outliers and to use appropriate techniques to take them into account.
But unfortunately 😔

> There is no absolute agreement among statisticians about how to define *outliers* […][^1]

So what can be done?
Fortunately 😏

> Various rules of thumb have been developed to make the identification of outliers more consistent.[^1]

One common definition uses the concept of **interquartile range (IQR)**.

# IQR

> The *interquartile range* [IQR] is the range of the middle 50% of the values in a data set, which is calculated as the difference between the 75th [upper quartile Q3] and 25th percentile [lower quartile Q1] values.[^1]

And now how to use IQR to identify and remove outliers—filter values?

# Using IQR to find outliers

> […] mild outliers are those lower than the 25th quartile [Q1] minus 1.5 x IQR or greater than the 75th quartile [Q3] plus 1.5 x IQR. [^1]

And about the rationale

> Cases this extreme are expected in about 1 in 150 observations in normally distributed data.[^1]

On other example of the common usage of the **1.5 factor** is that it is generally taken as the default value in box plot implementations like in matplotlib , the python main plotting library.

> **whis** : float, sequence, or string (default = 1.5)
> As a float, determines the reach of the whiskers to the beyond the first and third quartiles. In other words, where IQR is the interquartile range (Q3-Q1), the upper whisker will extend to last datum less than Q3 + whis x IQR).[^2]

A last word to say that this 1.5 factor can be substituted by higher values.

> 3 x IQR […] are expected about once per 425 000 observations in a normally distributed data.[^1]

# In Python

Great, but how to use it in Python + Pandas to filter values in a dataset ?
Here is a simple solution taken from a quite popular [answer I made on Stack Overflow](https://stackoverflow.com/questions/34782063/how-to-use-pandas-filter-with-iqr).


## 1. Producing some test data

```python
import pandas as pd
import numpy as np
%matplotlib inline

# Some test data
np.random.seed(33454)
df = (
    # A standard distribution
    pd.DataFrame({'nb': np.random.randint(0, 100, 20)})
        # Adding some outliers
        .append(pd.DataFrame({'nb': np.random.randint(100, 200, 2)}))
        # Reseting the index
        .reset_index(drop=True)
    )
```

## 2. Computing IQR

```python
Q1 = df['nb'].quantile(0.25)
Q3 = df['nb'].quantile(0.75)
IQR = Q3 - Q1
```

## 3. Filtering data

It makes use of the pandas `query` method for clarity.

```python
#Values between Q1-1.5IQR and Q3+1.5IQR
filtered = df.query('(@Q1 - 1.5 * @IQR) <= nb <= (@Q3 + 1.5 * @IQR)')
```

## 4. Plotting the result to check the difference

```python
df.join(filtered, rsuffix='_filtered').boxplot()
```
![outliers](/post/outliers_files/outliers.png)

*Note: SciPy proposes [an implementation of the IQR computing](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.boxplot.html)  `scipy.stats.iqr`.*

# Conclusion

Outliers identification based on IQR is a **useful technique simple and generally accepted**. So it can be used at least as a first tool during exploratory analysis.

[^1]: Sarah Boslaugh, *[Statistics in a Nutshell](https://www.goodreads.com/book/show/15808133-statistics-in-a-nutshell)* (O'Reilly, 2012)
