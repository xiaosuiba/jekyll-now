---
layout: post
title: Shade Issue With Openstack Newton
---
Recently, I'm trying to create a installation tool to deploy openstack newton automatically. Ansible has a great support for openstack, the cloud module. With ansible I can create openstack users, projects easily with building module instead of shell scripts. All I need is to install shade python package in both controller and the node to be controlled.

However, this time I ran into a big trouble. After install shade with pip:

```
pip install shade
```

Then I found the openstack client has been damaged!!  When I execute `openstack user list`, I got this error:

```
__init__() got an unexpected keyword argument 'app_name'
```

After hours of debugging and googling, I found the reason why it happened. When I installed shade through pip, it update a bunch of other libraries which are not in the dependencies list of shade in the pip official site.

```bash
Installing collected packages: os-client-config, jmespath, pyparsing, cliff, oslo.utils, osc-lib, python-magnumclient, openstacksdk, python-openstackclient, python-ironicclient, munch
  Found existing installation: os-client-config 1.21.1
    Uninstalling os-client-config-1.21.1:
      Successfully uninstalled os-client-config-1.21.1
  Found existing installation: pyparsing 2.0.3
    DEPRECATION: Uninstalling a distutils installed project (pyparsing) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
    Uninstalling pyparsing-2.0.3:
      Successfully uninstalled pyparsing-2.0.3
  Found existing installation: cliff 2.2.0
    Uninstalling cliff-2.2.0:
      Successfully uninstalled cliff-2.2.0
  Found existing installation: oslo.utils 3.16.0
    Uninstalling oslo.utils-3.16.0:
      Successfully uninstalled oslo.utils-3.16.0
  Found existing installation: osc-lib 1.1.0
    Uninstalling osc-lib-1.1.0:
      Successfully uninstalled osc-lib-1.1.0
  Found existing installation: openstacksdk 0.9.5
    Uninstalling openstacksdk-0.9.5:
      Successfully uninstalled openstacksdk-0.9.5
  Found existing installation: python-openstackclient 3.2.0
    Uninstalling python-openstackclient-3.2.0:
      Successfully uninstalled python-openstackclient-3.2.0
  Running setup.py install for munch ... done
Successfully installed cliff-2.4.0 jmespath-0.9.1 munch-2.1.0 openstacksdk-0.9.13 os-client-config-1.26.0 osc-lib-1.3.0 oslo.utils-3.22.0 pyparsing-2.1.10 python-ironicclient-1.11.0 python-magnumclient-2.5.0 python-openstackclient-3.8.1
```

I try to recover the issue by reinstall those python packages changed by shade and find out it's the very package **cliff** that brings the issue.

I found two workarounds:

1. update python-openstackclient through pip.
2. install shade and its essensial dependencies with —no-deps arg through pip.

As it's best to keep those packages intalled from yum repository,  it's always better to choose the 2nd way.



