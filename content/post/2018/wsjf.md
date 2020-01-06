---
title: WSJF
date: '2018-11-27'
categories: [dev]
tags: ['agile']
---

**WSJF** stands for Weighted Shortest Job First. It’s a technique used in scaled Agile framework (SAFe) to prioritise jobs—epics, features and capabilities—according to their value relative to the cost to perform it. Basically it’s **a way of ranking a list of features in order to maximise the outcome—the value produced—with a constrained capacity to produce it**. The job with the highest WSJF (value over the cost) is selected first for implementation.

It’s not a brand new idea, long ago a very similar technique known as **Value Engineering** (VE) representing the ratio of function to the cost was used. 

> Value engineering (VE) is a systematic method to improve the "value" of goods or products and services by using an examination of function. Value, as defined, is **the ratio of function to cost**. Value can therefore be manipulated by either improving the function or reducing the cost. 

# Computing the WSJF

## Estimating the value

In SAFe we are talking about the **Cost of Delay** (CoD). 3 elements contribute to it that can be estimated using the modified Fibonacci numbers (1,2,3,5,8,13,20)

* **Business value**: An estimation of the value brought to the business i.e. the end users. 
* **Time criticality**: Will the feature value decrease over the time, meaning for example that if it is not done right now, it will not deliver value anymore.
* **Risk reduction opportunity**: How this feature contribute to reduce a risk

I see **several disadvantages** with this method:

* When a feature is not mature enough is not always easy to estimate 3 components (value, risk, time) when it’s already hard to estimate a single one. This ends in putting dumb values in the tables.
* Each of these values does not have any meaning in a global (absolute) scale—what does a time criticality of 13 mean? They only make sense as relative measures, so you have to define a referential and define the values by comparing them to each others. It could be a tedious task particularly when you try to compare ratings across different teams
* The two previous issues may lead to produce inaccurate estimation of the value. The value being by definition 3 times greater than the duration (the cost). So if the estimation is not accurate you end with a useless result

## Estimating the duration

Instead of using duration which is less straightforward to compute, the job size can be used—even if team capacity is not the same, duration is proportional to the size. Since size is always estimated in story points it can be used as is.

## Computing the ratio

Now the simplest part of the method putting all together and sort the result.
For fun I’ve made the example with some Python code.

```python
import random
from random import sample
import pandas as pd
from tabulate import tabulate

random.seed(3)

# Modified Fibonacci sequence
fib = [1, 2, 3, 5, 8, 13, 20]

# Generating sample data for the Cost of Delay
df = pd.DataFrame({'feature': ['A', 'B', 'C'],
                   'b_value': sample(fib, 3), 
                   'risk': sample(fib, 3), 
                   'time': sample(fib, 3)})

# Computing the Cost of Delay (CoD)
df['cod'] = df['b_value'] + df['risk'] + df['time'] 

# Generation sample data for the duration
df['duration'] = sample(fib, 3)

# Computing the Weight
df['weight'] = df['cod'] / df['duration']

# Implementing the featuer with the higher weight
df.sort_values('weight', ascending=False, inplace=True)

print(tabulate(df, headers='keys', showindex=False))
```
And the result is that you should implement the feature "A" first!

```
feature      b_value    risk    time    cod    duration    weight
---------  ---------  ------  ------  -----  ----------  --------
A                  2       2       5      9           1      9
B                  8       3      13     24           8      3
C                 13       8       8     29          20      1.45
```

# References / Further reading

* [Value engineering - Wikipedia](https://en.wikipedia.org/wiki/Value_engineering)
* [WSJF – Scaled Agile Framework](https://www.scaledagileframework.com/wsjf/)
* [Using Weighted Shortest Job First (WSJF) to prioritize your backlog and improve ROI](https://techbeacon.com/prioritize-your-backlog-weighted-shortest-job-first-wsjf-improved-roi)
* [Ranking by SAFe WSJF - VersionOne Community](https://community.versionone.com/VersionOne-Lifecycle/Product_Planner/Backlog_Inputs/Ranking_Work_Items/Ranking_by_SAFe_WSJF)