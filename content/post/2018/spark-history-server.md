---
title: Spark History Server available in docker-spark
date: '2018-11-29'
tags: [data]
---

Spark comes with a **history server**, it provides a great UI with many information regarding Spark jobs execution (event timeline, detail of stages, etc.).
Details can be found in the [Spark monitoring page][spark-monit].

I've modified the [docker-spark](https://github.com/romainx/docker-spark) to be able to run it with the `docker-compose up`command.

With this implementation, its UI will be running at `http://${YOUR_DOCKER_HOST}:18080`.

![history server UI](/post/spark-history-server_files/history-server.png)

To use the Spark’s history server you have to tell your Spark driver:

* to log events: `spark.eventLog.enabled true` (it's `false` by default)
* the log directory to use: `spark.eventLog.dir file:/tmp/spark-events`

By default the `/tmp/spark-events` is mounted on the `./spark-events` at the root of the repo (I call it `$DOCKER_SPARK`).
So you have to tell the driver to log events in this directory (on your local machine).

This example shows this configuration for a `spark-submit` (the two `--conf` options): 

    DOCKER_SPARK="/Users/xxxx/Git/docker-spark"

    $SPARK_HOME/bin/spark-submit \
      --class org.apache.spark.examples.SparkPi \
      --master spark://localhost:7077 \
      --conf "spark.eventLog.enabled=true" \
      --conf "spark.eventLog.dir=file:$DOCKER_SPARK/spark-events" \
      $SPARK_HOME/examples/jars/spark-examples_2.11-2.3.1.jar \
      10

*Note: This settings can be defined in the driver's `$SPARK_HOME/conf/spark-defaults.conf` to avoid using the `--conf` option.*

[spark-monit]: https://spark.apache.org/docs/latest/monitoring.html