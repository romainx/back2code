---
title: Spark on Kubernetes Python and R bindings
date: '2018-12-28'
tags: [data]
---

The version `2.4` of Spark for Kubernetes introduces **Python and R bindings**.

- `spark-py`: The Spark image with Python bindings (including Python 2 and 3 executables)
- `spark-r`: The Spark image with R bindings (including R executable)

Databricks has published an [article][lk-1] dedicated to the Spark `2.4` features for Kubernetes. 

It's exactly the same principle as already explained in my [previous article][lk-2]. But this time we are using:

- A different image: `$MY_IP:5000/spark-py`
- Another example: `local:///opt/spark/examples/src/main/python/pi.py`, once again a Pi computation :-|
- A dedicated Spark **namespace**: `spark.kubernetes.namespace=spark-namespace`

The namespace must be created first:

```bash
$ k create namespace spark-namespace
namespace "spark-namespace" created
```

It permits to isolate the Spark pods from the rest of the cluster and could be used later to **cap available resources**.

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
    --conf spark.kubernetes.namespace=spark-namespace \
    local:///opt/spark/examples/src/main/python/pi.py
```

`spark.kubernetes.pyspark.pythonVersion` is an additional (an optional) property that can be used to select the major Python version to use (it's `2` by default).

> This sets the major Python version of the docker image used to run the driver and executor containers. Can either be 2 or 3.

# Labels

An interesting that has nothing to do with Python is that Spark defines **labels** that are applied on pods. They permit to easily identify the role of each pod.

```bash
$ k get po -L spark-app-selector,spark-role -n spark-namespace

NAME                            READY     STATUS              RESTARTS   AGE       SPARK-APP-SELECTOR                       SPARK-ROLE
spark-pi-1545987715677-driver   1/1       Running             0          12s       spark-c4e28a2ef3d14cfda16c007383318c79   driver
spark-pi-1545987715677-exec-1   0/1       ContainerCreating   0          1s        spark-application-1545987726694          executor
spark-pi-1545987715677-exec-2   0/1       ContainerCreating   0          1s        spark-application-1545987726694          executor
```

You can for example use the label to delete all the terminated driver pods.

```bash
#  Can also decide to switch from the default to the spark namespace
#$ k config set-context $(kubectl config current- context) --namespace spark-namespace
$ k delete po -l spark-role=driver -n spark-namespace
# Or you can delete the whole namespace
$ k delete ns spark-namespace
```

[lk-1]: https://databricks.com/blog/2018/09/26/whats-new-for-apache-spark-on-kubernetes-in-the-upcoming-apache-spark-2-4-release.html
[lk-2]: {{< ref "/post/2018/spark-on-k8s-first-run.md" >}}