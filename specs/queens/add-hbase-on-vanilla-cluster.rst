..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Add HBase on Vanilla cluster
============================

https://blueprints.launchpad.net/sahara/+spec/hbase-on-vanila

Apache HBase provides large-scale tabular storage for Hadoop using
the Hadoop Distributed File System(HDFS). This document serves as
a description to add the support of HBase and ZooKeeper services on
Vanilla cluster.

Problem description
===================

Sahara vanilla plugin allows user to quickly provision a cluster
with many core services, but it doesn't support HBase and ZooKeeper.

Proposed change
===============

To go against the Vanilla cluster distributed architecture, we only support
fully-distributed HBase deployment. In a distributed configuration,
the cluster contains multiple nodes, each of which runs one or more HBase
Daemon. These include HBase Master instance, multiple ZooKeeper nodes and
multiple RegionServer nodes.

A distributed HBase installation depends on a running ZooKeeper cluster.
HBase default manages a ZooKeeper "cluster" for you, but you can also
manage the ZooKeeper ensemble independent of HBase. The variable
"HBASE_MANAGES_ZK" in "conf/hbase-env.sh", which default to true, tells
HBase whether to start/stop the ZooKeeper ensemble servers as part of HBase.

We should expose this variable in "cluster_configs" to let user determine
the creator of ZooKeeper service.

In production, it is recommended that run a ZooKeeper ensemble of 3, 5 or 7
machines; the more members an ensemble has, the more tolerant the ensemble
is of host failures. Also, run an odd number of machines. An even number
of peers is supported, but it is normally not used because an even sized
ensemble requires, proportionally, more peers to form a quorum than an odd
sized ensemble requires.

* If we set "HBASE_MANAGES_ZK" to false, Sahara will validate the number
  of ZooKeeper services in node groups to keep ZK instances in odd number.
* If we set "HBASE_MANAGES_ZK" to true, Sahara will automatically
  determine the instances to start ZooKeeper. The cluster contains ZK
  nodes more than 1 nodes, less than 5 nodes. If we want to have more
  ZK nodes, setting HBASE_MANAGES_ZK to false would be a good choice.

If we want to scale the cluster up or down, ZooKeeper and HBase services
will be restarted. And after scaling up or down, the rest of ZooKeeper nodes
should also be kept in odd number. If there is only one ZooKeeper node, the
status of ZooKeeper service will be "standalone".

One thing should be specified is the default value used in configuration:

ZooKeeper Configuration in "/opt/zookeeper/conf/zoo.cfg":


.. code-block:: console

   dataDir=/var/data/zookeeper
   clientPort=2181
   server.1=zk-0:2888:3888
   server.2=zk-1:2888:3888


HBase Configuration in "/opt/hbase/conf/hbase-site.xml":


.. code-block:: console

   hbase.tmp.dir=/var/data/hbase
   hbase.rootdir=hdfs://master:9000/hbase
   hbase.cluster.distributed=true
   hbase.master.port=16000
   hbase.master.info.port=16010
   hbase.regionserver.port=16020

Security Group will open ports (2181, 2888, 3888, 16000, 16010, 16020) after
this change if configuration is not changed.

Alternatives
------------

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

* Build new Vanilla image includes ZK and HBase packages

Sahara-dashboard / Horizon impact
---------------------------------

* An option should be added to the Node Group create and update forms.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Shu Yingya

Work Items
----------

* Build new image by sahara-image-elements
* Add ZooKeeper to Vanilla in sahara
* Add HBase to Vanilla in sahara
* Update Sahara-dashboard to choose ZK creator in sahara-dashboard

Dependencies
============

None

Testing
=======

* Unit test coverage in sahara

Documentation Impact
====================

* Vanilla plugin description should be updated

References
==========

None
