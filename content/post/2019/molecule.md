---
title: 'Molecule'
date: '2019-08-26'
tags: [dev]
---

> Molecule is designed to aid in the development and testing of Ansible roles.
Molecule provides support for testing with multiple instances, operating systems and distributions, virtualization providers, test frameworks and testing scenarios.
Molecule encourages an approach that results in consistently developed roles that are well-written, easily understood and maintained.

[Molecule](https://molecule.readthedocs.io/)

# Installation

```bash
$ conda create -n molecule python=3.7
$ source activate ansible
$ conda install -c conda-forge ansible docker-py docker-compose molecule
# docker-py seems to be called docker in PyPi
$ pip install ansible docker docker-compose molecule
```

# Main features

- Cookiecutter to create role from a standardized template.
- Check role syntax and raise syntax errors.
- Linters (`yaml-lint`, `ansible-lint`, and `flake8`) to assess that the all the elements are well written.
- Provide sandbox to develop and test roles.
- Test platform to validate the role behavior.
- Continuous Integration platform to ensure that the role is not broken before using it in production.

Great to **develop**, **test** and **validate Ansible roles** within an **industrial process**.

# Detail

## Create role

This acts as a **cookiecutter** to create a role from a pre-defined template.

```bash
$ molecule init role -r molecule-role
# also init a scenario in an existing role
$ molecule init scenario -r my-role-name
```

```bash
$ tree molecule-role

# molecule-role
# ├── README.md
# ├── defaults
# │   └── main.yml
# ├── handlers
# │   └── main.yml
# ├── meta
# │   └── main.yml
# ├── molecule # This is the molecule specific folder
# │   └── default
# │       ├── Dockerfile.j2
# │       ├── INSTALL.rst
# │       ├── molecule.yml
# │       ├── playbook.yml
# │       └── tests
# │           ├── __pycache__
# │           │   └── test_default.cpython-37.pyc
# │           └── test_default.py
# ├── tasks
# │   └── main.yml
# └── vars
#     └── main.yml
```

## Check syntax

Verifies the role for syntax errors. Saves time instead of finding error when running the playbook on a host.

```bash    
$ molecule syntax
```

## Lint

Executes `yaml-lint`, `ansible-lint`, and `flake8`, reporting failure if there are issues.

```bash
molecule lint
```

### Configuration of linters

The linters for example `yaml-lint` can be customized in the `molecule.yml` file or in a `.yamlint` file.

Note: default behavior comes from the [cookiecutter template](https://github.com/ansible/molecule/blob/master/molecule/cookiecutter/molecule/%7B%7Bcookiecutter.role_name%7D%7D/.yamllint).

```yaml
lint:
  name: yamllint
  # see https://yamllint.readthedocs.io/en/stable/index.html
  options:
    config-data:
      extends: default
      rules:
        indentation:
          level: warning
          indent-sequences: consistent
        line-length: disable
```

They can also be disabled it's useful for flake8 since it's not always necessary to check rules for tests.

```yaml
verifier:
  name: testinfra
  lint:
    name: flake8
    enabled: false
```

## Create instances

Create an instance with the configured driver and configure instances with preparation playbooks.

```bash
$ molecule create

# Can check that an image has been created
$ docker images

# REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
# molecule_local/centos                 7                   0de51c165b8c        58 seconds ago      251MB

# Can check that at this step the containers are running
$ docker ps

# CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS              PORTS               NAMES
# a4d4ed773ce2        molecule_local/ubi7/ubi:latest   "bash -c 'while true…"   5 minutes ago       Up 5 minutes                            ubi_7
# 6b6e61d7ce42        molecule_local/centos:7          "bash -c 'while true…"   5 minutes ago       Up 5 minutes                            centos_7
```

The default driver is docker, the platforms are defined in the `molecule.yml` file.

```yaml
driver:
  name: docker
platforms:
  # https://molecule.readthedocs.io/en/stable/configuration.html#docker
  - name: instance
    image: centos:7
  - name: ubi_7
    # https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/ubi7/ubi
    image: ubi7/ubi:latest
    registry:
      url: registry.access.redhat.com
```

Can be configured to pull images from private registries with authentication. Check [here](https://molecule.readthedocs.io/en/stable/configuration.html#docker).

## Prepare / Cleanup

> The prepare playbook executes actions which bring the system to a given state prior to converge. It is executed after create, and only once for the duration of the instances life.
This can be used to bring instances into a particular state, prior to testing.

```bash
molecule prepare
```

Run the playbook configured in the `molecule.yml` file.

```yaml
provisioner:
  name: ansible
  playbooks:
    prepare: prepare.yml
    cleanup: cleanup.yml
```

## Converge (run)

Execute (run) playbooks targeting hosts. In fact the playbook `playbook.yml` inside the `molecule` folder is run.

```bash
$ molecule converge
```

## Login

It's a handy feature that permit to login into the running instances for troubleshooting or testing commands. Instances have to be created (need to run)  to be able to use this command

```bash
$ molecule login --host ubi_7
# Or if only one instance
$ molecule login
```

## Idempotence

Execute a playbook twice and fails in case of changes in the second run (non-idempotent).

```bash
$ molecule idempotence
```

## Verify

Execute server state verification tools (`testinfra` or `goss`).

```bash
$ molecule verify
```

## Destroy

Destroy instances.

```bash
$ molecule destroy
```

## Test

Executes **all the previous steps**. So in this case all the workflow is started from scratch.

``` bash
$ molecule test   

# Default workflow     
# └── default
#     ├── lint
#     ├── cleanup
#     ├── destroy
#     ├── dependency
#     ├── syntax
#     ├── create
#     ├── prepare
#     ├── converge
#     ├── idempotence
#     ├── side_effect
#     ├── verify
#     ├── cleanup
#     └── destroy
```

# Tips

Configure ansible `stdout` to `yaml` to improve human reading experience.

```yaml
provisioner:
  name: ansible
  config_options:
    defaults:
			# Part of ansible.cfg configuration to improve reading experience
      stdout_callback: yaml
```

# References / Further reading

- [Test-driven infrastructure development with Ansible & Molecule - codecentric AG Blog](https://blog.codecentric.de/en/2018/12/test-driven-infrastructure-ansible-molecule/)
- [Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule)
- [Testing Ansible roles with Molecule](https://opensource.com/article/18/12/testing-ansible-roles-molecule)