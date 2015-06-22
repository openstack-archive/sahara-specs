..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Running Spark Jobs on Cloudera Clusters 5.3.0
=============================================

https://blueprints.launchpad.net/sahara/+spec/spark-jobs-for-cdh-5-3-0

This specification proposes to add ability to run Spark Jobs on clusters with
CDH (Cloudera Distribution Including Apache Hadoop).


Problem description
===================

Sahara is able to run CDH clusters with running Spark services. However there
was no possibility to run Spark jobs on clusters of this type.


Proposed change
===============

The work involves adding a class for running Spark job via Cloudera plugin.
Existing Spark engine was changed so that it lets to run Spark jobs with Spark
and Cloudera plugins.

Alternatives
------------

Do nothing.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

Required processes:
- Master: SPARK_YARN_HISTORY_SERVER
- Workers: YARN_NODEMANAGER

Deployer impact
---------------

None.

Developer impact
----------------

None.

Sahara-image-elements impact
----------------------------

None.

Sahara-dashboard / Horizon impact
---------------------------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Alexander Aleksiyants

Other contributors:
  Oleg Borisenko

Work Items
----------

* https://review.openstack.org/#/c/190128/


Dependencies
============

None.

Testing
=======

* Unit tests to cover CDH engine for working with Spark jobs.
* Unit tests for EDP Spark is now used for Spark Engine and EDP engine.

Documentation Impact
====================

None.

References
==========

None.
