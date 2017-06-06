..

 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Pooled resources management
===========================

This blueprint is to add framework in valence for pooled resources management.
https://blueprints.launchpad.net/openstack-valence/+spec/add-device-orchestration 


Problem description
===================

Currently, valence doesn't provide support for the dynamic management of
pooled resources like storage, network and other pci devices which can be 
connected on demand to a composed node, giving user the ability to attach or
detach the devices dynamically based on workload.


Proposed change
===============

* Discover the pooled resources associated with particular podmanager.
* New DB table 'devices' to be added to register the pooled resources to valence
* Two options :
    1. Auto register all the devices when podm registered
    2. Provide option to user to register the devices to be managed by valence
       from the pooled resources

Alternatives
------------

Data model impact
-----------------

REST API impact
---------------

Driver API impact
-----------------
None

Nova driver impact
------------------
None

Security impact
---------------
None

Other end user impact
---------------------

Scalability impact
------------------

Performance Impact
------------------
None

Other deployer impact
---------------------

Developer impact
----------------

Valence GUI / Horizon impact
----------------------------


Implementation
==============
Assignee(s)
-----------


Work Items
----------


Dependencies
============

Testing
=======

Documentation Impact
====================
None

References
==========
None
