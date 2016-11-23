..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================
RSD PXE driver
======================

https://bugs.launchpad.net/ironic/+bug/1614825

This blueprint proposes adding new driver that supports deployment of Rack
Scale Design (RSD) Server.

In this blueprint servers, nodes are used interchangeably that denotes Rack
Scale Design (RSD) servers.

Problem description
===================
Rack Scale Design (RSD) is a open architecture for management of disaggregated
server hardware resources.

Currently RSD deesn't provide IPMI controll for its node, so ironic can't manage
RSD nodes now.

Valence is an opensource openstack project that provides all controll of RSD.

This blueprint proposes new driver to manage RSD servers using openstack-valence
client.

Proposed change
===============

New power and management interfaces will be added as part of pxe_rsd driver.
This driver uses the openstack-valence-client for communicating with RSD.

This driver uses
    * pxe.PXEDeploy for PXE deployment operations
    * rsd_tool.RSDPower for power operations
    * rsc_tool.RSDManagement for management interface operations

The RSD Power interface will inherit the base PowerInterface and implement:

  * get_power_state
  * set_power_state
  * reboot
  * get_properties
  * validate

The RSD Management interface will inherit the base ManagementInterface and
implement:

  * get_properties
  * validate
  * get_supported_boot_devices
  * get_boot_device
  * set_boot_device
  * get_sensors_data - This will raise NotImplemented


Alternatives
------------
None

Data model impact
-----------------
None

RPC API impact
--------------
None

State Machine Impact
--------------------
None

REST API impact
---------------
None

Client (CLI) impact
-------------------
None

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
None

Scalability impact
------------------
None


Performance Impact
------------------
None


Other deployer impact
---------------------
The following driver_info fields are required while enrolling node into ironic:
    * podm_address: PodManager hostname/ip-address
    * podm_username: User account for the podm manager
    * podm_password: User account password

The following driver_info fields are optional to help rsd filter the proper node
while enrolling node into ironic:
    * min_cpu: minnum cpu num for the node
    * min_memory: minnum memory for the node
    * remote_storage: remote disk attached to the node

The following parameters are added in to the node properties after node successful
enroll

    * node_uuid: the node's uuid that rsd returns, this will help identify the node
    * memory_mb: the node's memory size
    * cpus: the nodes's cpu number
    * cpu_arch Processor instruction set including: x86, x86-64, IA-64, ARM-A32, ARM-A64
    * cpu_model Processor model, like "Multi-Core Intel(R) Xeon(R) processor 7xxx Series"
    * interfaces: the node's ethernet interfaces


Developer impact
----------------
None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
zhangyufei

Other contributors:
huangShuquan


Work Items
----------

* Add new pxe_rsd driver, extending power and management interface APIs.
* Writing and unit-test cases for pxe_rsd driver.
* Writing configuration documents.

Dependencies
============
This driver requires openstack-valence-client installed on the conductor node.

Testing
=======
Unit-tests will be implemented for new pxe_rsd driver.
tempest test suite will be updated to cover the pxe_rsd driver.
Continuous integration (CI) support will be added for rsd servers.

Upgrades and Backwards Compatibility
====================================
This driver will not break any compatibility with either on REST API or RPC
APIs.

Documentation Impact
====================
* Writing configuration documents.
* Updating Ironic documentation section _`Enabling Drivers`:
  http://docs.openstack.org/developer/ironic/deploy/drivers.html with pxe_rsd
  driver related instructions.

References
==========

_`openstack valence`:https://wiki.openstack.org/wiki/Valence
