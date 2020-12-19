---
title: Postmortem
date: '2020-12-19'
categories: [ops]
tags: ['itsm']
---

Recently **big outages** have occurred impacting two of the main cloud vendors

- [Amazon Kinesis Event](https://aws.amazon.com/message/11201/): *2020-11-25*
- [Google Cloud Infrastructure Components Incident #20013](https://status.cloud.google.com/incident/zall/20013#20013004): *2020-12-14*

highlighting the importance of a process dedicated to analyse the incidents or outage in the aim of knowing what happened and why, but above all to learn and take appropriate actions in order to avoid the same problems to occur again. This process is called **postmortem** — or post mortem in two words.

<!--more-->

## The postmortem process

In the great book written by **Google** *Site Reliability Engineering*[^FN1], [a chapter](https://sre.google/sre-book/postmortem-culture/) — in fact an article — called *Postmortem Culture: Learning from Failure* is dedicated to this process. This article focus on the process around the postmortem, how it is done, communicated and promoted. The idea behind is to consider that beyond the study and the actions taken there is a need to **communicate widely to educate people** and also to encourage speak up on this topic by promoting a **blameless culture**.

## The postmortem template

Another more pragmatic aspect of performing a postmortem is simply how to write it, what need to be done. Saying it differently what are the sections to address and a template is great to enforce a consistent writing of postmortems. **Atlassian** proposes a [complete guideline](https://www.atlassian.com/incident-management/postmortem) on this topic and identifies the following sections. I have made some regroupments and highlighted the fact that everything is not done in the same time frame, the **priority remaining obviously to resolve the incident and restore the service**.

### Incident summary

- **Incident summary**: Write a summary of the incident in a few sentences.
- **Leadup**: Describe the circumstances that led to this incident
- **Fault**: Describe what failed to work as expected.
- **Impact**: Describe how the incident impacted internal and external users during the incident.
- **Detection**: When did the team detect the incident? How did they know it was happening?
- **Response**: Who responded to the incident? When did they respond, and what did they do? Note any delays or obstacles to responding.
- **Recovery mitigation and resolution**: What steps did you take to resolve this incident?
- **Timeline**: Detail the incident timeline. We recommend using UTC to standardize for timezones.

### Incident analysis

- **Root causes**: Run [a 5-whys analysis](https://www.atlassian.com/team-playbook/plays/5-whys) to understand the true causes of the incident
- **Recurrence**: Now that you know the root cause, can you look back and see any other incidents that could have the same root cause?
- **Backlog check**: Review your engineering backlog to find out if there was any unplanned work there that could have prevented this incident, or at least reduced its impact?

### Corrective actions and improvements

- **Lessons learnt**: What went well? What could have gone better? What else did you learn?
- **Corrective actions**: Describe the corrective action ordered to prevent this class of incident in the future.

[^FN1]: Beyer, Betsy, et al. *[Site Reliability Engineering: How Google Runs Production Systems](https://www.goodreads.com/book/show/27968891-site-reliability-engineering)*. O’Reilly Media, Inc, USA, 2016.