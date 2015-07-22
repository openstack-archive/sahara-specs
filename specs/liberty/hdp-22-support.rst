..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Add support HDP 2.2 plugin
==========================

https://blueprints.launchpad.net/sahara/+spec/hdp-22-support

This specification proposes to add new HDP plugin based on Ambari
Blueprints [1] with Ambari Management Console.

Problem description
===================

Currently we support old HDP plugin which contains old HDP distribution.
Also old HDP plugin looks like unsupported by HortonWorks team every year [2].
Many customers want new version of HDP. New HDP plugin will be based on Ambari
Blueprints. Ambari Blueprints are a declarative definition of a cluster.
With a Blueprint, you specify a Stack, the Component layout and
the Configurations to materialize a Hadoop cluster instance via REST API.

Proposed change
===============

New HDP plugin will support provisioning HDP stack via Ambari Blueprints.

Plugin will support key Sahara features:

* Cinder integration
* Swift integration
* EDP
* Scaling
* Event logs

New HDP plugin will support the following OS: Ubuntu 12.04 and CentOS 6. Aslo
new plugin will support mirrors with HDP packages.

New HDP plugin will support all services which supports Ambari. Also new plugin
will support HA for NameNode and ResourceManager. Client will be installed on
all nodes if selected our process. For example if selected Oozie then will be
installed Oozie client on all nodes.

Plugin wil be support the following services:

+-------------------+---------------------------+
| Service           | Process                   |
+===================+===========================+
| Ambari            | Ambari                    |
+-------------------+---------------------------+
| Falcon            | Falcon Server             |
+-------------------+---------------------------+
| Flume             | Flume                     |
+-------------------+---------------------------+
| HBase             | HBase Master              |
|                   +---------------------------+
|                   | HBase RegionServer        |
+-------------------+---------------------------+
| HDFS              | NameNode                  |
|                   +---------------------------+
|                   | DataNode                  |
|                   +---------------------------+
|                   | SecondaryNameNode         |
|                   +---------------------------+
|                   | JournalNode               |
+-------------------+---------------------------+
| Hive              | Hive Metastore            |
|                   +---------------------------+
|                   | HiveServer                |
+-------------------+---------------------------+
| Kafka             | Kafka Broker              |
+-------------------+---------------------------+
| Knox              | Knox Gateway              |
+-------------------+---------------------------+
| Oozie             | Oozie                     |
+-------------------+---------------------------+
| Ranger            | Ranger Admin              |
|                   +---------------------------+
|                   | Ranger Usersync           |
+-------------------+---------------------------+
| Slider            | Slider                    |
+-------------------+---------------------------+
| Spark             | Spark History Server      |
+-------------------+---------------------------+
| Sqoop             | Sqoop                     |
+-------------------+---------------------------+
| Storm             | DRPC Server               |
|                   +---------------------------+
|                   | Nimbus                    |
|                   +---------------------------+
|                   | Storm UI Server           |
|                   +---------------------------+
|                   | Supervisor                |
+-------------------+---------------------------+
| YARN              | YARN Timeline Server      |
|                   +---------------------------+
|                   | MapReduce History Server  |
|                   +---------------------------+
|                   | NodeManager               |
|                   +---------------------------+
|                   | ResourceManager           |
+-------------------+---------------------------+
| ZooKeeper         | ZooKeeper                 |
+-------------------+---------------------------+

Alternatives
------------

Add support of HDP 2.2 in old plugin, but it is very difficult to do without
Ambari Blueprints.

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

Need to add elements for building images with pre-installed Ambari packages.
For installing HDP Stack plugin should use mirror with HDP packages. Also
should add elements for building local HDP mirror.

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
  nkonovalov

Work Items
----------

* Add base implementation of plugin [3] [4]
* Add elements for building image with Ambari [5]
* Add EDP support [6]
* Add additional services support [7]
* Add scaling support [8]
* Add HA support [9]
* Add elements for building HDP mirror [10]

Dependencies
============

None


Testing
=======

* Add unit tests for plugin
* Add scenario tests and job on sahara-ci


Documentation Impact
====================

New plugin documentation should be added to Sahara docs.


References
==========

[1] https://cwiki.apache.org/confluence/display/AMBARI/Blueprints

[2] http://stackalytics.com/?module=sahara-group&release=all&company=hortonworks&metric=commits

[3] https://review.openstack.org/#/c/184292/

[4] https://review.openstack.org/#/c/185100/

[5] https://review.openstack.org/#/c/181732/

[6] https://review.openstack.org/#/c/194580/

[7] https://review.openstack.org/#/c/195726/

[8] https://review.openstack.org/#/c/193081/

[9] https://review.openstack.org/#/c/197551/

[10] https://review.openstack.org/#/c/200570/
