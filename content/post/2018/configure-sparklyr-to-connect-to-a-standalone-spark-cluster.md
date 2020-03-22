---
title: 'Configure Sparklyr to connect to a Standalone Spark Cluster'
date: '2018-11-28'
categories: [data]
tags: ['spark','R']
---

# Run a Standalone Spark Cluster

One of the simplest way to run a standalone spark cluster is to use
Docker!

*   [GitHub - gettyimages/docker-spark: Docker build for Apache
    Spark](https://github.com/gettyimages/docker-spark)

```bash
# Clone the repo
$ git clone https://github.com/gettyimages/docker-spark.git
$ cd docker-spark
# Launch The Spark Cluster
$ docker-compose up
# You should see this in the log output
# INFO  Worker:54 - Running Spark version 2.3.1
# ...
# INFO  Worker:54 - Successfully registered with master spark://master:7077
```

Then the Spark UI is reachable at: `http://localhost:8080/`, Great !
And the master is available at: `spark://localhost:7077`. You have
to grab the Spark version from here (2.3.1 in my example).

To see how it works have a look at the `docker-compose.yml` definition
file.

## Note

> By default, it \[an application\] will acquire all cores in the
> cluster, which only makes sense if you just run one application at a
> time.

To put it another way you will only be able to run only one application

You can change this behavior in the `conf/master/spark-defaults.conf` to
tell the master to allocate 1 core by default.

```bash
spark.deploy.defaultCores 1
```

# Using Spark with Sparklyr

In order to be able to connect to the cluster you need to have the Spark
libraries installed locally (i.e. where you R code will run). In our
example the Spark cluster runs in Docker containers so the libraries are
not available in the filesystem.

```r
# install.packages("sparklyr")
spark_v <- "2.3.1"
library(sparklyr)
cat("Installing Spark in the directory:", spark_install_dir())
## Installing Spark in the directory: /Users/xxxx/spark
spark_install(version = spark_v)
cat("Checking if the version is installed")
## Checking if the version is installed
spark_installed_versions()
##   spark hadoop                                               dir
## 1 2.3.1    2.7 /Users/xxxx/spark/spark-2.3.1-bin-hadoop2.7
```

Then you can simple make connection to the Spark cluster.

```r
sc <- spark_connect(spark_home = spark_install_find(version=spark_v)$sparkVersionDir, 
                    master = "spark://localhost:7077")
sc$master
## [1] "spark://localhost:7077"
```

And run one of the example given in the [Home page of
Sparklyr](https://spark.rstudio.com/).

```r
library(dplyr)
#install.packages(c("nycflights13", "Lahman"))

flights_tbl <- copy_to(sc, nycflights13::flights, "flights")
flights_tbl %>% filter(dep_delay == 2)

## # Source: spark<?> [?? x 19]
##     year month   day dep_time sched_dep_time dep_delay arr_time
##  * <int> <int> <int>    <int>          <int>     <dbl>    <int>
##  1  2013     1     1      517            515         2      830
##  2  2013     1     1      542            540         2      923
##  3  2013     1     1      702            700         2     1058
##  4  2013     1     1      715            713         2      911
##  5  2013     1     1      752            750         2     1025
##  6  2013     1     1      917            915         2     1206
##  7  2013     1     1      932            930         2     1219
##  8  2013     1     1     1028           1026         2     1350
##  9  2013     1     1     1042           1040         2     1325
## 10  2013     1     1     1231           1229         2     1523
## # ... with more rows, and 12 more variables: sched_arr_time <int>,
## #   arr_delay <dbl>, carrier <chr>, flight <int>, tailnum <chr>,
## #   origin <chr>, dest <chr>, air_time <dbl>, distance <dbl>, hour <dbl>,
## #   minute <dbl>, time_hour <dttm>
```

Once done you can stop the Spark context.

```r
spark_disconnect(sc)
```

# References


* [Home page of Sparklyr](https://spark.rstudio.com/)
* [Master URL Scheme](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)
* [Question on SO](https://stackoverflow.com/questions/39798798/connect-sparklyr-to-remote-spark-connection)
