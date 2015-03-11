..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================================
Add suspend and resume cluster operations
==============================================================


https://blueprints.launchpad.net/sahara/+spec/cluster-suspend-resume


Once Sahara has deployed a cluster the only possible operation to free
its resources is to delete the cluster. Hence we should provide
additional features to suspend and resume a cluster.


Problem description
===================

The current cluster operations does not include "suspend" and "resume".

Proposed change
===============

The proposed change implies to add these two operations:

+---------+-----------------------------+-------------------------------------+
| Action  | Description                 | Implementation Details              |
+=========+=============================+=====================================+
|         | Suspend all the VMs of the  | engines will suspend all vms of the |
|         | cluster                     | cluster by calling nova.servers.    |
|         | WARN: Make sure suspend     | suspend, if EDP jobs are running    |
| Suspend | doesn't damage the cluster  | they will also be suspended.        |
| cluster |                             | Once these operations completed,    |
|         |                             | the status of the cluster will be   |
|         |                             | updated to 'Suspended'.             |
+---------+-----------------------------+-------------------------------------+
|         | Resume the VMs of the       | engines will resume all vms of the  |
|         | cluster that were suspended | cluster by calling nova.servers.    |
| Resume  |                             | resume, the users will have to      |
| Cluster |                             | relaunch manually their EDP jobs    |
|         |                             | that were canceled during the       |
|         |                             | suspend operation. Status of the    |
|         |                             | cluster will be set to 'Active'.    |
+---------+-----------------------------+-------------------------------------+


Alternatives
------------

None

Data model impact
-----------------

Add resume/suspend types in the db models


REST API impact
---------------

Add API endpoints: /resume and /suspend


Other end user impact
---------------------

None

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

New operations suspend and resume will need to be integrated

Implementation
==============

Assignee(s)
-----------

Primary Assignee:
Pierre Padrixe (stannie)

Work Items
----------

* Add resume/suspend types in the db models
* Add operations in DirectEngine and HeatEngine services
* Add API endpoints: /resume and /suspend
* Add resume/suspend operations in the UI
* Add integration tests
* Update the documentation


Dependencies
============

None

Testing
=======

* Integration tests for the new API endpoints
* Update tests with the new types
* Add requiring unit tests for new operations in each service

Documentation Impact
====================

* New API endpoints need to be documented
* New operations in the UI will induce update of the documentation


References
==========

None
