..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Multiple PodManager Scheduler
=============================

This proposal describes adding new scheduler service into valence to determine
how to dispatch compose operation to the appropriate Pod manager.

https://blueprints.launchpad.net/openstack-valence/+spec/valence-multipodm-scheduler

Problem description
===================

Valence will support multiple Pod managers on the backend instead of one single
instance to improve its scalability. It requires valence to provide a scheduling
service to determine how to dispatch each compose operations on the appropriate
Pod manager. The scheduler should filter out the inappropriate Pod Manager
without requested hardware resource and rank the priority for the remaining Pod
manager with different algorithms. For different scheduling goal, it should
allow admin to plugin new algorithms.

Proposed change
===============

The valence scheduler runs as a separate process alongside the other valence
services such as the API server. Its interface to the API server is accepting
the request proprieties of each compose operation, and it does a posts to
controller to indicate where the composition should be scheduled.

The scheduler is divided into two layers from high level:
- Scheduler framework:
The main() entry that does service initialization and calls the scheduler
algorithm.
- Scheduling algorithm:
The scheduling algorithm that assigns target Pod manager for each compose
operation.

The Scheduler tries to find a PODM for each compose operation, one at a time.
- First it applies a set of "filter functions" to filter out inappropriate
nodes. If the compose operation specifies resource requests, then the scheduler
will filter out PODM that don't have at least that much resources available.
- Second, it applies a set of "priority functions" that rank the PODM that
weren't filtered out in the first step. The "priority functions" may vary for
different scenarios. For example, it tries to spread all composed node across
all PODM.
- Finally, the PODM with the highest priority is chosen. If there are multiple
such PODM, then one of them is chosen at random.

For given compose operations::

    +---------------------------------------------+
    |               Schedulable PODM:             |
    |                                             |
    | +--------+    +--------+      +--------+    |
    | | PODM 1 |    | PODM 2 |      | PODM 3 |    |
    | +--------+    +--------+      +--------+    |
    |                                             |
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+
     Filters function: PODM 3 doesn't have enough
                       resource
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+
    |             remaining PODM:                 |
    |   +--------+                 +--------+     |
    |   | PODM 1 |                 | PODM 2 |     |
    |   +--------+                 +--------+     |
    |                                             |
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+
     Priority function: PODM 1: p=5
                        PODM 2: p=3
    +-------------------+-------------------------+
                        |
                        |
                        v
           select max{PODM priority} = PODM 1

Both filters function and Priority function should be configurable to allow
admin to choose proper algorithm for different scenarios, like disable all
algorithms and let scheduler randomly choose one.

Alternatives
------------

Make scheduler as a valence module instead of standalone service. This solution
will be more simple but tight couple with other services, which will bring more
overhead if scheduler service need to be upgraded or restarted.

Data model impact
-----------------
None

REST API impact
---------------
Be default, scheduler will determine the target POD manager for each compose
operation. However, valence should also allow user to specify the target POD
manager. So a new parameter is needed for node composition request.

```
/v1/nodes/:
POST : add a new param to let user specify a POD manager for compose operation.
```

Driver API impact
-----------------
None

Security impact
---------------
None

Other end user impact
---------------------
User can specify the target POD manager for compose operation if needed.

Scalability impact
------------------
The valence scalability will be significantly improved by supporting dispatch
compose operations on multiple POD manager.

Performance Impact
------------------
The scheduler will bring more complexity and overhead, which might add
latency into valence response one compose operation. Given the compose
operations on the data center will not be so frequently as launch VM/continer,
so the scheduler will not be the performance bottleneck in the current stage.

Other deployer impact
---------------------
The admin should deploy and start scheduler process alongside other valence
services.

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
  Lin Yang

Work Items
----------
* Implement the framework of scheduler service.
* Implement the default algorithms for both filter and priority steps.
* Add unit tests.

Dependencies
============
None

Testing
=======
* Add unit tests for service framework and scheduling algorithms.

Documentation Impact
====================
None

References
==========
None
