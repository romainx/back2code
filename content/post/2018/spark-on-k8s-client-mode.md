---
title: Spark on Kubernetes Client Mode
date: '2018-12-29'
categories: [data]
tags: ['spark', 'kubernetes']
---

This is the third article in the Spark on Kubernetes (K8S) series after:

- [Spark on Kubernetes First][lk-2]
- [Spark on Kubernetes Python and R bindings][lk-3]

This one is dedicated to the **client mode** a feature that as been introduced in Spark `2.4`.
In client mode the driver runs locally (or on an external pod) making possible **interactive mode** and so it cannot be used to run REPL like Spark shell or Jupyter notebooks.

# spark-submit

To activate the client the first thing to do is to change the property `--deploy-mode client` (instead of `cluster`).

In client mode the file to execute is provided by the driver. So I'm using `file` instead of `local` at the start of the URI.
The [`spark-submit`][lk-1] documentation gives the reason

> - **file**: Absolute paths and `file:/` URIs are served by the driver’s HTTP file server, and every executor pulls the file from the driver HTTP server.
> - **local**: a URI starting with `local:/` is expected to exist as a local file on each worker node. This means that no network IO will be incurred, and works well for large files/JARs that are pushed to each worker, or shared via NFS, GlusterFS, etc.

```bash
# Do not forget to create the spark namespace, it's handy to isolate Spark resources
$ k create namespace spark
$ cd $SPARK_HOME
$ ./bin/spark-submit \
    --master k8s://https://localhost:6443 \
    --deploy-mode client \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=2 \
    --conf spark.driver.memory=512m \
    --conf spark.executor.memory=512m \
    --conf spark.kubernetes.container.image=spark:v2.4.0 \
    --conf spark.kubernetes.namespace=spark \
    file://$SPARK_HOME/examples/jars/spark-examples_2.11-2.4.0.jar
```

This time there is no more pod for the driver.

```bash
$ k get po -n spark

# NAME                            READY     STATUS    RESTARTS   AGE
# spark-pi-1546030938784-exec-1   1/1       Running   0          4s
# spark-pi-1546030939189-exec-2   1/1       Running   0          4s
```

The result can be seen directly in the console.

```bash
18/12/28 22:02:27 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 1.433958 s
Pi is roughly 3.137875689378447
```

## spark-shell

Now let's try something more interactive.

```bash
$ cd $SPARK_HOME
$ ./bin/spark-shell \
    --master k8s://https://localhost:6443 \
    --deploy-mode client \
    --name spark-shell \
    --driver-memory 512M \
    --executor-memory 512M \
    --conf spark.executor.instances=2 \
    --conf spark.kubernetes.container.image=spark:v2.4.0 \
    --conf spark.kubernetes.namespace=spark
```

And it works!

```
Spark context available as 'sc' (master = k8s://https://localhost:6443, app id = spark-application-1546031781274).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.0
      /_/

Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_181)
Type in expressions to have them evaluated.
Type :help for more information.
```

We can see the 2 executors running.

```bash
$ k get po -l spark-app-selector=spark-application-1546031781274 -n spark

# NAME                               READY     STATUS    RESTARTS   AGE
# spark-shell-1546031781314-exec-1   1/1       Running   0          4m
# spark-shell-1546031781735-exec-2   1/1       Running   0          4m
```

Now let's try a simple example with an RDD.

```scala
scala> val data = 1 to 10000
// data: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, ...

scala> val distData = sc.parallelize(data)
// distData: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:26

scala> distData.filter(_ < 10).collect()
// res0: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)
```

We can check in the executor logs

```bash
$ k logs -l spark-app-selector=spark-application-1546031781274 -n spark

# 2018-12-28 21:27:22 INFO  Executor:54 - Finished task 1.0 in stage 0.0 (TID 1). 734 bytes result sent to driver
```

And also in the Spark UI without the need to forward a port since the driver runs locally, so you can reach it at http://localhost:4040/.

![spark ui](/post/2018/spark-on-k8s-client-mode_files/spark-shell.png)

At the end of the shell, the executors are terminated. Cool!

[lk-1]: http://spark.apache.org/docs/latest/submitting-applications.html#launching-applications-with-spark-submit
[lk-2]: {{< ref "/post/2018/spark-on-k8s-first-run.md" >}}
[lk-3]: {{< ref "/post/2018/spark-on-k8s-bindings.md" >}}