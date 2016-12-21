..

 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Provide Restful API and CLI for valence
==========================================

https://blueprints.launchpad.net/openstack-valence/+spec/python-valenceclient

As we known, valence project is a collection of all things Rack Scale Control,
which can consider computer, storage and network as disaggregated resource
that can be composed on the fly to meet various needs in a data center or
cloud.

But there hava no valence client for user to call the valence api gracefully
or get information conveniently in console as other OpenStack "Big Tent"
project. So not only it's neccesary to provide a valence client for user but
also for valence developer.

Problem description
===================
There have no valence client for user to call the valence api by restful api
or command-line script.

Proposed change
===============

Create a new project named python-valenceclient for valence on the OpenStack
API to support gracefully and conveniently Python API call and command-line. 

So this blueprint will be implemented into 3 phases.

**Phase I.** Support command-line 

- In this phase, the python-valenceclient will only support the command-line
  for user to get the valence information.

**Phase II.** Support Python API

- Base on the phase I, add the Python API support. So developer can call the
  Python API to get valence data directly.

**Phase III.** Integrate with OpenStack command-line

- Integrate with OpenStack command line, user can user ``openstack valence 
  *`` to get information

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Versioning impact
-----------------

None 

Other end user impact
---------------------

To use the python-valenceclient need to install the python-valenceclient package
by pip or other repo management tool.

Deployer impact
---------------

None 

Developer impact
----------------

None 

Implementation
==============

Assignee(s)
-----------

Primary Assignees
-----------------

Primary assignee:
  Jinxing Fang <fang.jinxing@99cloud.net>
  Yufei Zhang <zhang.yufei@99cloud.net>
  Shuquan Huang <huan.shuquan@99coud.net>

Other contributors:

Work Items
----------

* Add python-valenceclient initially file to openstack repo
* Add the abstract and base http request(PUT,POST,GET,DELETE...) process
* Support keystone authentication in the REST service.
* Add the RSD power API

  - power off
  - power on
  - soft power off
  - soft power on
  - inject NMI

* Add nodes API
* Add flavor API
* Add storage API
* Add systems API

Dependencies
============

Depends on valence implement related function first.
It should be possible to install the REST API service via an rpm or debian
installation.

Testing
=======

- write functional tests to cover all REST API
- unit tests

Documentation
=============

- REST API Reference
- REST API User's Guide

References
==========

None