..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
CDH Zookeeper Support
==========================================

https://blueprints.launchpad.net/sahara/+spec/cdh-zookeeper-support

This specification proposes to add Zookeeper support for CDH Plugin in Sahara.


Problem description
===================

Currently, Zookeeper isn't supported in the cdh plugin. Zookeeper is a
centralized service to provide functions like maintaining configuration,
distributed synchronization, and providing group services. It's important to
have a Zookeeper to prevent data loss and to avoid a single point failure(SPoF)
in a cluster. It has become a basic service to deploy a hadoop cluster in CDH
environment.


Proposed change
===============

The implementation will support CDH 5.0.0.
Support Features:

* Install Zookeeper Service in a CDH cluster
* Support to run Standalone Zookeeper in a cluster
* Support to run Replicated Zookeepers(Multiple Servers) in a cluster
* To have an option to select Zookeeper in the node group templates
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

The end users need to select Zookeeper process in their node group templates.

Deployer impact
---------------

None

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

It will be required to put the necessary packages in the cdh image.

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
  weiting-chen

Work Items
----------

The work items can be divided to several parts:

* Investigate Zookeeper service in a CDH cluster via Cloudera Manager(CM)
* Leverage CM-API Client to call functions to install Zookeep via CM
* Test and Evaluate the Concept
* Implement Source Code in Sahara cdh plugin
* Test the Code

Dependencies
============

None

Testing
=======

Writing Unit Tests to basically test the configuration. It is also required
to have an integration test with the cluster creation.

Documentation Impact
====================

Add Zookeeper in the list and add more information about the configuration.

References
==========

* [1] http://www.cloudera.com/content/cloudera/en/documentation/cdh5/v5-0-0/

