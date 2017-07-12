..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
Add Vendor Extensible framework
================================

https://blueprints.launchpad.net/openstack-valence/+spec/add-vendor-extensible-framework

This spec is to provide vendor extensible framework in valence to support
management of pooled infrastructure using drivers other than
redfish.

Problem description
===================

Currently, valence supports only redfish driver, which limits it's use case
to specific hardware's which support redfish only. To provide extensibility
for other non-Redfish disaggregated/composable hardwares to integrate with
valence, framework needs to be in place where other vendors will be able to
plug driver's specific to manage their pooled infrastructure.

This blueprint still proposes redfish as the default driver to manage
resources, but it will provide an abstract layer to plug vendor drivers inside
valence.

Proposed change
===============

Note: The following proposed change assumes valence have multi-podm support,
depends on https://blueprints.launchpad.net/openstack-valence/+spec/refactor-to-support-mutli-podmanager

* New field 'type' to be added while creating pod manager. If none specified,
  by default instantiates default podmanager, PodManagerBase as defined in
  multi-podm patch.
* 'type' would take following values: 'default', '2.1', 'lenovo' etc.
* Use stevedore to load respective podmanager based on 'type' dynamically and
  store the cache to avoid loading each time.
* New module manager.py to be added, to load respective podmanager instance.
* Controller will use generic interface, to call respective function on Manager
  instance.
* Create a new folder where all the podmanagers files would be placed and
  move the current redfish folder to the same.

API --> Controller --> Manager --> Podmanager

The change will be more of extension of multi-podm support to make it more
generic, no major changes would be required.

Alternatives
------------
None

Data model impact
-----------------
New field 'type' would be added to pod_manager DB.

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
* Modify podmanager DB to add new field 'type'
* Update controller to use generic interface exposed by Manager class
  to invoke the methods
* Add new folder and move the current redfish implementation to the same.

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
