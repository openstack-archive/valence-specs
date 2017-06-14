..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Valence scalability
=================================================

https://blueprints.launchpad.net/openstack-valence/+spec/valence-scalability

Problem description
===================
Valence currently has a monolithic design, where the API validates input and
calls appropriate functions from the Redfish library to invoke operations on
the Pod Manager (PODM). Some of the operations that Valence invokes result in 
long running operations, which Valence doesn't track to completion, but returns
a success/failure status for the invocation alone. 
The monolithic design approach, wouldn't scale in the face of new features such
as multi-podm support, Redfish telemetry, etc that will be supported in Valence
in the future. As a result, the current architecture of Valence must be changed
to support the scalability to 1000s of managed entities and to handle the
asynchronous nature of some of the APIs.

Proposed change
===============
The proposed change involves separating the current monolithic Valence service
into micro-services. The services which will be started as a result of this
would be:

    1. Valence API service 
    2. Valence compute service
    3. Valence network service
    4. Valence storage service
    5. New services (such as telemetry) based on future Redfish extensions
    6. Other services (such as a PODM scheduler) internal to Valence used by
	   one ore more of the above services.

 --------------       ------------       ------------
|              |     |            |     |            |
| Valence API  | <=> | Task queue | <=> | Services   |
|              |     |            |     |            |
 --------------       ------------       ------------

For the remainder of the document, we refer to the services listed in items 2
through 6 as functionality services. 

This change in design  introduces an inter-process communication channel
between the API service and the functionality services.

Steps
-----
  1. Create a message queue between the API service and the functionality
  services.
	
  2. Define topics with which the API service will post "tasks" onto the 
  message queue. In order to track the status of the tasks, API layer will 
  generate a task id and insert the task id and initial status as "in progress"
  into the database.
  
  3. Some operations supported by Valence could be long running - such as a
  compose operation, OS installation, telemetry/monitoring  - whereas some
  could return immediately - such as querying system/node properties. In order
  to support both, the API will either initiate synchronous or asynchronous
  task.
    
	1. Synchronous : The API layer will post a synchronous task to the
	functionality service and wait for the	response. Examples: all GET
	requests which query the database or the PODM for information.
	
	2. Asynchronous : The API layer will post an asynchronous task to 
	the functionality service and return immediately to the client after
	creating the task. The task id is unique and can be used for tracking the
	status of the task.
	Asynchronous calls are typically limited for use when invoking PODM API
	which result in a long running operation or to create a long running job.
	This could create an internal thread which either peforms the long running
	job or tracks the status of the long running operation.
	
  4. The functionality services listen on the bus and pick up messages posted
  for their topic. After completing the task, the task status is set to
  "Completed".

Flow
----
The diagram below captures the the flow of a request from the API to the
functional services and return the result.
                                                         ---------
                            ---- Insert task to DB ---> | Database |<---------
                           |                             ---------            |
                        -----                    --------------               |
------------Request--->| API |---Create task--->| Message queue |-----------  |
                        -----                    --------------             | |
                                                                            | |
                         ----------------------			 ---------------    | |
                        | Return API result or |<-------| Functionality |<--  |
                        |start background task |        |  service(s)   |     |
                         ----------------------          --------------       |
                                    |                          |              |
                                    |		                   |              |
                                    V						   |              |
                   ---------------------------                 V              |
                  | Background task completes |--->Update task status in db---
                  | or is terminated		  |
                   ---------------------------				  
Alternatives
--------------
None.

Data model impact
-----------------
This change adds one table to maintain the status of tasks.

REST API impact
---------------
No impact to existing Valence APIs. The following new API for task management
are introduced.

Security impact
---------------
None.

Notifications impact
--------------------
None.

Scalability Impact
------------------
This patch is intended to address scalability shortcomings present in the
current Valence implementation

Other end user impact
---------------------
None.

Performance Impact
-------------------
None.

Other deployer impact
---------------------
None.

Developer impact
----------------
None.

Implementation
==============

Assignee(s)
-----------
Primary assignees:
  Ananth Narayan S <ananth-narayan>
  Mrittika Ganguli <mrittika-ganguli>

Work Items
----------
* Create message queue end points at the API and controller ends.
* Modify API service to post tasks onto the message queue.
* Modify controller end point to receive tasks and initiate action - internal
to Valence or at the PODM.
* Implement test cases.

Dependencies
============
None.

Testing
=======


Documentation Impact
====================
None.

References
==========
None.
