..

This work is licensed under a Creative Commons Attribution 3.0 Unported
License.

http://creativecommons.org/licenses/by/3.0/legalcode

===============================================================================
Create flavors exposing composed infrastructure features to openstack end users
===============================================================================
This proposal describes
  BP link : https://blueprints.launchpad.net/openstack-valence/+spec/valence-openstack-flavor

Problem description
====================

At present openstack cloud makes use of basic config properties described in
flavor while spawning the VM.
Properties exposed through flavors are limited those are ramsize, no of vcpus,
disk size.But cloud users are unaware of additional capabilities that its
compute node possess properties like cpu speed, memory speed, cpu_arch
are unknown as they are currently not exposed through flavors.

Implementation of this feature will enable to create flavors in
openstack cloud through valence. We will be able to create flavors
exposing additional features that
'composed node in valence' or 'compute node in openstack' possess.
We are doing validation of all RSD features that we are exposing in
field 'extra_specs' in flavor creation requestt, while creating flavor.
Cloud end users can make efficient use of additional features of compute node.

Proposed change
===============

To enable valence to add flavor to openstack cloud we will add new
API endpoints. Details of enpoints are as below.

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
It will use details of flavor that is created in valence 'openstack_flavors' db
Flavor details are retrieved using its uuid and call is made
to openstack nova apis to create flavor in openstack cloud

After flavor is created in openstack cloud. Cloud end users can make
use of that flavor.

Below is sample output for all above REST calls:-

1) /v1/openstack_flavors/ (POST)

Request Body:-

::

 {
        "ram":"512",
        "vcpus":"2",
        "name":"f5",
        "disk":"100",
        "extra_specs":{
                       "processor": {
                                      "speed_mhz": "3000"
                                    }
                      }
 }

output:-

::

  {
  "created_at": "2017-04-07 05:28:29 UTC",
  "disk": 100,
  "extra_specs": {
    "processor": {
      "speed_mhz": "3000"
    }
  },
  "name": "f5,
  "ram": 512,
  "updated_at": "2017-04-07 05:28:29 UTC",
  "uuid": "ee0be55d-7be8-44e2-a4ee-5aa1cd347c02",
  "vcpus": 2
  }


In Above API call, we are creating flavor for openstack cloud, which will be stored in valence openstack_flavors database.


2) /v1/openstack_flavors/ee0be55d-7be8-44e2-a4ee-5aa1cd347c02/register (POST)

output:-
"Flavor successfully added to openstack"

In above API call, we have created flavor in openstack cloud


output from openstack cloud after we insert flavor from valence

command:- nova flavor-show f5

output:-
+----------------------------+--------------------------------------+
| Property                   | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 100                                  |
| extra_specs                | {"processor_speed_mhz": "3000"}      |
| id                         | dca462ed-5cd9-4483-a9ec-2ac5ac7ec02c |
| name                       | f5                                   |
| os-flavor-access:is_public | True                                 |
| ram                        | 512                                  |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 2                                    |
+----------------------------+--------------------------------------+


3) /v1/openstack_flavors/ (GET)

output:-

::

  [
  {
    "created_at": "2017-04-07 05:28:29 UTC",
    "disk": 100,
    "extra_specs": {
      "processor": {
        "speed_mhz": "3000"
      }
    },
    "name": "f1",
    "ram": 512,
    "updated_at": "2017-04-07 05:28:29 UTC",
    "uuid": "2d4cf703-22b1-4584-9556-1e45116f22d4",
    "vcpus": 2
  },
  {
    "created_at": "2017-04-07 06:00:29 UTC",
    "disk": 100,
    "extra_specs": {
      "processor": {
        "speed_mhz": "3000"
      }
    },
    "name": "f5",
    "ram": 512,
    "updated_at": "2017-04-07 06:00:29 UTC",
    "uuid": "ee0be55d-7be8-44e2-a4ee-5aa1cd347c02",
    "vcpus": 2
  }
  ]

Above API call we give list of flavors that we have in valence openstac_flavors db

4) /v1/openstack_flavors/2d4cf703-22b1-4584-9556-1e45116f22d4 (GET)

output:-

::

  {
  "created_at": "2017-04-07 05:28:29 UTC",
  "disk": 100,
  "extra_specs": {
    "processor": {
      "speed_mhz": "3000"
    }
  },
  "name": "f1",
  "ram": 512,
  "updated_at": "2017-04-07 05:28:29 UTC",
  "uuid": "2d4cf703-22b1-4584-9556-1e45116f22d4",
  "vcpus": 2
   }

Above API Call will retrive flavor details for given uuid from valence openstack_flavors db

5) /v1/openstack_flavors/2d4cf703-22b1-4584-9556-1e45116f22d4 (DELETE)

output:-
   "Flavor deleted successfully"

Above API call will delete flavor for given uuid from valence openstac_flavors db


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
5) /v1/openstack_flavors/<string:flavoruuid>/register (POST)

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

Versioning impact
-----------------
None

Other end user impact
---------------------
None

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
      5) /v1/openstack_flavors/<string:flavoruuid>/register (POST)
* API implementations

Dependencies
============
python-novaclient needs to be added to projects  requirement

Testing
=======
None

Documentation Impact
====================
update document to include details of openstack flavor create, register,
delete, show APIs

References
==========
None
