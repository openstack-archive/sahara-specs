..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
Plugin for Sahara with MapR
===========================

https://blueprints.launchpad.net/sahara/+spec/mapr-plugin

This specification proposes to add MapR plugin with MapR Distribution of
Hadoop in Sahara.

Problem description
===================

The MapR Distribution for Apache Hadoop provides organizations with an
enterprise-grade distributed data platform to reliably store and process
big data. MapR packages a broad set of Apache open source ecosystem
projects enabling batch, interactive, or real-time applications. The
data platform and the projects are all tied together through an advanced
management console to monitor and manage the entire system.

MapR is one of the largest distributions for Hadoop supporting more than
20 open source projects. MapR also supports multiple versions of the
various individual projects thereby allowing users to migrate to the latest
versions at their own pace. The table below shows all of the projects
actively supported in the current GA version of MapR Distribution for
Hadoop as well as in the next Beta release.[1]


Proposed change
===============

MapR plugin implementation will support  Hadoop 0.20.2 and Hadoop 2.4.1.
Plugin will support key Sahara features:

* Cinder integration
* Cluster scaling/decommission
* EDP
* Cluster topology validation
* Integration with Swift


Plugin will be able to install following services:

* MapR-FS
* YARN
* Oozie (two versions are supported)
* HBase
* Hive (two versions are supported)
* Pig
* Mahout
* Webserver

MapR plugin will support the following OS: Ubuntu 14.04 and CentOS 6.5.

MapR plugin will support the following node types:

* Nodes Running ZooKeeper and CLDB
* Nodes for Data Storage and Processing
* Edge Nodes

In a production MapR cluster, some nodes are typically dedicated to cluster
coordination and management, and other nodes are tasked with data storage
and processing duties. An edge node provides user access to the cluster,
concentrating open user privileges on a single host.

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

MapR plugin uses specific pre-installed images with
MapR local repository files.

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  aosadchiy

Other contributors:
  ssvinarchuck

Work Items
----------

* Add implementation of plugin for bare images.

Dependencies
============

Depends on OpenStack requirements.

Testing
=======

* Add unit tests to Sahara to cover basic functionality of plugin
* Add integration tests to Sahara

Documentation Impact
====================

MapR plugin documentation should be added to the plugin section of Sahara docs.

References
==========

* [1] https://www.mapr.com/products/apache-hadoop
* [2] http://doc.mapr.com/display/MapR/Home
* [3] https://www.mapr.com/
* [4] https://github.com/mapr
