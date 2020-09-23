---
title: 'Pytest smoke testing'
date: '2020-09-23'
categories: ['ops']
tags: ['python', 'pytest', 'testinfra']
---

It's a good practice to test that some prerequisites are met before going further. This practice is sometimes called **Smoke testing**.
When you have to setup an infrastructure either on a host or in docker images. 
In this case [Pytest][LK1] + [testinfra][LK2] is a terrific combo to perform this kind of testing.

<!--more-->

# A simple example

In a few lines of code you can easily test that some commands are working.

```python
import logging
import pytest
from testinfra import host

logger = logging.getLogger(__name__)


cases = [
    ("Dumb", "echo 'dumb'"),
    ("Golang", "go version"),
    ("Python", "python --version"),
    ("Node", "node --version"),
    ("Failed", "Failed")
]


@pytest.mark.parametrize("name,command", cases, ids=[case[0] for case in cases])
def test_command(host, name, command):
    out = host.check_output(command)
    logger.info(f"{name} [ OK ]")
```

Running this test will check that the commands run successfully.

```bash
$ pytest smoke_tests.py

=========== short test summary info ===========
FAILED smoke_tests.py::test_command[local-Failed] - AssertionError: Unexpected exit code 127 for CommandResult(command=b'Failed', exit_status=127, stdout=None, stderr=b'/bin/sh: Failed: command not found\n')
PASSED smoke_tests.py::test_command[local-Dumb]
PASSED smoke_tests.py::test_command[local-Golang]
PASSED smoke_tests.py::test_command[local-Python]
PASSED smoke_tests.py::test_command[local-Node]
=========== 1 failed, 4 passed in 0.22s ===========
```

# Going further

It's only a simple example however it is interesting since it's reproducible and will do the tedious work for you.
Beyond this simple example this smoke test can

- run, without modification, on various targets thanks to the implementation of pluggable [backends][LK3]: local machine, a docker image, several hosts through ansible, etc.,
- be extended to integrate more complex checks thanks to the implementation of built-in [modules][LK4].

[LK1]: https://docs.pytest.org/en/latest/
[LK2]: https://testinfra.readthedocs.io/en/latest/index.html#
[LK3]: https://testinfra.readthedocs.io/en/latest/backends.html
[LK4]: https://testinfra.readthedocs.io/en/latest/modules.html
