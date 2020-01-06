---
title: Rack Awareness
date: '2016-11-18'
categories: [data, ops]
---

**Hadoop** is a distributed system, so its core services[^1] have to know the network topology — in short where data nodes reside in the data center — either to ensure reliability (try to write replicas of a block at different location for fault tolerance) and performance (try[^2] to read the closest blocks).
**HDFS** (the *NameNode*) uses this information during its writing process in order to choose the destination of each block replica[^3]. To find the good tradeoff between reliability and performance it tries to use this kind of strategy:

1. The first replica is written on the same node as the client
2. The second replica is written on a different rack (*off-rack*)
3. The third replica is written on the same rack as the second but on a different node

Then it uses this information to maximise the performance during read phases by trying to read the closest blocks.
**YARN** (the *Resource Manager*) uses this information to maximise performance. It tries to limit data transfer (answer to locality constraint) by computing data on a node hosting a replica or on a node close to a node hosting the replica — i.e. on the same rack.

We talk about awareness of network topology and distance between nodes, but **how do Hadoop components know or compute this distance**? The simplest solution is often the best. The network is represented as a tree. The distance between two nodes is the sum of their distance to their closest common ancestor (0 when they are on the same node)[^4]. Levels are not defined — this means that the algorithm is agnostic — but the convention is to use the rack as depicted below (a data center level can be used but usually a cluster nodes reside on the same data center)

```
|-- Rack 1
|   |-- Node 1
|   |-- Node 2
|-- Rack 2
|   |-- Node 3
```

* Same node: `distance = 0`
* Same rack (node 1 & 2): `distance = 2`
* Different rack in the same data center (node 1 & 3): `distance = 4`

This information is obtained either by an external script or java class[^5]. Ambari can also take care of this. The administrator has just to define the rack ID of each node and Ambari generates and deploy the corresponding configuration — once done it is also possible to browse a *heatmap* by rack ID[^6].

[^1]: Core services are the file system HDFS and the resources (CPU and memory) manager YARN.
[^2]: I often use the verb “try” because finding the best tradeoff is not always possible according to other constraints like nodes too full or too busy.
[^3]: The typical replication factor is 3 meaning that for each logical block, 3 blocks are written on the Cluster
[^4]: Tom White, *[Hadoop: The Definitive Guide](https://www.goodreads.com/book/show/25000038-hadoop) (O'Reilly, 2009)*
[^5]: [Apache Hadoop Rack Awareness](https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/RackAwareness.html)
[^6]: [HDP Rack Awareness] (https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.0/bk_Ambari_Users_Guide/content/ch03s11.html)
