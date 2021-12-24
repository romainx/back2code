---
title: 'The with statement'
date: '2021-12-24'
categories: ['dev']
tags: ['Python']
---

The `with` statement in Python allows the execution of a block of code inside a context manager responsible of the initialization and the finalization code. It is often used in the standard library and one of the most frequent usage is for interacting with files. The context manager is responsible of the file lifecycle: initialization code is responsible of opening the file while the finalization code is responsible of closing it. It's common and convenient to use `with` statement, but it can be interesting to define our own implementation for dedicated tasks like running a Docker container. Let's check how to do it.

<!--more-->

## Overview

Implementing the `with` statement in a `class` is straightforward. It is just a matter of implementing two methods.

- Initialization code lies into the `__enter__` method that is called when entering the `with` context.
- Finalization code lies into the `__exit__` method that is called when leaving the `with` context.

## In the Docker context

In this case the context manager will be responsible of a Docker container lifecycle (run and stop it) to let the user focus on the code interacting with the running container. Here is a typical usage where we want to run an arbitrary command inside a running container.

```Python
# interacting with a ubuntu container 
with DockerRunner("ubuntu") as cont:
    # here in the with context, the container is started
    # we can execute a command
    out = cont.exec_run("echo hello")
# here out of the with context the container is stopped
```

## Prerequisites

First will store in the class the properties required to manage the container lifecycle.

```python
class DockerRunner:
    def __init__(
        self, image_name, docker_client=docker.from_env(), command="sleep infinity"
    ):
        # the container
        self.container = None
        # the name of the image to use
        self.image_name = image_name
        # the command to run to start the container
        self.command = command
        # the docker client used to interact with the container
        self.docker_client = docker_client
```

## Initialization code

When entering the context we need to run the container, it is done by calling the `containers.run` method.
This method takes the following parameters and returns a container object.

- `image`: The name of the image to use
- `command`: The command to run the container. Here we are using `sleep infinity` to keep the container running.
- `detach`: We tell the docker client to run it as a detached container, equivalent to the `-d` flag.

It is the equivalent of running the command `docker run -it -d ubuntu sleep infinity`

```Python
def __enter__(self):
    logging.info(f"Creating container for image {self.image_name} ...")
    self.container = self.docker_client.containers.run(
        image=self.image_name,
        command=self.command,
        detach=True,
    )
    logging.info(f"Container {self.container.name} created")
    return self.container
```

## Finalization code

When leaving the context we want to stop and remove the running container.

```Python
def __exit__(self, exc_type, exc_value, traceback):
    if self.container:
        try:
            logging.info(f"Stopping container {self.container.name} ...")
            self.container.stop(timeout=0)
        finally:
            logging.info(f"Removing container {self.container.name} ...")
            self.container.remove(force=True)
        logging.info(f"Container {self.container.name} removed")
```

## Testing it

Now we want to test our new `DockerRunner` featuring a `with` context manager.
We will run a `ubuntu` image and assert that we can run an `echo` command inside.

```Python
def test_echo(text="test", image="ubuntu"):
    cmd = f"echo {text}"
    logging.info(f"Checking if echo command is working in {image} ...")
    with DockerRunner(image) as cont:
        out = cont.exec_run(cmd)
    assert out.exit_code == 0, f"Command: {cmd} failed"
    result = out.output.decode("utf-8")
    logging.info(f"Got output from command: {result}")
    assert text in result, f"Command result not expected: {result}"
```

## Putting it all together

Here is the code in a `docker_with.py` file.

```python
import docker
import logging


class DockerRunner:
    def __init__(
        self, image_name, docker_client=docker.from_env(), command="sleep infinity"
    ):
        self.container = None
        self.image_name = image_name
        self.command = command
        self.docker_client = docker_client

    def __enter__(self):
        logging.info(f"Creating container for image {self.image_name} ...")
        self.container = self.docker_client.containers.run(
            image=self.image_name,
            command=self.command,
            detach=True,
        )
        logging.info(f"Container {self.container.name} created")
        return self.container

    def __exit__(self, exc_type, exc_value, traceback):
        if self.container:
            try:
                logging.info(f"Stopping container {self.container.name} ...")
                self.container.stop(timeout=0)
            finally:
                logging.info(f"Removing container {self.container.name} ...")
                self.container.remove(force=True)
            logging.info(f"Container {self.container.name} removed")


def test_echo(text="test", image="ubuntu"):
    cmd = f"echo {text}"
    logging.info(f"Checking if echo command is working in {image} ...")
    with DockerRunner(image) as cont:
        out = cont.exec_run(cmd)
    assert out.exit_code == 0, f"Command: {cmd} failed"
    result = out.output.decode("utf-8")
    logging.info(f"Got output from command: {result}")
    assert text in result, f"Command result not expected: {result}"
```

Let's run the test.

```bash
pytest --log-cli-level=INFO docker_with.py

# docker_with.py::test_echo 
# ----------
# INFO     root:docker_with.py:37 Checking if echo command is working in ubuntu ...
# INFO     root:docker_with.py:14 Creating container for image ubuntu ...
# INFO     root:docker_with.py:20 Container sweet_ganguly created
# INFO     root:docker_with.py:27 Stopping container sweet_ganguly ...
# INFO     root:docker_with.py:30 Removing container sweet_ganguly ...
# INFO     root:docker_with.py:32 Container sweet_ganguly removed
# INFO     root:docker_with.py:42 Got output from command: test

# PASSED  
```

## References / Further reading

- [With Statement Context Managers](https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers)
- [Docker SDK Containers](https://docker-py.readthedocs.io/en/stable/containers.html)