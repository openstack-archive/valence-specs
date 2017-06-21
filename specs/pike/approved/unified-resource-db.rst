..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================
Unified Resource Database
=========================

This spec reworks the Valence DB for resources controlled by pod manager so that
they can all fall under one 'Resources' table with pointers to where they can be
found via the pod manager APIs.

https://blueprints.launchpad.net/openstack-valence/+spec/unified-resource-db

Problem description
===================

With each resource having its own database entry and maintaining descriptions of
the properties and metadata, a couple of problems arise.

- If we do maintain information about metadata, we have to ensure that it stays up
  to date if something about the pod manager or resource changes (e.g. it is allocated
  or fails). This will require a call out to the pod manager each time the resource
  is requested, which is the thing that we would be trying to avoid doing by keeping
  it in our own DB.

- If we don't maintain information about metadata but give each resource its own
  database table, there will be a lot of redundancy between tables since all we'll
  be storing is info about links to pod managers and resource type. This can be
  collapsed into one all-encompassing table.

Proposed change
===============

Create one DB table for all pod manager managed resources, "Resources", that looks
like the following.

+----------------+-------------------------------------------------------+
| Field          | Value                                                 |
+----------------+-------------------------------------------------------+
| uuid           | Valence UUID for resource.                            |
+----------------+-------------------------------------------------------+
| podm_id        | Valence UUID of pod manager containing resource.      |
+----------------+-------------------------------------------------------+
| resource_url   | URL where resource exists in the pod manager.         |
+----------------+-------------------------------------------------------+
| resource_type  | Type of resource (System, NVMe, Ethernet Port, etc.). |
+----------------+-------------------------------------------------------+

Alternatives
------------

Alternatives and why they aren't ideal were covered in the problem description. Either
maintaining all of the metadata information about each resource in the Valence DB
(which could get out of sync with the pod manager and would need to be refreshed) or
maintining multiple database entries for pointers to resources in the pod manager,
which would create redundancy and isn't needed.

Data model impact
-----------------

All resource database entries will be covered by the table provided in the proposed
change.

REST API impact
---------------

The API should stay the same, this is an internal controller/db change.

Versioning impact
-----------------

N/A

Other end user impact
---------------------

N/A

Deployer impact
---------------

The installation of etcd will need to be changed to account for this table
being set up.

Developer impact
----------------

N/A

Valence GUI / Horizon impact
----------------------------

N/A


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Nate Potter

Testing
=======

Unit testing will be written alongside the change.


Documentation Impact
====================

Any documentation about the database tables will need to be updated.