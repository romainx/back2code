---
title: Monitoring YARN resources allocation through its REST API
date: '2017-09-02'
tags: ['hadoop']
categories: [ops]
---

This article explains how to put in place quickly a basic monitoring of the **Hadoop YARN resource allocation system** through the ResourceManager REST API’s[^1]. The same information is also available in metrics collectors like Ambari metrics (AMS), but YARN API is really easy to use and is available everywhere — and most of the time without requiring authentication. A last word, triggers (thresholds) on these metrics make efficient source to integrate into a global monitoring and alerting system, like Nagios or Zabbix, since they are not available out of the box in the standard Ambari alerting system[^2]. Now let's go!

# Capacity metrics

YARN is sometimes called the **operating system of data**. And it's a good analogy. Its role is mainly to fulfill allocation requests (for *VCore* and RAM) and deliver them through container allocations. So, like for any operating system, it's important to monitor the amount of available resources.

* **Memory used**: The percentage of available YARN memory currently allocated.
* **VCore used**: The percentage of VCore available currently allocated.

These values are often expressed as percentages, they are easy to understand and moreover they can be easily integrated in any monitoring system by using standard thresholds. For example 70% of usage throws a warning while 90% of usage is a critical event. Moreover it's worth doing it some information is not immediately available in YARN UI or in the CLI tool yarn top that instead display absolute values.

These metrics can be easily obtained by using the end point `http://<rm http address:port>/ws/v1/cluster/metrics`.

```json
"clusterMetrics": {
  "availableMB":17408,
  "totalMB":17408,
  "allocatedMB":0,
  "availableVirtualCores":7,
  "totalVirtualCores":8,
  "appsRunning":0,
  "appsPending":0,
  "containersReserved":0,
  "totalNodes":1
  [...]
}
```

* `memory_used = (1 - availableMB / totalMB) x 100`
* `vcore_used = (1 - availableVirtualCores / totalVirtualCores) x 100`

# Scheduler policy

It's also a good practice to compare **the queue usage** to the **resources usage** in order to check **if the scheduler policy and the queue definitions are properly configured and uses efficiently the available resources on the cluster**. According to **the chosen scheduler** and the way it allocates resources, it's also useful to track the root queue usage — the usage of the cluster. For example the Hortonworks Data Platform (HDP) is configured to use by default a capacity scheduler based on memory (DefaultResourceCalculator) — and not on dominant resource fairness (DominantResourceCalculator).

This metric is for the same reasons better expressed as a percentage and easy to build by using the end point `http://<rm http address:port>/ws/v1/cluster/scheduler`.

```json
"schedulerInfo": {
    "queueName": "root"
    "capacity": 100.0,
    "usedCapacity": 10.0,
    "maxCapacity": 100.0
    [...]
}
```

* `root_queue_used = usedCapacity / capacity x 100`

# Monitoring the consequences

## Pending applications

The consequences of an overloaded cluster, **either due to a lack or resources or to a bad allocation policy**, is that applications cannot run as expected. In this case they are submitted but wait in **pending state** (ACCEPTED instead of RUNNING).

So it is important to track the **number of pending apps**. It can be either expressed as an absolute value or as a percentage -- I prefer the absolute value seems it seems more meaningful for me. Information are available through the end point `http://<rm http address:port>/ws/v1/cluster/metrics`.

* `pending apps = appsPending / appsRunning x 100`

It is also possible to get the same information for a dedicated queue through the scheduler end point.

## Reservations

Another indication of a problem of resource allocation is **the amount of reservations**. The scheduler has the ability to reserve resources when it cannot fulfill applications allocation requests. A high number means that it may take longer to run application since they are waiting for resources.

> The Capacity Scheduler is responsible for matching free resources in the cluster with the resource requirements of an application. Many times, a scheduling cycle occurs such that even though there are free resources on a node, they are not sized large enough to satisfy the application waiting for a resource at the head of the queue. [...] This mismatch can lead to starving these resource-intensive applications. [^3]

And the solution to this problem is the reservation.

> To prevent this unfortunate situation, when space on a node is offered to an application, if the application cannot immediately use it, it reserves it, and no other application can be allocated a container on that node until the reservation is fulfilled.[^4]

To turn the number of container reserved (containersReserved) into a percentage we can use the number of nodes (totalNodes) because of this.

> To prevent the number of reservations from growing in an unbounded manner, and to avoid any potential scheduling deadlocks, the Capacity Scheduler maintains only one active reservation at a time on each node.[^3]
And from the metrics end point we can simply define.

* `containers reserved = containersReserved / totalNodes) x 100`

We have also other derived information that can be exploited since it is possible to get the amount of reserved *VCore* (`reservedVirtualCores`) and Memory (`reservedMB`). But I think it is a good start for a first level of information easy to implement and to understand. We may study this point in a next post.

[^1]: [Apache Hadoop 2.7.4 - ResourceManager REST APIs](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/ResourceManagerRest.html)
[^2]: [YARN Alerts - Hortonworks Data Platform](https://docs.hortonworks.com/HDPDocuments/Ambari-2.5.1.0/bk_ambari-operations/content/yarn_alerts.html)
[^3]: [Application Reservations - Hortonworks Data Platform](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.3/bk_yarn-resource-management/content/application_reservations.html)
[^4]: [Improvements in the Hadoop YARN Fair Scheduler - Cloudera Engineering Blog](http://blog.cloudera.com/blog/2013/06/improvements-in-the-hadoop-yarn-fair-scheduler)