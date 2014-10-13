..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Plugin for CDH with Cloudera Manager
====================================

https://blueprints.launchpad.net/sahara/+spec/cdh-plugin

This specification proposes to add CDH plugin with Cloudera Distribution of
Hadoop and Cloudera Manager in Sahara.

Problem description
===================

Cloudera is open-source Apache Hadoop distribution, CDH (Cloudera Distribution
Including Apache Hadoop). CDH contains the main, core elements of Hadoop that
provide reliable, scalable distributed data processing of large data sets
(chiefly MapReduce and HDFS), as well as other enterprise-oriented components
that provide security, high availability, and integration with hardware and
other software. Cloudera Manager is the industry's first end-to-end management
application for Apache Hadoop. Cloudera Manager provides many useful features
for monitoring the health and performance of the components of your cluster
(hosts, service daemons) as well as the performance and resource demands of
the user jobs running on your cluster. [1]

Proposed change
===============

CDH plugin implementation will support Cloudera Manager version 5 and CDH
version 5.

Plugin will support key Sahara features:

* Cinder integration
* Cluster scaling
* EDP
* Cluster topology validation
* Integration with Swift
* Data locality

Plugin will be able to install following services:

* Cloudera Manager
* HDFS
* YARN
* Oozie

CDH plugin will support the following OS: Ubuntu 12.04 and CentOS 6.5.
CDH provisioning plugin will support mirrors with CDH and CM packages.

By default CDH doesn't support Hadoop Swift library. Integration with Swift
should be added to CDH plugin. CDH maven repository contains Hadoop Swift
library. [2]

CDH plugin will support the following processes:

* MANAGER - Cloudera Manager, master process
* NAMENODE - HDFS NameNode, master process
* SECONDARYNAMENODE - HDFS SecondaryNameNode, master process
* RESOURCEMANAGER - YARN ResourceManager, master process
* JOBHISTORY - YARN JobHistoryServer, master process
* OOZIE - Oozie server, master process
* DATANODE - HDFS DataNode, worker process
* NODEMANAGER - YARN NodeManager, worker process

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

CDH plugin must be support vanilla images and images with Cloudera packages.
For building pre-installed images with Cloudera packages use specific CDH
elements.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  sreshetniak

Other contributors:
  iberezovskiy

Work Items
----------

* Add implementation of plugin
* Add jobs in Sahara-ci
* Add integration tests
* Add elements to Sahara-image-elements for building images with pre-installed
  Cloudera packages

Dependencies
============

Depends on OpenStack requirements, needs a `cm_api` python library version
6.0.2, which is not present in OS requirements. [3] Need to add `cm_api` to
OS requirements. [4]

Testing
=======

* Add unit tests to Sahara to cover basic functionality of plugin
* Add integration tests to Sahara

Documentation Impact
====================

CDH plugin documentation should be added to the plugin section of Sahara docs.

References
==========

* [1] http://www.cloudera.com/content/cloudera/en/products-and-services/cdh.html
* [2] https://repository.cloudera.com/artifactory/repo/org/apache/hadoop/hadoop-openstack/
* [3] https://pypi.python.org/pypi/cm-api
* [4] https://review.openstack.org/#/c/106011/
