---
title: 'ipaddress: IPv4/IPv6 manipulation like a boss'
date: '2024-11-02'
categories: ['ops']
tags: ['python']
---

If you are not a network engineer having to deal with IP adresses and  IP networks (like me), you might be interested by this post.
I recently discovered a very convenient Python module called [`ipaddress`](https://docs.python.org/3/library/ipaddress.html). It's a no-brainer for common questions.

* Is an IP address part of a network?

```python
from ipaddress import ip_address, ip_network

ip_address("192.168.0.16") in ip_network("192.168.0.0/28")
# False
```

* Is a network a subnet of another network?

```python
ip_network("192.168.0.0/28").subnet_of(ip_network("192.168.0.0/22"))
# True
```

* How many ip addresses is there in a network?

```python
ip_network("192.168.0.0/22").num_addresses
# 1024
# I know also the formulae 2^(32-n)
2 ** (32-22)
# 1024
```