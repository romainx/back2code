---
title: AWS CLI Setup
date: '2018-05-13'
categories: [ops]
tags: ['aws']
---

I explain here how to interact with AWS either with the **CLI** (Command Line Interface) and with an IT automation tool: **Ansible**. Ansible is not the first tool that comes in mind for AWS (Serverless, Terraform or the built-in CloudFormation make more sense) however Ansible could be useful if you just want to configure some EC2 and specially if you have already an Ansible script somewhere around.

<!--more-->

## Prerequisites

I'm using an Anaconda as the python distribution, it’s not required but I find this distribution practical to use. I’m assuming either Anaconda or Miniconda is already installed. Please refer to [Anaconda Installation page] (https://conda.io/docs/installation.html) if it’s not the case.
You will also need AWS **Access and Secret Key pair**. If you do not know how to get them, check [this blog post](https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/).

### Conda environment

Setting up a fresh `conda` environment with the latest python version and giving it a meaningful name `aws`.

`$ conda create -n aws python=3.6.5`

Activating the newly created environment

`$ source activate aws`

## AWS CLI

Installing the AWS CLI.

`$ conda install -c conda-forge awscli`

It’s up!

```bash
$ aws --version
aws-cli/1.15.19 Python/3.6.5 Darwin/17.5.0 botocore/1.10.19
```

*Note: It's possible to define all the environment configuration in a `YAML` definition file. But here we are doing it step by step.*

## Configuring AWS CLI

Simple as answering to the questions.

```bash
$ aws configure
AWS Access Key ID [None]: AXXX
AWS Secret Access Key [None]: XXX
Default region name [None]: eu-west-1
Default output format [None]: (json is the default format)
```

Testing it with a simple command to list EC2 instances — It’s possible to use additional options like filter.

```bash
# Listing EC2 instances 
$ aws ec2 describe-instances
...
````

## Ansible

Installing Ansible

```bash
$ conda install -c conda-forge ansible
# Installing Boto3, the AWS Python SDK
$ conda install boto3
```

Testing AWS connection with a simple playbook called *List EC2 instances* retrieving the list of `t2.micro` EC2 instances already created.

```YAML
---
# Source: https://gist.github.com/romainx/681f9ea6a96ebe79ea970289cae1a59f
- name: List EC2 instances
  hosts: localhost
  # To run the playbook locally
  # http://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html#local-playbooks
  connection: local
  # There is no need to gather facts here
  gather_facts: false
  vars:
    # To tell ansible to use the Python env just created, if not set the default interpreter will be used
    # Ref.: https://stackoverflow.com/questions/41774695/ansible-ec2-boto-required-for-this-module
    - ansible_python_interpreter: "/Users/xxxx/anaconda/envs/aws/bin/python"
    - instance_type: "t2.micro"
  tasks:
    - name: "List EC2 {{ instance_type }} instances"
      ec2_instance_facts:
        aws_access_key: "AXXX"
        aws_secret_key: "XXX"
        region: "eu-west-1"
        filters:
          instance-type: "{{ instance_type }}"
      register: ec2
    - name: Print EC2 instances
      debug:
        var: ec2.instances
```

To launch the script we have to use a little hack telling Ansible to use `localhost` instead of an inventory file.

```bash
$ ansible-playbook -i localhost, test_ec2.yml
PLAY [List EC2 instances] *************************************************************************************************************
TASK [Print EC2 instances] ************************************************************************************************************
ok: [localhost] => {
    "ec2.instances": [
        {
        ...
```

Here, for convenience reasons, I’ve put the keys in the playbook. However this shall not be done like that for security reason. There are several ways to manage these keys from environment variables to `ansible-vault`. Check this [documentation](http://docs.ansible.com/ansible/latest/scenario_guides/guide_aws.html#authentication) for further information on this topic.