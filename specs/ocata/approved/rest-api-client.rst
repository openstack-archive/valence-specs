====================================
Provide REST API and CLI for Valence
====================================

Rack Scale Design (RSD) is a open architecture for management of disaggregated
server hardware resources.
Valence is an opensource openstack project that provides a mechanism to control
all of RSD.

Python-valenceclient is a client for the OpenStack valence API. There's a
Python API (the valenceclient module), and a command-line script (valence)

The current plan for Ocata is to add to the API:
- nodes
  - post
  - delete
  - list
  - get
- power
  - power off
  - power on
  - soft power off
  - soft power on
  - inject NMI
- flavor
  - get
  - post
  - list
  - delete
- storage
  - get
  - post
  - delete
- systems
  - list
  - get

The current plan for Ocata is to add to the CLI:
- nodes
  - post
  - delete
  - list
  - get
- power
  - power off
  - power on
  - soft power off
  - soft power on
  - inject NMI
- flavor
  - get
  - post
  - list
  - delete
- storage
  - get
  - post
  - delete
- systems
  - list
  - get
- stop/kill services on host

===================
Problem Description
===================

Proposed Change
===============
- create a new project named python-valenceclient

Dependencies
------------
Depends on valence implement related function first.

It should be possible to install the REST API service via an rpm or debian installation.

Security
--------
Authentication will be provided by:
- keystone
- noauth(like ironic)

Primary Assignees
-----------------
- Shuquan Huang
- Jinxing
- Yufei Zhang

Milestones
----------
Target Milestone for completion:
  Ocata

Work Items
----------
- implement simple / initial REST API supporting current CLI
commands
- support keystone authentication in the REST service.
- add support for system
- add support for nodes
- add support for flavor
- add support for power

Testing
=======
- write functional tests to cover all REST API
- unit tests

Documentation
=============
- REST API Reference
- REST API User's Guide
