..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add More Services into CDH plugin
==========================================

https://blueprints.launchpad.net/sahara/+spec/add-cdh-more-services

This specification proposes to add below services into CDH plugins:
Flume, Sentry, Sqoop, SOLR, Key-Value Store Indexer, and Impala.

Those services can be added into CDH plugin by using CM API first_run to start
them, which can save much effort to prepare and start those services one by
one.

Problem description
===================

Now services supported in Sahara CDH plugin is still limited. We want to add
more services ASAP. They are Flume, Sentry, Sqoop, SOLR, Key-Value Store
Indexer, and Impala.

Proposed change
===============

Since we plan to use first_run to prepare and start those services, we will
not need to call other CM APIs for those services in start_cluster() method.

The implementation will need below changes on codes for each service:

* Add process names of the service in some places.
* Add service or process configuration, and network ports to open.
* Add service validation.
* Modify some utils methods, like get_service, to meet more services.
* Some other changes for a few specific services if needed.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

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

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ken chen

Other contributors:
  ken chen

Work Items
----------

The work items will be:

* Change python codes in sahara/sahara/plugins/cdh.
* Add more service resource files in sahara/sahara/plugins/cdh/resources.
* Test and evaluate the change.

Dependencies
============

None

Testing
=======

Use an integration test to create a cluster.

Documentation Impact
====================

None

References
==========

None
