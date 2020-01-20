---
title: Scaling for Humans
date: '2017-09-16'
tags: ['readability']
categories: [data]
---

**Erik Meijer**[^1] in one of his lecture[^2] of the Functional Program Design in Scala course uses a so terrible example that I wanted to write it down. He used it to introduce reactive programming in Scala through the use of a `Future[T]` monad. But this is not the topic I want to talk about here. I just want to highlight, by reusing his example, the concept of **scaling for humans**.

The example taken is a chunk of code using *sockets* and so making calls over the network. The call to read data from memory and to send a packet over the network will not obviously take the same time. But we do not exactly know how much.

```scala
val packet = socket.readFromMemory()
// block for ...?
val confirmation = socket.sendToEurope(packet)
// block for ...?
```

To help us performing duration estimation, let's have a look to a table of approximate timing for various operations on a typical PC  expressed in nanoseconds[^4].

```
+-------------------------------------+----------+
| operation                           |   timing |
|-------------------------------------+----------|
| fetch from L1 cache memory          |      0.5 |
| branch misprediction                |        5 |
| fetch from L2 cache memory          |        7 |
| Mutex lock/unlock                   |       25 |
| fetch from main memory              |      100 |
| send 2K bytes over 1Gbps network    |    20000 |
| read 1MB sequentially from memory   |   250000 |
| fetch from new disk location (seek) |    8e+06 |
| read 1MB sequentially from disk     |    2e+07 |
| send packet US to Europe and back   |  1.5e+08 |
+-------------------------------------+----------+
```

First insight, but with time expressed in nanosecond (1/1,000,000,000 sec), it’s not easy to make the difference since **as humans we are not able to evaluate what each value represents quite well**. So well, the program will block a first time for 50,000 ns then, when sending a packet over the network, for 150,000,000 ns. It’s not a problem, no? So this does not help so much, at least immediately without an analysis.

```scala
val packet = socket.readFromMemory()
// block for 50,000 ns
val confirmation = socket.sendToEurope(packet)
// block for 150,000,000 ns
```

To realize what it means let’s **translate from computer scale (nanoseconds) to human scale (seconds)** by applying a straightforward rule.

1 nanosecond => 1 second

Now we have to represent the time according to proper unit we use all the day long: seconds, minutes, hours, days, months and years. I will do that in Python by simply using the humanize.naturaltime function to convert values stored in a DataFrame. It’s part of package I’ve already used in a previous article[^3]: **humanize**.

```python
# True is for future=True
df['timing'] = df['timing'].apply(humanize.naturaltime, args=(True,))

+-------------------------------------+---------------------+
| operation                           | timing              |
|-------------------------------------+---------------------|
| fetch from L1 cache memory          | now                 |
| branch misprediction                | 5 seconds from now  |
| fetch from L2 cache memory          | 7 seconds from now  |
| Mutex lock/unlock                   | 25 seconds from now |
| fetch from main memory              | a minute from now   |
| send 2K bytes over 1Gbps network    | 5 hours from now    |
| read 1MB sequentially from memory   | 2 days from now     |
| fetch from new disk location (seek) | 3 months from now   |
| read 1MB sequentially from disk     | 7 months from now   |
| send packet US to Europe and back   | 4 years from now    |
+-------------------------------------+---------------------+
```

It’s pretty clear now! **We can visualize immediately--without any computation--the difference between the time required for each operation**. Thus, by seeing that the code will block for **something like 3 days and then 5 years in human time** we immediately realize that **it will last a very long time at computer scale** and that we should better do something to handle it properly.

```scala
val packet = socket.readFromMemory()
// block for 3 days
val confirmation = socket.sendToEurope(packet)
// block for 5 years
```

[^1]: [Erik Meijer (computer scientist) - Wikipedia](https://en.wikipedia.org/wiki/Erik_Meijer_(computer_scientist))
[^2]: [Lecture 4.4 - Latency as an Effect 1 - École Polytechnique Fédérale de Lausanne | Coursera](https://www.coursera.org/learn/progfun2/lecture/D569Q/lecture-4-4-latency-as-an-effect-1)
[^3]: [The Ehrenberg’s rule](https://www.back2code.me/2017/08/the-ehrenbergs-rule/)
[^4]: [Teach Yourself Programming in Ten Years](http://norvig.com/21-days.html)