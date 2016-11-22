
================================
Valence support Multi-PodManager
================================


This proposal adds the ability to dynamically manage pod manager
in valence platform.

BP link : https://blueprints.launchpad.net/openstack-valence/+spec/multi-podm


Problem description
===================
From the architecture level, pod manager now is locked in the config file with
low scalability. This will hinder the development and growth of valence

From the user case level, There would be some problems that we need to face,
for example:

1. Two or more datacenters with different limitations that it's hard to integrate
all the pod managers into a single one. Such as network limited, long distance
, different assets authority and so on.

2. Users may not good at pod managers, maybe they don't know how to
integrate two or more pod managers into a single one. It has a learning cost

3. Deploy more valences is not a good way to manage all the resources

4. It's not convenient to switch different pod managers.

5. Pod manager venders want to sell more pod managers and there were not
only one vender. So users would have more pod managers before user deploy
valence. If we don't support multi-podm management,that would reflect a gap
and user may unwilling to use Valence.

If Valence could support multi-podm management , these problems were be
fixed in time. Also it's the earlier the better to take this into consideration.

Proposed change
===============
Add a Database table to store the pod managers info dynamically as well as
some public api to let admins to do the management operations. Also this 
will effect other features like components inventory and compose operation,
they would be need to specify a exist pod manager.

And this would insert Database into valence. This also be a new change. We can 
use mysql(mariaDB) to sync with OpenStack's other components.

Alternatives
------------
Integrate all the pods into one pod manager.Let users do the integration.
Valence just lock the one pod manager in the config file.

Data model impact
-----------------
Insert pod manager data and their relations between pod manager and
components(chassis, composedNode, etc). Pod manager database schema may be
like this：

| Columns    | type     | description                       |
| :--------: | :------: | :------------------------------   |

| uuid       | varchar  | primary key                       |
| name       | varchar  | pod manager name                  |
| username   | varchar  | auth: username                    |
| password   | varchar  | auth: password                    |
| url        | varchar  | full access path of pod manager   |
| status     | varchar  | pod manager status:online/offline |
| description| varchar  | decribe to pod manager            |
| created_at | datetime | create time                       |
| updated_at | datetime | update time                       |


REST API impact
---------------
Add some new public apis:

```
/v1/pod_manager/<string:podm_uuid> :
GET : get podm info by pod manager's uuid
PUT : update exist podm properties with body : <json:update_items>
DELETE : delete a exist podm by pod managers's uuid
```

```
/v1/pod_managers:
GET : get pod manager list
POST : add a new podm in the platform with body : <json:create_items>
```

Exist api impacts:    
This will change a lot, because pod manager data is the parent data for exist 
components from data structure level.Chassis, rack, composed node are under 
a pod manager. Maybe no need to reflect on the API level, but the implementation 
logics would have a little improvement.

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

Valence GUI / Horizon impact
----------------------------
Pod managers management site may need to be added.


Implementation
==============
Assignee(s)
-----------
Primary assignee:
  Bian.Hu

Other contributors:
  Andy Yan

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

