This work is licensed under a Creative Commons Attribution 3.0 Unported License.

http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================================
Create flavors exposing composed infrastructure features to openstack cloud users
=================================================================================
This proposal describes
  BP link : https://blueprints.launchpad.net/openstack-valence/+spec/valence-openstack-flavor
  
Problem description
====================

At present openstack cloud makes use of basic config properties described in flavor while spawning the VM.
Properties exposed through flavors are limited those are ramsize, no of vcpus, disk size. 
But cloud users are unaware of additional capabilities that its compute node possess.
properties like cpu speed, memory speed, cpu_arch are unknown as they are currently not exposed through flavors.

Implementation of this feature will enable to create flavors in openstack cloud through valence.
We will be able create flavors exposing additional features that (composed node in valence)/(compute node in openstack) possess.
Cloud users can make efficient use of additional features of compute node.
 
Proposed change
===============

To enable valence to add flavor to openstack cloud we will add new API endpoints.
Details of enpoints are as below.

1. Add new valence API /v1/openstack_flavors/ (POST)
This api will be used to create new flavors for openstack, 
which will be stored in valence 'openstack_flavors' db

2. Add new valence API /v1/openstack_flavors/ (GET)
This api will be used to to view all flavors in valence 'openstack_flavors' db

3. /v1/openstack_flavors/<string:flavoruuid> (DELETE)
This api will be used to delete flavor from valence 'openstack_flavors' db

4. /v1/openstack_flavors/<string:flavoruuid> (GET)
This api will be used to view openstack flavor details

5. /v1/openstack_flavors/<string:flavoruuid>/register (POST)
This api will create label in openstack cloud.
It will use details of flavor that is created in valence 'openstack_flavors' db.
Flavor details are retrieved using its uuid and call is made 
to openstack nova apis to create flavor in openstack cloud

After flavor is created in openstack cloud.Cloud users can make use of that flavor.

Alternatives
------------
None

Data model impact
-----------------
Insert a new model 'OpenstackFlavor' and new db 'openstack_flavors'.

REST API impact
---------------
Add new REST API's

1) /v1/openstack_flavors/ (GET)
2) /v1/openstack_flavors/ (POST)
3) /v1/openstack_flavors/<string:flavoruuid> (DELETE)
4) /v1/openstack_flavors/<string:flavoruuid> (GET)
5) /v1/openstack_flavors/<string:flavoruuid>/register	(POST) 

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
None


Implementation
==============
Assignee(s)
-----------
Primary assignee:
  Raghavendra Umrikar

Work Items
----------
* Update valence.conf with openstack cloud credentials
* Create conf file for openstack(valence/conf/openstack.py) 
  to register openstack opts
* Create new model class OpenstackFlavor, add new db 'openstack_flavors'
* Add new REST APIs 
      1) /v1/openstack_flavors/ (GET)
	  2) /v1/openstack_flavors/ (POST)
      3) /v1/openstack_flavors/<string:flavoruuid> (DELETE)
	  4) /v1/openstack_flavors/<string:flavoruuid> (GET)
      5) /v1/openstack_flavors/<string:flavoruuid>/register	(POST) 	  
* API implementations
 
Dependencies
============
python-novaclient needs to be added to projects  requirement

Testing
=======
None

Documentation Impact
====================
update document to include details of openstack flavor create, register, delete, show APIs

References
==========
None
