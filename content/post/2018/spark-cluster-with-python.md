---
title: Configure PySpark to connect to a Standalone Spark Cluster
date: '2018-11-29'
categories: [data]
tags: ['spark', 'python']
---

In one of my [previous article][LK-1] I talked about running a Standalone Spark Cluster inside Docker containers through the usage of [docker-spark][LK-2]. I was using it with R Sparklyr framework. 

However if you want to use from a **Python environment** in an interactive mode (like in Jupyter notebooks where the driver runs on the local machine while the workers run in the cluster), you have several steps to follow.

* You need to run the same Python version on the driver and on the workers.
* You need to configure PySpark to use the same Spark version, see how to use findspark in my [article][LK-3].

# Use the same Python version

The **docker-spark** containers run a Python `3.5` version. If you want to interact with it with from an external Jupyter notebook running on your machine you have to run a Kernel with the same version.

```bash
# With a python version
$ conda create -n py35 python=3.5
# Or with anaconda
$ conda create -n py35 python=3.5 anaconda
# Activate the newly created environment
$ source activate py35
# Register the kernel
$ python -m ipykernel install --user --name py35 --display-name "Python 3.5"
# You can check installed kernels if you are curious
# jupyter kernelspec list
```

You can now choose your "Python 3.5" Kernel to run PySpark.
If you use another version say `3.7` for example you will see an explicit error.

```
Exception: Python in worker has different version 3.5 than that in driver 3.7, PySpark cannot run with different minor versions.Please check environment variables PYSPARK_PYTHON and PYSPARK_DRIVER_PYTHON are correctly set.
```

There is another step to follow.
You have to specify the path of the python executable to use in the worker. Without this setting it will try to use the same as in the driver so something like `/Users/xxxx/anaconda3/envs/py35/bin/python`. And you will get this kind of error in the logs.

```
java.io.IOException: Cannot run program "/Users/xxxx/anaconda3/envs/py35/bin/python": error=2, No such file or directory
```

In the `Dockerfile` of docker-spark we can see that the python environment is available unuder `/usr/bin/python`

```bash
RUN apt-get update \
 && apt-get install -y curl unzip \
    python3 python3-setuptools \
 && ln -s /usr/bin/python3 /usr/bin/python \
```

So we have to specify it through the environment variable `PYSPARK_PYTHON`. See [here][LK-4] for all the available environment variables.

```python
import os
os.environ['PYSPARK_PYTHON'] = '/usr/bin/python'
```

# Putting it all together

That's it!

```python
import os

# Make sure you call it before importing pyspark
import findspark
findspark.init('/Users/xxxx/spark/spark-2.3.1-bin-hadoop2.7')

# Defining the python version to use on the workers
os.environ['PYSPARK_PYTHON'] = '/usr/bin/python'

# Pyspark imports
import pyspark
from pyspark.sql import SparkSession

# For pi test
import random

spark = SparkSession.builder.master('spark://localhost:7077').appName('spark-cluster').getOrCreate()

NUM_SAMPLES = 10

def inside(p):
    x, y = random.random(), random.random()
    return x*x + y*y < 1

count = spark.sparkContext.parallelize(range(0, NUM_SAMPLES)) \
             .filter(inside).count()

print('Pi is roughly {}'.format(4.0 * count / NUM_SAMPLES))

# Pi is roughly 4.0
```

[LK-1]: {{< ref "configure-sparklyr-to-connect-to-a-standalone-spark-cluster.md" >}}
[LK-2]: https://github.com/gettyimages/docker-spark
[LK-3]: {{< ref "find-spark.md" >}}
[LK-4]: http://spark.apache.org/docs/latest/configuration.html#environment-variables