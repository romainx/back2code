---
title: Using data in Spark Standalone Cluster
date: '2018-11-29'
tags: []
draft: true
---



```bash
#
$ mkdir -p /tmp/data
# Creating a symlink from the container /data to the local filesystem
$ ln -s /Users/to145733/Git/docker-spark/data/flights.parquet /tmp/data/flights.parquet
$ ln -s /Users/to145733/Git/docker-spark/data /tmp/data
```
