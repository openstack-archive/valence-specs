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

+-----------------------------------------------------------+
|                                                           |
|   +--------------+     +------------+     +------------+  |
|   | Valence API  | <=> | Task queue | <=> | Services   |  |
|   + -------------+     +------------+     +------------+  |
|                                                           |
+-----------------------------------------------------------+


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
  generate a task id, mark the task status as "Created" and insert the 
  information into the database. Once the task starts, its status is changed to
  "In Progress".
  
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
  "Completed". If the task fails, the status is set to "Failed".

Flow
----
The flow is 
   1. The API service receives a request
   2. API layer create a task
      a. It posts the task into the message queue.
	  b. It inserts the details of the task into the database. 
   3. One of the functionality services which are subscribing to the message
   queue picks up the task. This is determined by the topic.
   4. Based on the nature of the request, the API either initiates a
   synchronous or asynchronous task.
      a. In case of synchronous task, the functionality service will perform
	  the operation and return result. The state of the task is updated in the
	  database.
	  b. In case of an asynchronous task, the functionality service will start
	  a background thread.
	5. When the background task completes or is terminated, the status is
	updated in the database.

Alternatives
--------------
None.

Data model impact
-----------------
This change adds one table to maintain the status of tasks.
+----------------+-------------------------------------------------------+
| Field          | Description                                           |
+================+=======================================================+
| uuid           | Task UUID                                             |
+----------------+-------------------------------------------------------+
| status         | Status of the task.                                   |
+----------------+-------------------------------------------------------+
| request        | API Request which result in the task being created    |
+----------------+-------------------------------------------------------+
| request_body   | Request body.                                         |
+----------------+-------------------------------------------------------+
| failure_reason | Failure reason if the task did not successfully       |
|                | complete. Null otherwise.                             |
+----------------+-------------------------------------------------------+

REST API impact
---------------
No impact to existing Valence APIs. The following new API for task management
are introduced.

1) /v1/tasks (GET)
   Get list of tasks (from tasks table in database)
   
   REST api call:- /v1/tasks (GET)
   
   output:-
::
        [
		  {
			"created_at": "2017-06-21 08:34:10 UTC",
			"request": "compose_node",
			"request_body": {
			  "request_params": [
				{
				  "description": "test1",
				  "name": "test1"
				}
			  ]
			},
			"status": "Created",
			"updated_at": "2017-06-21 08:34:10 UTC",
			"uuid": "0c739e2a-17a1-47b2-bd83-8f4faa027c90"
		  },
		  {
			"created_at": "2017-06-21 08:34:03 UTC",
			"request": "compose_node",
			"request_body": {
			  "request_params": [
				{
				  "description": "test2",
				  "name": "test2"
				}
			  ]
			},
			"status": "Created",
			"updated_at": "2017-06-21 08:34:03 UTC",
			"uuid": "9c6cb331-e6b9-4b83-9327-56ef4d0c2b74"
		  }
        ]
   

2) /v1/tasks/<string:task_uuid> (GET)
   Gets task with mentioned task_uuid (from tasks table in database)
   
   REST api call:- /v1/tasks/9c6cb331-e6b9-4b83-9327-56ef4d0c2b74 (GET)
   
   output:-

::
	      {
			"created_at": "2017-06-21 08:34:03 UTC",
			"request": "compose_node",
			"request_body": {
			"request_params": [
			  {
				"description": "test2",
				"name": "test2"
			  }
			]
			},
			"status": "Created",
			"updated_at": "2017-06-21 08:34:03 UTC",
			"uuid": "9c6cb331-e6b9-4b83-9327-56ef4d0c2b74"
		  }

   
3) /v1/tasks/<string:task_uuid> (DELETE)
    Deletes task with task_uuid (from tasks table in database)
	
	REST api call:- /v1/tasks/9c6cb331-e6b9-4b83-9327-56ef4d0c2b74 (DELETE)

	output:-
	
::
	     {
		  "code": "DELETED",
		  "detail": "This task 9c6cb331-e6b9-4b83-9327-56ef4d0c2b74 has been deleted successfully",
		  "request_id": "00000000-0000-0000-0000-000000000000"
         }  
		 
		 

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
