---
title: Error Budgets
date: '2018-10-24'
categories: ['ops']
---

> SRE has found that roughly 70% of outages are due to changes in a live system.

# Problem

Knowing this, there is no need to look any further the reasons why SRE teams--or production team or whatever the team that will be called by angry customers--are so reluctant to change. If it's not enough, just remind that their objectives are certainly based on the **reliability** of the services they maintain.

On the other side, teams in charge of developing new products are trying to push their code into production as often as possible--agility for better or worse encourages this trend-- to provide new features to customers or to fix their own bugs and also because they are evaluated on their **velocity**.

So you end up with a kind of dichotomy between two populations that work on the same product but not at the same level and that does not share the same vision nor the same objectives.

# Solution

The obvious solution is to have only one team performing the whole scope, but it's not always possible. 
To solve this dilemma without hours of negotiations or a blind approval of some manager--who will only take the decision based on its own objectives--the **Google SRE team** has found a data-driven answer called **the error budget**.

> Instead, our goal is to define an objective metric, agreed upon by both sides, that can be used to guide the negotiations in a reproducible way. The more data-based the decision can ben the better.

The answer is to base the decision on SLO (Service Level Objective): How unreliable the service is allowed to be within a single quarter. **How much margin (this is the error budget) do we have to deploy new versions that will potentially break the production and impact the SLO?**

And it's quite simple to implement:

* SLO are already defined for other purposes,
* Actual performance against the SLO are already measured by the monitoring system.

Ending with a simple rule: **As long as actual performance is above the SLO new releases can be pushed in production.**

# Benefits

There are many benefits to apply this strategy.

* Avoid endless meetings.
* It helps to get closer to the right balance between innovation and reliability.
* It is based on an objective metric shared across the organisation, data-driven, reproductible and not based on feelings nor on politics.
* When release deployment is frozen, the time is invested in reliability instead of new features, this will be useful in the future.
* Can be used to show to the stakeholders the drawback of too high
  * availability targets slowing down the deployment of new features
  * frequency of new features at the expense of reliability

**Disclaimer**: Most of this article is a rephrasing--or a summary or at least my understanding--of the chapter "Motivation for Error Budgets" of the amazing book from Google *Site Reliability Engineering*[^fn-sre].

[^fn-sre]: Collective work, *[Site Reliability Engineering](https://www.goodreads.com/book/show/27968891-site-reliability-engineering)* (O'Reilly, 2016). This book is also available for free [here](https://landing.google.com/sre/sre-book/toc/index.html).