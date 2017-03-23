
==========================
Valence Ironic Integration
==========================
This proposal tries to explain the integration of projects Valence and
OpenStack Baremetal Service, Ironic.

Problem description
===================
Valence allows to compose nodes out of a pool of disaggregated hardware.
And Ironic allows to provision bare metal machines. Once the node is
composed by Valence that can be registered to Ironic to be consumed.
So this feature aims on providing a way to integrate Valence and Ironic,
so that users will be able to provision bare metal machines on the nodes
composed by Ironic.

Proposed change
===============
To enable Valence to be able to register its composed node to Ironic, we
will add a new API endpoint /v1/nodes/{ID}/register. The detailed explanation
of this API is as below:
 
1. Add a new Valence /v1/nodes/{ID}/register API
This introduces a new nodes API in valence that will be used to register a
composed node in Ironic. Below steps explains a detailed flow of registering
a node from Valence to Ironic.
 
Step 1: User calls Valence APIs to register a node.

    $ valence node-register test-node
 
    This API will register the test-node to Ironic.

Step 2:	Valence calls Ironic API internally to register this node which is equivalent to:

    $ ironic node-create –d redfish –i redfish_root_uri <Redfish URL> -i refish_username <username> ...
 
    This will create a new node in Ironic with all the required details related to Redfish.

After the node is succesfully registered to Ironic, users can use the Ironic or Nova to install
an OS on the node.


Alternatives
------------
None


Data model impact
-----------------
None


REST API impact
---------------
1. Add a new API endpoint /v1/node/{ID}/register

Driver API impact
-----------------
1. Add a new driver that can be used to interact with ironic.


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
  Madhuri Kumari

Work Items
----------
* Add a driver in Valence to interact with Ironic.
* Add new API endpoint /v1/node/{ID}/register.
* Update api-ref to include the new API.

Dependencies
============
Depends on Ironic project. python-ironicclient needs to be
added to the project's requirement.

Testing
=======
* Each patch will be accompanied with unit testcases.

Documentation Impact
====================
None

References
==========
None
