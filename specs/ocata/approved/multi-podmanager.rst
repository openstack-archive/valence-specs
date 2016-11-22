
================================
Valence support Mutli-PodManager
================================


This proposal adds the ability to dynamicly manage pod manager
in valence platform.

BP link : https://blueprints.launchpad.net/openstack-valence/+spec/muti-podm


Problem description
===================
From the architecture level, pod manager now is locked in the config file with
low scalability. This will hinder the development and growth of valence

From the user case level, There would be some problems that we need to face,
for example:

1. Two or more datacenters with different limited that it's hard to integrate 
all the pod managers into a single one. Such as network limited, long distance
, different assets authority and so on.

2. Users may not good at pod managers, maybe they don't know how to
integrate two or more pod managers into a singal one. It has a learning cost

3. Deploy more valences is not a good way to manage all the resources

4. It's not convenient to switch different pod managers.

5. Pod manager venders want to sell more pod managers and there were not 
only one vender. So users would have more pod managers before user deploy
valence. If we don't support multi-podm management,that would reflect a gap 
and user may unwilling to use Valence.

If Valence could support multi-podm management , these problems were be
fixed in time. Also it's the earlier the better to take this into consideration
.

Proposed change
===============
Add a Database table to stor the pod managers info dynamicly as well as 
some public api to let admins to do the management operations. Also this 
will effect other features like components inventory and compose operation,
they would be need to specify a exist pod manager.

And this would insert Database into valence. This also be a new change.


Alternatives
------------
None


Data model impact
-----------------
Insert pod manager data and their relations between pod manager and
components(chassis, composedNode, etc) Pod manager database schema may be 
like this：

| Columns       | type        | description |
| :--------:| :--------------: | :----------|   

| id | int(11) | primary key |
| name | varchar | pod manager name |
| ip | varchar | ip address |
| username | varchar | auth: username |
| password | varchar | auth: password |
| url | varchar | pod manager url |
| status | varchar | pod manager status:online/offline |
| description | varchar | decribe to pod manager |
| created_at | datetime | create time |
| updated_at | datetime | update time |


REST API impact
---------------
Add some new public apis:

```
/api/pod_manager :
GET : get podm info with a param : <pod_manager_id>
PUT : update podm properties with body : <json:update_items>
POST : add a new podm in the platform with body : <json:create_items>
DELETE : delete a exist podm with a param : <pod_manager_id>

```

```
/api/pod_manager/list:
GET : get pod manager list

```

Exist api impacts:     
This will change a lot, because pod manager data is the parent data for exist
 components from data structure level.Chassis, rack, composed node are under 
a pod manager. So those component listing api and compose operation would 
take pod manager into consideration.For example:

```
GET /v1/nodes: could change to /v1/`pod_manager_id`/nodes or 
/v1/nodes?pod_manager_id=`pod_manager_id`
POST /v1/node: could change to /v1/`pod_manager_id`/node

```

etc ...
 

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
Much improve valence's scalability 

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
  None

Work Items
----------
* DB insert and table create, import sqlalchemy and other required lib.
* API definition，urls design.
* API implementation
* Fix the gap for those effected api


Dependencies
============
None

Testing
=======
* Unit tests: Mocking Pod manager library.

Documentation Impact
====================
* TODO

References
==========
None

