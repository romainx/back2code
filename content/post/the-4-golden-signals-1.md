---
title: The 4 Golden Signals + 1
date: '2018-01-03'
tags: [ops]
---

The term **4 golden signals** has been introduced by **Google SRE team** in the book *Site Reliability Engineering*[^1]. The main definitions presented below are borrowed from this book.

> The four golden signals of monitoring are latency, traffic, errors, and saturation. If you can only measure four metrics of your user-facing system, focus on these four.

# 1 - Latency (Performance)

> The time it takes to service a request, with a focus on distinguishing between the latency of successful requests and the latency of failed requests.[^1]

I often call it **performance** since it sounds more natural for most people. The distinction has to be made between performance of **successful and failed requests**.

* The first reason is obvious, you want to know **the performance of the service when it delivers correct answers** without being distorted by the performance of failed queries--which you should expect to be faster, see next point.
* The second is less obvious, since measuring the **performance of failed queries (errors)** is not the first thing that comes in mind. But if a user has to wait a lot for a failed query is the double penalty. It’s one of the reason why system have to comply with the **fail fast stability pattern** described by **Michael T. Nygard** in his book *Release It!*[^2]. The second reason is that the system should avoid to consume resources to end with a failure.

The best way to express is the **time needed to perform an operation** (query, run a job, etc.). For this kind of measures you should use **percentiles** (P9X for example) since values like average are often not representative.

# 2 - Traffic (Usage)

> A measure of how much demand is being placed on the service. This is measured using a high-level service-specific metric, like HTTP requests per second in the case of an HTTP REST API.[^1]

I also call it **usage**. There is not many additional things to say, except that it’s important to be able to measure the usage of a service. It can be an important clue when you have to analyse problems since they have a tendency to increase depending on the usage. It’s often expressed as **a number of operations per unit of time**--human units like second, minutes and so on are always a best choice since this is easier to figure out the amount of traffic with this kind of units than with smaller ones (ms for example).

# 3 - Errors (Success rate)

> The rate of requests that fail. The failures can be explicit (e.g., `HTTP 500` errors) or implicit (e.g., an `HTTP 200 OK` response with a response body having too few items).[^1]

I prefer using the inverse measure that I call **success rate**. It measures the rate of error--or success--of an operation. I think it’s more natural to put threshold saying like 99% of operations shall succeed than we allow 1% of error. **This is a key metric**--I would say the most important. It’s here to **warn that something bad is happening**. It’s expressed **as a percentage of errors over the total of operations**--or success over the total if you are positive like me.

# 4 - Saturation (Capacity)

> How “full” is the service. This is a measure of the system utilization, emphasizing the resources that are most constrained (e.g., memory, I/O or CPU). Services degrade in performance as they approach high saturation.[^1]

I prefer the term **capacity** since it is more widely used. It can be simple to use when talking about finite resources like the number of CPU or an amount of memory. In this case you just have to measure **the amount of resources used over the total available resources and express it has a percentage** (70% of memory used for example). However in some cases-- like the number of connections to a database--it could be trickier to determine the amount of resources used and available. Like Michael T. Nygard emphasis in the chapter “Capacity” of his book[^2], it’s important to **identify the most constraint metric** since--it’s obvious but worth to remind

> Improving the nonconstraint metrics will not improve capacity.

# 5 - Availability

The additional mandatory metrics is **availability**. Everyone will always inquire about availability. Availability can be computed in various ways from incidents duration to formulas built from other metrics. My advice is to track it through **dedicated availability tests**. Availability tests are a kind of **smoke tests** from a simple test (perform a connection to a database) to more complex tests involving several operations **performed in black-box mode** (Testing externally visible behaviour as a user would see it). **Start simple and improve it** them each time the test is not representative of the availability. In this case the availability is expressed **as a percentage of time when the service is available (when the test is `OK`) over the total time of the measure**. It’s also possible to measure a degraded availability when the test ends in `WARNING`--for example when the result is OK but late. There is always a lot of discussions around availability mainly concerning the downtime for maintenance or when the team is out of office. This topic deserve a dedicated article.

[^1]: Collective work, *[Site Reliability Engineering](https://www.goodreads.com/book/show/27968891-site-reliability-engineering)* (O'Reilly, 2016). Also available freely [here](https://landing.google.com/sre/book/chapters/monitoring-distributed-systems.html).
[^2]: Michael T. Nygard, *Release It!* (https://www.goodreads.com/book/show/1069827.Release_It_) (Pragmatic Bookshelf, 2007)