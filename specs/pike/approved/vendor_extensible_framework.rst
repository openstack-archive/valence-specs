..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
Add Vendor Extensible framework
================================

https://blueprints.launchpad.net/openstack-valence/+spec/add-vendor-extensible-framework

This blueprint is to provide vendor extensible framework in valence to support
management of pooled infrastructure using drivers other than
redfish.

Problem description
===================

Currently, valence supports only redfish driver, which limits it's use case
to specific hardware's which support redfish only. To provide extensibility
for other hardwares to integrate with valence, framework needs to be in place 
where other vendors will be able to plug driver's specific to manage their
pooled infrastructure.    
    
This blueprint still proposes redfish as the default driver to manage resources
but, it will provide a abstract layer to plug vendor drivers inside valence.

Proposed change
===============

Note: The following proposed change assumes valence have multi-podm support,
depends on https://blueprints.launchpad.net/openstack-valence/+spec/refactor-to-support-mutli-podmanager

With multi-podm support the following will be in place:

* New 'Connection' class added would create new instance of
  redfish using the details specified in podm. And controller would invoke
  the respective function on redfish using the connection instance.

New changes required on top of above change:

* New field 'driver' to be added while creating pod manager. If none, specified
  by default 'redfish' would be used as default driver.
* Add logic inside 'Connection' class to determine the driver to be used
  using the 'driver' field in podm
* Use stevedore to load the driver extension dynamically and store the cache
  to avoid loading each time.
* connection instance created in controller would have the details on which
  driver the call should be invoked.

API --> Controller --> Manager --> Driver

The change will be more of extension of multi-podm support to make it more
generic, no major changes would be required.

Alternatives
------------
None

Data model impact
-----------------
New field 'driver' would be added to pod_manager DB.

REST API impact
---------------
podmanager create request POST http://127.0.0.1/pod_managers would
accept new attribute driver(optional field) to specify.

Versioning impact
-----------------


Other end user impact
---------------------


Deployer impact
---------------
None

Developer impact
----------------
None

Valence GUI / Horizon impact
----------------------------
None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
 ramineni

Other contributors:

Work Items
----------
* Modify podmanager DB to add new field 'driver'
* Update controller to use generic interface exposed by Connection class
  to invoke the methods

Dependencies
============
Multi-podmanager support


Testing
=======



Documentation Impact
====================



References
==========
multi-podm patch : https://review.openstack.org/#/c/445360/8
