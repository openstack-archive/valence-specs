
===================================
OpenStack Ironic Plugin for Valence
===================================

This proposal adds the connection between ironic and valence.

BP link:https://blueprints.launchpad.net/openstack-valence/+spec/openstack-components-plugin

Problem description
===================
From OpenStack's framework level, valence now is a independent project of OpenStack,
too independent to connect with other OpenStack components. This will bring a gap to OpenStack
users as well as the community.

From valence roadmap's level, valence has a great available space to OpenStack especially Ironic!
They could have a awesome collaborative work for users on dataceners resource management. But it
is still lack of the connections.

Setup the connection and cover the roadmaps on both side will bring a profound influence.

Proposed change
===============
We should have to setup a driver for valence in ironic that has already supported by ironic for
users to insert their own customized driver. The driver can let ironic communicate with other
service. So we would setup a valence driver in Ironic.

And this is a big picture if we base on the base driver defined by Ironic itself. So i suggest
that we should treat this spec as the parent spec, then split the whole plugin into small parts
and child spec. Then we could schedue the priority of each part to match valence's growth.

For example:
BaseDriver of ironic should include these driver properties:
* power
* deploy
* console
* management
* boot
* vendor
* inspect
* raid
* Other extend properties like pxe

Not all these properties are required for valence, we just set priority for required property.
Then contributors write child spec for each property and we would disscuss and contribute there.

For example, shuquan has writen a child spec "pxe-rsd driver for ironic" link :
https://review.openstack.org/#/c/401192/.

Alternatives
------------
None


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

Implementation
==============
Assignee(s)
-----------
Primary assignee:
  Bian.Hu

Other contributors:
  Andy yan
  Yang Xing
  Mao HaiJun
  Wang Zhandong
  Zhang Xiaotong

Work Items
----------
* Setup a new plugin driver instance for valence in ironic
* Match valence roadmap to figure out required driver properties
* Implementation on the required property's features

Testing
=======
* Unit tests: Mocking valence library.
* Ironic commands: driver commands usage test

Documentation Impact
====================
* TODO

References
==========
None

