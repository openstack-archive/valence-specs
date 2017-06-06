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

* Discover the pooled resources associated with specified podmanager.
* New DB table 'devices' to be added to register the pooled resources to
  valence

For registering the devices to valence DB

* Two options are available :
    1. Auto register all the devices when podm is registered
    2. Provide option to user to register the devices to be managed by valence
       from the pooled resources
* API's to be provided to list the pooled devices, attach/detach device to node
  using uuid


Alternatives
------------
None

Data model impact
-----------------
New table 'devices' will be added with following schema

+-------------------+
| Field             |
+-------------------+
| uuid              |
| podm_id           |
| node_id           |
| type              |
| properties        |
| pooled_group_id   |
| state             |
| created_at        |
| deleted_at        |
+-------------------+

REST API impact
---------------
New API's to be added list pooled resources and attach/detach devices

GET v1/pooled API to be added with filter params to list specific devices
GET v1/pooled?type=storage
POST /v1/nodes/{node_ident}/action
{
  "attach":{
    device_id: 'a698b7c2-0ca1-481a-87cd-4717a3d9c4d6'
   }
}
POST /v1/nodes/{node_ident}/action
{
  "detach":{
    device_id: 'a698b7c2-0ca1-481a-87cd-4717a3d9c4d6'
   }
}

Ref: https://github.com/openstack/valence/blob/master/api-ref/source/valence-api-v1-pooled.inc
Ref: https://github.com/openstack/valence/blob/master/api-ref/source/mockup/node-post-action-attach-request.json

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

* https://github.com/openstack/valence/blob/master/api-ref/source/valence-api-v1-pooled.inc
* https://github.com/openstack/valence/blob/master/api-ref/source/mockup/node-post-action-attach-request.json
