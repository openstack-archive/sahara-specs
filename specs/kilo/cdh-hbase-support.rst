..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
CDH HBase Support
==========================================

https://blueprints.launchpad.net/sahara/+spec/cdh-hbase-support

This specification proposes to add HBase support for CDH Plugin in Sahara.


Problem description
===================

There is no HBase support in current cdh plugin, but Cloudera Manager
supports to install this service in a cluster. HBase is a non-relational,
distributed database model and can provide BigTable-like capabilities for
Hadoop. This service should be supported in Sahara cdh plugin.


Proposed change
===============

The implementation will support CDH 5.0.0.
Support Features:

* Install HBase processes in a CDH cluster using cm-api
* Zookeeper must be selected and launched in a cluster first
* Support HMaster and Multiple HRegion processes in a cluster
* Most Configuration Parameters Support in a CDH cluster


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

The end users need to select HMaster and HRegion process in node group
templates.

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

It will be required to install the necessary HBase package in the cdh image.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  lu huichun

Other contributors:
  weiting-chen

Work Items
----------

The work items can be divided to several parts:
* Investigate HBase service in a CDH cluster via Cloudera Manager(CM)
* Leverage CM-API Client to call functions to install Zookeep via CM
* Test and Evaluate the Concept
* Implement Source Code in Sahara cdh plugin
* Test the Code

Dependencies
============

Zookeeper process must be installed first in the cluster. HBase needs to use
Zookeeper service.

Testing
=======

Writing Unit Tests to basically test the configuration. It also required to
have an integration test with the cluster creation.

Documentation Impact
====================

Add HBase in the list and add more information about the configuration.

References
==========

* [1] http://www.cloudera.com/content/cloudera/en/documentation/cdh5/v5-0-0/

