---
title: Manage Python virtual environments with Anaconda
date: '2017-08-26'
categories: ['ops']
tags: ['python']
---

> A Virtual Environment is a tool to keep the dependencies required by different projects in separate places, by creating virtual Python environments for them. It solves the “Project X depends on version 1.x but, Project Y needs 4.x” dilemma, and keeps your global site-packages directory clean and manageable.
> -- source[^1] 

They can also be used to supply very different things like different versions of Python. Virtual Environments can be managed by the Python package manager pip, but when you are using the popular [Anaconda](https://www.continuum.io/what-is-anaconda) distribution it is necessary to use the conda package manager coming with it. This article details the most common use cases.

# Create a virtual environment

There are several methods to create a virtual environment

* **from scratch** by creating an empty environment and let the user install the required packages,
* **from an existing environment** by cloning it,
* **from a template** defining all the environment parameters.

The created environment will be available in $ANACONDA_HOME/envs.

## From scratch

It is possible to create an empty virtual environment with a specific python version.

`$ conda create --name project_x python=3`

## From an existing environment (clone)

To create an environment identical to another environment.
The example below shows how to create an environment from the root (default conda environment).

`$ conda create --name project_x --clone root`

## From a template

In python requirements (required packages) are defined in requirement.txt files[^2]. Anaconda offers the possibility to specify environments in YAML files. **This a great feature since these files can be managed in configuration** **and contain all the information required to build an environment**

* its name
* the Python version
* package to be installed by conda (under dependencies)
* package to be installed by pip

Here is an example of YAML configuration file.

```yaml
name: project_x
dependencies:
- python>=3.5
- anaconda
- yaml=0.1.6
- pip:
  - pip-check==0.2
```

The distinction between packages managed by conda and pip is also a very interesting feature.
The creation of an environment from this kind of file is triggered by the following command.

`$ run: conda env create --file environment.yml`

Note: an export in this format of the current environment can be obtained by the following command.

`$ conda env export`

# Check installed virtual environments

The following command gives the list of the installed environment and indicates the active one.

`$ conda info --envs`

or

`$conda env list`


# Activate a virtual environment**

To activate a virtual environment.

`$ source activate project_x`

And a similar command to deactivate it.

`$ source deactivate`

# Check packages

Activate the virtual environment then list installed packages by using the following command.

`$ conda list`

The result of the command shows the origin of the package. For example it displays packages installed via pip.

# Install packages

To install packages it is possible to use conda package manager. This is the preferred solution.

`$ conda install scipy`

If the package is not available, it is possible to use pip in the same environment.

`$ pip install pip-check`

# Check outdated packages

To check packages needing update (outdated), conda provides a command for that.

`$ conda search --outdated`

pip provides a similar command.

`$ pip list --outdated`

A more readable solution is provided by an external project called [pip-check](https://github.com/bartTC/pip-check/). It displays a comprehensive view of the updates to perform.

```bash
$ pip install pip-check
$ pip-check
```

However it does not work currently with pip versions > 9.0. A pull request is in progress but not merged yet.

# Update packages

The preferred way is to use the conda package manager.

`$ conda update scipy`

It is also possible to update all packages (a confirmation will be asked before proceeding).

`$ conda update --all`

Packages installed by pip have to be updated through pip.

`$ pip install pip-check --upgrade`

There is no pip built-in command to update all packages, but the following script can be used[^3]

`$ pip freeze --local | grep -v '^\-e' | cut -d = -f 1  | xargs -n1 pip install -U`

# Remove a virtual environment

To remove (delete) a virtual environment.

`$ conda remove --name project_x --all`

# Update package manager

To update the package manager itself, the following commands can be used.

`$ conda update conda`
`$ pip install --upgrade pip`

# Useful links

* [Conda cheat sheet](http://conda.pydata.org/docs/_downloads/conda-cheatsheet.pdf)
* [Conda vs. pip vs. virtualenv translator](http://conda.pydata.org/docs/_downloads/conda-pip-virtualenv-translator.html)

[^1]: [Virtual Environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/)
[^2]: [pip install - pip 9.0.1 documentation](https://pip.pypa.io/en/stable/reference/pip_install/#requirements-file-format)
[^3]: [Upgrading all packages with pip](http://stackoverflow.com/questions/2720014/upgrading-all-packages-with-pip)
