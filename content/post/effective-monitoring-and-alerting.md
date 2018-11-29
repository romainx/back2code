---
title: Effective Monitoring and Alerting
date: '2018-11-12'
tags: [ops, book]
---

![Book cover](/post/effective-monitoring-and-alerting_files/effective-monit.jpg)

A short note about this book I used in my work. First of all two good points. The first is that it deals with monitoring, alerting and reporting in general, that is to say **independently of the tools used**. This is both a strong point and a weak point since it could be useful to identify families of tools adapted to each use. This step back is not so common and allows to introduce higher level concepts, for example the organization of the monitoring in stacks which is absolutely crucial but also notions and general definitions applicable in all circumstances - or almost. And we come to the second strong point, **definitions**. It is essential in the professional context to rely on precise definitions that allow framing concepts that most people have an unfortunate tendency to confuse as monitoring and alerting, for example.

In the weak points, it lacks background and practical cases. If we do not know the subject well, we will finish reading about as bad -- I exaggerate, we will at least be armed with definitions and concepts and that's already a lot. The writing is completely devoid of soul: no humor, no anecdotes which makes reading quite boring. 

The most problematic point is the structure of the book that is really unclear and will not permit to refer to it easily to find an element. More importantly, it lacks structuring elements for the implementation of a solution like the **4 golden signals**.

> - **Latency**: The time it takes to service a request, with a focus on distinguishing between the latency of successful requests and the latency of failed requests.
> - **Traffic**: A measure of how much demand is being placed on the service. This is measured using a high-level service-specific metric, like HTTP requests per second in the case of an HTTP REST API.
> - **Errors**: The rate of requests that fail. The failures can be explicit (e.g., HTTP 500 errors) or implicit (e.g., an HTTP 200 OK response with a response body having too few items).
> - **Saturation**: How “full” is the service. This is a measure of the system utilization, emphasizing the resources that are most constrained (e.g., memory, I/O or CPU). Services degrade in performance as they approach high saturation.

Or the **5 golden signals** if we add to that the measure of **availability** -- I wrote an [article](https://back2code.svbtle.com/the-4-golden-signals-1) about it. Despite these reservations, it is still useful to have it on hand to refer to it from time to time, but it is not a must have -- far from it. I would rather read with interest a recent book published by the same editor: *Practical Monitoring: Effective Strategies for the Real World*. 

**Slawek Ligus**, *[Effective Monitoring and Alerting](https://www.goodreads.com/book/show/17084113-effective-monitoring-and-alerting)* (O'Reilly, 2012)