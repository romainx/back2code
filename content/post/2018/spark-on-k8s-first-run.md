---
title: Spark on Kubernetes first run
date: '2018-12-27'
tags: [data]
---

Since the version `2.3`, Spark can run on a Kubernetes cluster. Let's see how to do it.
In this example I will use the version `2.4`. Prerequisites are:

- [Download][lk-1] an install (unzip) the corresponding Spark distribution,
- Setup a local Kubernetes registry as explained in [my previous article][lk-2].

For more information, there is a [section][lk-3] on the Spark site dedicated to this use case.

# Spark images

I will build and push Spark images to make them available to the K8S cluster.

## Build

The Spark distribution comes with a script (`docker-image-tool.sh`) that permits to build Spark images. You might wonder why build your own image and not use an image already available on the hub? One of the main reason is to give the ability to customize it to match your Spark distribution.

Several options are available to define a repo and / or to push images but I will do it in two steps.

```bash
# Building the images
$ cd $SPARK_HOME
$ ./bin/docker-image-tool.sh -t v2.4.0
# Listing the images
$ spark-2.4.0-bin-hadoop2.7 docker images | grep spark

spark-r   v2.4.0  7257138f9086  38 hours ago  764MB
spark-py  v2.4.0  fbc77732ab07  38 hours ago  438MB
spark     v2.4.0  d952a2b3506f  38 hours ago  348MB
```

The 3 images built

- `spark`: The standard Spark image
- `spark-py`: The Spark image with Python bindings (including Python 2 and 3 executables)
- `spark-r`: The Spark image with R bindings (including R executable)

## Push

I'm using here the same principle I've already explained in [my previous article][lk-2].

```bash
$ MY_IP="$(ipconfig getifaddr en0)"
# Run registry
$ docker run -d -p 5000:5000 --name registry registry:2
# Tag the image with the registry information and the latest tag
$ docker image tag spark:v2.4.0 $MY_IP:5000/spark
# Push
$ docker push $MY_IP:5000/spark
# List the content
$ curl -X GET http://$MY_IP:5000/v2/_catalog

"repositories":["spark"]}
```

# Run

In this first example, I will run a Spark job in **cluster mode** (the driver runs on the Cluster). This is a non interactive mode and so it cannot be used to run Spark shell or Jupyter notebooks. This is the first mode that has been made available for Spark on K8S.

## Master

In the following example the magic line is `--master k8s://https://localhost:6443`. This means that `spark-submit` will interact with the K8S API server.

```bash
$ k cluster-info
Kubernetes master is running at https://localhost:6443
```

## Sizing params

In the conf, I've changed the default settings to restrict the amount of memory (512 MB) that will be requested for the driver and the two workers (additionaly they will require 1 CPU each so 3 CPUs).

```bash
--conf spark.executor.instances=2 \
--conf spark.driver.memory=512m \
--conf spark.executor.memory=512m \
```

This sizing has to be checked since it can exceed the allocatable memory.

```bash
2018-12-26 07:47:39 WARN  TaskSchedulerImpl:66 - Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources
```

The allocatable memory can be checked here 3 GB and 5 CPUs.

```bash
# There is only one node in the K8S shipped with Docker
k get node  --output=json | jq ".items[0].status.allocatable"

{
  "cpu": "5",
  "ephemeral-storage": "56453061334",
  "hugepages-1Gi": "0",
  "hugepages-2Mi": "0",
  "memory": "2973324Ki",
  "pods": "110"
}
```

## Running params

### Spark image

For the Spark image I'm using the one that has been pushed to the repository.

```bash
--conf spark.kubernetes.container.image=$MY_IP:5000/spark
``` 

### Executable

We can see in the `Dockerfile` that the examples are available in `/opt/spark/examples`. 

```Dockerfile
COPY examples /opt/spark/examples
```

So I can specify the `jar` path by using this location `local:///opt/spark/examples/jars/spark-examples_2.11-2.4.0.jar` where Spark will be able to find the class to run `org.apache.spark.examples.SparkPi`.

## Putting it all together

And finally here is the command to run.

```bash
$ cd $SPARK_HOME
$ ./bin/spark-submit \
    --master k8s://https://localhost:6443 \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=2 \
    --conf spark.driver.memory=512m \
    --conf spark.executor.memory=512m \
    --conf spark.kubernetes.container.image=$MY_IP:5000/spark \
    local:///opt/spark/examples/jars/spark-examples_2.11-2.4.0.jar
```

Once started you can check the pods running (the driver and the two executors).

```bash
$ k get pods

NAME                          READY STATUS             RESTARTS  AGE
spark-pi-1545849731069-driver 1/1   Running            0         10s
spark-pi-1545849731069-exec-1 0/1   ContainerCreating  0         0s
spark-pi-1545849731069-exec-2 0/1   ContainerCreating  0         0s
```

At the end, once again, Pi is *roughly* computed :-).

```bash
$ k logs spark-pi-1545849731069-driver

2018-12-26 15:51:49 INFO  DAGScheduler:54 - Job 0 finished: reduce at SparkPi.scala:38, took 1.751761 s
Pi is roughly 3.1380156900784506
```

Note that you can access to the Driver UI by forwarding the `4040` port.

```bash
$ k port-forward <driver-pod-name> 4040:4040
# So, in this example
$ k port-forward spark-pi-1545849731069-driver 4040:4040
```

[lk-1]: https://spark.apache.org/downloads.html
[lk-2]: {{< ref "/post/2018/docker-local-registry.md" >}}
[lk-3]: https://spark.apache.org/docs/latest/running-on-kubernetes.html