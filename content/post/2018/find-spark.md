---
title: Find Spark
date: '2018-11-28'
categories: [data]
tags: ['spark', 'python']
---

**[Find Spark][LK-1]** is an handy tool to use each time you want **to switch between spark versions** in Jupyter Notebooks without the need to change the `SPARK_HOME` environment variable. It works by:

> adding `pyspark` to `sys.path` at runtime.

Note: You need to restart the *Kernel* in order to change the Spark version.

Install it.

```bash
$ pip install findspark
```

Use it.

```python
# Make sure you call it before importing pyspark
import findspark
# Without parameter it will use the SPARK_HOME variable to perform the init
findspark.init('/Users/xxxx/spark/spark-2.3.1-bin-hadoop2.7')

# It will import the corresponding version (2.3.1 in this case)
import pyspark
from pyspark.sql import SparkSession

spark = SparkSession.builder.master('local').appName('spark-local').getOrCreate()

f'Using Spark {spark.version} from {findspark.find()}'

# 'Using Spark 2.3.1 from /Users/xxxx/spark/spark-2.3.1-bin-hadoop2.7'
```

[LK-1]: https://github.com/minrk/findspark