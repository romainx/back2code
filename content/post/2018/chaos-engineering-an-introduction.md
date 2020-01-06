---
title: 'Chaos Engineering: an introduction'
date: '2018-05-01'
categories: [ops]
---

![chaos](/post/chaos-engineering-an-introduction_files/chaos.png)

> **Chaos Engineering** is the discipline of experimenting on a distributed system in order to build confidence in the system’s capability to withstand turbulent conditions in production.
-- *[Principles of Chaos](http://principlesofchaos.org/)*

**Netflix** has been a practitioner of **Chaos Engineering** for a long time and at the origin of what can be called now **a discipline** -- some organizations have a dedicated chaos engineering team.
Testing is about performing well defined tests in well defined conditions and expecting a well known answer. **Chaos Engineering** is more about **conducting experiments by producing a failure and learning from it** (how the system behave). According to the result it could be time to work on improving things.

According to each use case chaos engineering can spread from:

* **lower layers**: at infrastructure level by rebooting an instance,
* to **higher layers**: at service level by flooding an API with a kind of DoS,
* or even at **team level** by producing a failure requiring manual intervention, in this case it’s called **game days**.

The key point behind all of that is **if you have not experienced such cases**, **believing and saying to your customers that your services are resilient is just a fairy tale** -- even if well designed and documented.
And the last point is that it’s **much more confortable to experiment such failures when you know that they will happen and that you are ready to handle it**, than in an unexpected way when you are forced to fix it in urge while a lots of customers are stuck, your boss -- and also his boss -- is on the phone and it’s Friday 5 PM. Mature organizations like Netflix create failures 24/7 in an automated way with its [Simian Army](https://medium.com/netflix-techblog/the-netflix-simian-army-16e57fbab116), but it’s better to start smaller.

There is so much to say on this topic that I will write next posts. In the mean time my advice is to start with the free ebook -- my advice is always to start by reading a book -- *[Chaos Engineering: Building Confidence in System Behavior through Experiments](http://www.oreilly.com/webops-perf/free/chaos-engineering.csp)* (the illustration of this article has been borrowed from the book cover), you can also check -- and contribute to -- the **list of books** dedicated to this topic I’ve created on Goodreads: [Chaos Engineering](https://www.goodreads.com/list/show/122960.Chaos_Engineering).
