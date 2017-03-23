
====================================
Refactor to support Multi-PodManager
====================================


This proposal of refactoring the codes to support multi-PodManager in valence
platform fully.

BP link : https://blueprints.launchpad.net/openstack-valence/+spec/refactor-to-support-multi-podmanager


Problem description
===================

As multi pod manager logic service seems ready in valence , but the valence api
and logic were still focus on the locked on pod manager base on the options in
the config file. So this would be removed and changed.

Also now our controller api and redfish.py are take effect on the locked podm.
This also need to abstract them out to support multi-podmanager.

So we would take a refactor on the code to make this change.


Proposed change
===============

Remove the options of pod manager in the config file. And get the pod manager
from the database if needed.

Bring up the relations between pod manager and components if need , like nodes
and systems in database. Maybe add a column 'podm_uuid' in node's db schema.

Change or rename 'valence.redfish.redfish.py' to be a 'pod_manager_client.py'
or 'pod_manager_connection.py' that provide all the needed functions of podmanager
by input or specify a podm connection properties which coming from database.
(Seems like the db.api.db_connection that we input connection properties then
call the db functions)

Then we could customize a connection pool (local cache) for pod managers to make
a better performance.

Also, we could abstract out a 'http_client.py' to provide all the http request
methods for pod manager client to implement its functions. Like this:
https://review.openstack.org/#/c/445360/2/valence/common/http_adapter.py

So this whole logic could be changed like this:
Controller get a pod manager connection from local cache.
Then the connection provide the functions of the pod manager.
The functions would call 'http_client.py' to real pod manager service.

Alternatives
------------
Integrate all the pods into one pod manager.Let users do the integration.
Valence just lock the one pod manager in the config file.

Data model impact
-----------------
Insert a column 'podm_uuid' into model nodes' db schema.

REST API impact
---------------
This refactor would do a little change to exist apis , such as:

```
/v1/nodes/:
POST : add a new param to let user specify a exist podm to compose a new node.
GET : add a new param to do filter by specify a podm.
```

```
/v1/podmanagers/:
GET : add some usage info in returns to help user to do the selection.
```

```
/v1/podmanager/{uuid}:
DELETE : add some check logic to make save-deletion operation. Like we should
         ensure all the nodes which created by valence are released when we do
         the deletion.
```

```
/v1/systems/:
GET : returns all podms' systems not only the systems of locked one podm now.
GET : add a new param to do filter by specify a podm.
```

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
This would change the way how pod manager registered into valence. We would
register pod managers in frontend or CLI after Valence service started, instead
of configuring them in config file.

Developer impact
----------------
This will change a lot, because pod manager data is the parent data for exist
components from data structure level.Chassis, rack, composed node are under
a pod manager. And some component should have a column to show its belonging
pod manager.

Valence GUI / Horizon impact
----------------------------
Pod managers management site may need to be added and improved, as well as the
compos node apply forms should add a drop-down-menu to specify a podm.


Implementation
==============
Assignee(s)
-----------
Primary assignee:
  Bian.Hu

Other contributors:
  Mao Haijun


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


References
==========
None
