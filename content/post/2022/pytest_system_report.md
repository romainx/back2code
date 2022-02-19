---
title: 'Pytest System Report'
date: '2022-02-19'
categories: ['ops']
tags: ['python', 'pytest', 'testinfra']
---

In a [previous article][LK1], I introduced a simple way to perform system checks with [pytest][LK1] + [testinfra][LK2] .
Here it is a simple variant of the same principle with the intention of producing a **short human readable report about system configuration**.
The goal is to get and report quickly some value of interests from a system (a host). 

<!--more-->

## Using Ansible facts

One classic and efficient way to get system information is to use [Ansible facts][LK4].
Here is an example using the [`ansible` ad hoc command][LK5].

```bash
ansible localhost -m ansible.builtin.setup -a "filter=ansible_kernel"

# localhost | SUCCESS => {
#     "ansible_facts": {
#         "ansible_kernel": "21.2.0"
#     },
#     "changed": false
# }
```

However Ansible facts may not contain every information you want to check.
I find this solution sometimes overkill and not so convenient to obtain a clear output.

## Using a table output

I like table outputs in CLI, I think they are very clear. 
To obtain this kind of output in Python I often use [`tabulate`][LK6]--the use of `tabulate` to pretty print Python DataFrame is by far [my highest scored answer on Stack Overflow][LK8].
It is a convenient way to output tables in different formats from various Python data structures.

```Python
from typing import List
import pytest
from tabulate import tabulate


@pytest.fixture(scope="session")
def cases():
    return [
        ("Python", "python --version"),
        ("Node", "node --version"),
        ("Rust", "rustc --version | awk '{print $2}'"),
        ("Docker Cli", "docker version --format='{{.Client.Version}}'"),
    ]


def test_summary(host, cases):
    results: List[List] = [["Item", "Info"]]
    for label, command in cases:
        results.append([label, host.check_output(command)])
    print(tabulate(results, headers="firstrow", tablefmt="pretty"))
```

Then it should be run by 

- disabling [`pytest` capture][LK7] thanks to the `-s` flag
- reducing `pytest` verbosity thanks to the `-q` flag

```bash
pytest -s -q system_report.py

# +------------+--------------+
# |    Item    |     Info     |
# +------------+--------------+
# |   Python   | Python 3.9.1 |
# |    Node    |   v16.10.0   |
# |    Rust    |    1.58.1    |
# | Docker Cli |   20.10.12   |
# +------------+--------------+
# .
# 1 passed in 0.39s
```

This test can be run on **any testinfra backends** (remote hosts and docker images for example) to get a quick look at some important settings.

[LK1]: {{< ref "smoke_pytest.md" >}}
[LK2]: https://docs.pytest.org/en/latest/
[LK3]: https://testinfra.readthedocs.io/en/latest/index.html#
[LK4]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html#ansible-facts
[LK5]: https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html
[LK6]: https://github.com/astanin/python-tabulate
[LK7]: https://docs.pytest.org/en/6.2.x/capture.html
[LK8]: https://stackoverflow.com/a/31885295/4413446