
=======================================
Valence Web UI support Multi-PodManager
=======================================


This proposal adds the ability to dynamically manage pod manager
in valence web UI.

BP link : https://blueprints.launchpad.net/openstack-valence/+spec/web-ui-functionality


Problem description
===================
From web UI perspective, multi-podmanager is implemented in valence.

From the user case level, there would be some problems that we need to face,
for example:

1. Users should be able to select one or more podmanagers out of multi-podmanagers,
otherwise, they will always get all components for all podmangers.

2. Users should be able to register one or more podmanagers if there are new podmanagers

Proposed change
===============
Add a panel in web UI to show multi-podmanagers info dynamically to let admins to
do the management operations. Also this
will effect other features like components inventory and compose operation,
they should be need to switch to a specific podmanager or some podmanagers.

Alternatives
------------
None

Data model impact
-----------------
None

REST API impact
---------------
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
None

Developer impact
----------------
None

Valence GUI / Horizon impact
----------------------------
Pod managers management pages should be added.
One proposal for Multi-pod manager can look like below snapshots:
https://wiki.openstack.org/wiki/File:Valence-pod_manager_1.png
https://wiki.openstack.org/wiki/File:Valence-pod_manager_2.png
When one pod manager is selected , only racks/chassis belonged to this pod manager
need to be displayed.
Ref: https://blueprints.launchpad.net/openstack-valence/+spec/web-ui-functionality

Implementation
==============
Assignee(s)
-----------
Primary assignee:
  Andy Yan

Other contributors:
  Yang Xing
  Mao Haijun
  Lychee

Work Items
----------
* Implement functionaliy to switch between multi-podmanagers.
* Implement functionaliy to show accordingly different components under different podmanagers.

Dependencies
============
None

Testing
=======
Web UI testing.

Documentation Impact
====================
None

References
==========
None
