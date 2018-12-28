---
title: Spark on Kubernetes Python and R bindings
date: '2018-12-27'
tags: [data]
---

The version `2.4` of Spark for Kubernetes introduces **Python and R bindings**.

- `spark-py`: The Spark image with Python bindings (including Python 2 and 3 executables)
- `spark-r`: The Spark image with R bindings (including R executable)

Databricks has published an [article][lk-1] dedicated to the Spark `2.4` features for Kubernetes. 

It's exactly the same principle as already explained in my [previous article][lk-2]. But this time we are using:

- A different image: `$MY_IP:5000/spark-py`
- Another example: `local:///opt/spark/examples/src/main/python/pi.py`, once again a Pi computation :-|

```bash
$ cd $SPARK_HOME
$ ./bin/spark-submit \
    --master k8s://https://localhost:6443 \
    --deploy-mode cluster \
    --name spark-pi \
    --conf spark.executor.instances=2 \
    --conf spark.driver.memory=512m \
    --conf spark.executor.memory=512m \
    --conf spark.kubernetes.container.image=$MY_IP:5000/spark-py \
    --conf spark.kubernetes.pyspark.pythonVersion=3 \
    local:///opt/spark/examples/src/main/python/pi.py
```

`spark.kubernetes.pyspark.pythonVersion` is an additional (an optional) property that can be used to select the major Python version to use (it's `2` by default).

> This sets the major Python version of the docker image used to run the driver and executor containers. Can either be 2 or 3.

An interesting that has nothing to do with Python is that Spark defines **labels** that are applied on pods. They permit to easily identify the role of each pod.

```bash
k get po -L spark-app-selector,spark-role

NAME                            READY     STATUS              RESTARTS   AGE       SPARK-APP-SELECTOR                       SPARK-ROLE
spark-pi-1545985444398-driver   1/1       Running             0          11s       spark-344f13ea0f904c27a3cb510d5590ba80   driver
spark-pi-1545985444398-exec-1   0/1       ContainerCreating   0          0s        spark-application-1545985455669          executor
spark-pi-1545985444398-exec-2   0/1       ContainerCreating   0          0s        spark-application-1545985455669          executor
```
[lk-1]: https://databricks.com/blog/2018/09/26/whats-new-for-apache-spark-on-kubernetes-in-the-upcoming-apache-spark-2-4-release.html
[lk-2]: {{< ref "/post/2018/spark-on-k8s-first-run.md" >}}