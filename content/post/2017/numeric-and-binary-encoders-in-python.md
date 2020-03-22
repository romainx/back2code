---
title: Numeric and Binary Encoders in Python
date: '2017-09-24'
categories: ['data']
tags: ['python']
---

> A categorical variable, as the name suggests, is used to represent categories or labels. For instance, a categorical variable could represent major cities in the world, the four seasons in a year, or the industry (oil, travel, technology) of a company. […] The categories of a categorical variable are usually not numeric. [..] Thus, an encoding method is needed to turn these non-numeric categories into numbers.[^1]

Both Pandas  and scikit-learn  propose encoders to deal with categorical variables. But the distinction between each technique and implementation is not obvious. This is what I will try to clarify it in this article.

Basically we can distinct two kinds of encoder:

* Encode **labels (categorical variables) into numeric variables**: Pandas `factorize` and scikit-learn `LabelEncoder`. The result will have 1 dimension.
* Encode **categorical variable into dummy/indicator (binary) variables**: Pandas `get_dummies` and scikit-learn `OneHotEncoder`. The result will have n dimensions (or n-1 dimensions), one by distinct value of the encoded categorical variable.

The main difference between pandas and scikit-learn encoders is that they are made to be used in **scikit-learn pipelines** with `fit` and `transform` methods.

# Encode labels into numerical variables

Pandas `factorize` and scikit-learn `LabelEncoder` belong to the first category. They can be used to encode labels (nonnumerical variables) to numerical variables.

```python
from sklearn import preprocessing
# Test data
df = DataFrame(['A', 'B', 'B', 'C'], columns=['Col'])
df['Fact'] = pd.factorize(df['Col'])[0]
le = preprocessing.LabelEncoder()
df['Lab'] = le.fit_transform(df['Col'])

#   Col  Fact  Lab
# 0   A     0    0
# 1   B     1    1
# 2   B     1    1
# 3   C     2    2
```

But pay attention since converting nonnumerical variables to numbers is not the end of the road.

> The values may be represented numerically. However, unlike other numeric variables, the values of a categorical variable cannot be ordered with respect to one another. (Oil is neither greater than nor less than travel as an industry type.) They are called non-ordinal.$^1$

# Encode categorical variable into dummy/indicator (binary) variables

Pandas `get_dummies` and scikit-learn `OneHotEncoder` can be used to create binary variables. `OneHotEncoder` can only be used with categorical integers while get_dummies can be used with other type of variables. Another difference is that they refer to two feature engineering techniques:

* **One-hot encoding**: It uses k bit to encode k values. It is implemented by both `OneHotEncoder` and `get_dummies`.
* **Dummy coding**: It uses k-1 bit to encode k values. It is implemented only by `get_dummies`.

```python
df = DataFrame(['A', 'B', 'B', 'C'], columns=['Col'])
df = pd.get_dummies(df)

#    Col_A  Col_B  Col_C
# 0    1.0    0.0    0.0
# 1    0.0    1.0    0.0
# 2    0.0    1.0    0.0
# 3    0.0    0.0    1.0
```

We can see here that 3 bits are required to encode 3 distinct values where the variable itself needs only 2 bits (k-1 bits). In this case the extra feature is dropped (`A`) thanks to the parameter drop_first and so it is represented implicitly by all 0.

> This is known as the reference category.

```python
df = DataFrame(['A', 'B', 'B', 'C'], columns=['Col'])
df = pd.get_dummies(df, drop_first=True)

# Col_B  Col_C
# 0      0      0
# 1      1      0
# 2      1      0
# 3      0      1
```

It’s trickier to do the same thing with scikit-learn since data has to be converted first to numeric before using the `OneHotEncoder`.

```python
from sklearn.preprocessing import OneHotEncoder, LabelEncoder

df = DataFrame(['A', 'B', 'B', 'C'], columns=['Col'])
# We need to transform first character into integer in order to use the OneHotEncoder
le = preprocessing.LabelEncoder()
df['Col'] = le.fit_transform(df['Col'])
enc = OneHotEncoder()
df = DataFrame(enc.fit_transform(df).toarray())

#      0    1    2
# 0  1.0  0.0  0.0
# 1  0.0  1.0  0.0
# 2  0.0  1.0  0.0
# 3  0.0  0.0  1.0
```

*Note: This post is an augmented version of my Stack Overflow answer[^2]*

[^1]: Alice Zheng, *[Mastering Feature Engineering](https://www.goodreads.com/book/show/31393737-mastering-feature-engineering)*, (O'Reilly, 2016)
[^2]: [Want to know the diff among pd.factorize, pd.get_dummies, sklearn.preprocessing.LableEncoder and OneHotEncoder](https://stackoverflow.com/questions/40336502/want-to-know-the-diff-among-pd-factorize-pd-get-dummies-sklearn-preprocessing)
