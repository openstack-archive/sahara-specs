..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add check service test in integration test
==========================================

https://blueprints.launchpad.net/sahara/+spec/add-service-test-in-integration

This specification proposes to add check services tests in integration test
for CDH plugins.Currently we have added zookeeper,HBase,Flume,Sentry,Sqoop,
SOLR,Key-Value Store Indexer, and Impala services.

Problem description
===================

Currently we have enabled many new services in CDH plugin.And we want to increase
the coverage of the test cases.So we plan to add test cases in the integration test,
which will check the availability of those services by using simple scripts like
we did in map_reduce_testing.


Proposed change
===============

We plan to write test cases like the way we did in map_reduce_testing.First copy
the shell script to the node,then run this script, the script will run basic useage
of the services.

The implementation will need below changes on codes for each service:

* Add new cluster template (including all services process) in test_gating_cdh.py
* Add check_services.py (check all services) to check all basic usage of services
* Add shell scripts (check all services)

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
  huichun lu

Other contributors:
  huichun lu

Work Items
----------

The work items will be:

* Add python codes in sahara/sahara/tests/integration/tests/gating/test_cdh_gating.py.
* Add check services scripts files in sahara/sahara/tests/integrations/tests/resources.
* Add check_services.py in sahara/sahara/tests/integration/tests/.

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
