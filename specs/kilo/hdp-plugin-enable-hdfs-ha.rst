..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
  License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================================
Enable HDFS NameNode High Availability with HDP 2.0.6 plugin
============================================================

https://blueprints.launchpad.net/sahara/+spec/hdp-plugin-enable-hdfs-ha

Extend HDP 2.0.6 plugin to include the setup and configuration of the HDFS
NameNode High Availability after creating, configuring and starting the
cluster.

Problem description
===================

Hadoop clusters are created with a single NameNode which represents a SPOF
(Single Point Of Failure). If the NameNode becomes unavailable, the whole
cluster becomes unavailable and we have to wait for the NameNode to come back
up before using the cluster again. NameNode High Availability was introduced in
Hadoop 2.0.0 and integrated into HDP as of the 2.0.0 release.  The High
Availability is achieved by having 2 NameNodes, an active NameNode and a stand
by one. When the active fails the standby steps in and act as the cluster's
NameNode. NameNode's High Availability can be configured manually on clusters
deployed with HDP 2.0.6 plugin in Sahara through Ambari, but the process is
long, tedious and error prone.

End users might not have the necessary skills required to setup the High
Availability and deployers might prefer to deploy highly available clusters
without manually configuring each one.

HDFS NameNode High Availability (Using Quorum Journal Manager (QJM)) uses
Journal nodes to share HDFS edits between the active and the standby
namenodes. The journal nodes ensure that the two namenodes have the same set of
HDFS edits, but do not ensure the automatic failover. The automatic failover
requires Zookeeper servers and Zookeeper Failover Controllers (ZKFC).  A
typical cluster with HDFS NameNode High Availability uses at least three (or a
an odd number greater than three) journal nodes and a zookeeper cluster with
three or five zookeeper servers. The Zookeeper Failover Controllers are
installed on the servers acting as active and standby namenodes.  The setup
removes the secondary namenode (which is usually installed on a different
server than the one hosting the namenode) and installs a second namenode
process. The journal nodes and zookeeper servers can be installed on the same
servers running the active and standby (old secondary namenode) namenodes. This
leaves us with 2 journal nodes and 2 zookeeper servers. An additional server is
all what's needed with journal node and zookeeper server to setup a minimally
viable Hadoop cluster with HDFS NameNode High Availability. (For more info :
http://hadoop.apache.org/docs/r2.3.0/hadoop-yarn/hadoop-yarn-site/HDFSHighAvailabilityWithQJM.html)

Proposed change
===============

The 'Create Node Group Template' wizard will introduce a new process
'JOURNALNODE' to the list of available processes for HDP 2.0.6 plugin. Other
processes necessary for HDFS HA are either already included in the list
(NAMENODE, ZOOKEEPER_SERVER and SECONDARY_NAMENODE) or will be automatically
setup (Zookeeper Failover Controllers).

The 'Launch Cluster' wizard for HDP 2.0.6 plugin will include a checkbox
'Enable HDFS HA'. This option will default to False and will be added to the
cluster object.

The verification code will verify the necessary requirements for a Hadoop
cluster and a set of additional requirements in the case where 'Enable HDFS HA'
is set to True. The requirements include : NAMENODE and SECONDARY_NAMENODE on
different servers At least three journal nodes on different servers At least
three zookeeper servers on different servers

Upon successful validation the cluster will be created and once it's in the
'Active' state Availability' and if 'Enable HDFS HA' is True the service will
instruct the plugin to start configuring the NameNode High Availability. The
cluster will be set in 'Configuring HDFS HA' state and the plugin will start
the configuration procedure. The procedure starts by the plugin stopping all
the services and executing some preparation commands on the server with the
namenode process (through the hadoopserver objects). Then the plugin installs
and starts the journal nodes using Ambari REST API (POST, PUT, WAIT ASYNC).
Next the configuration is updated using Ambari REST API (PUT), other services
including Hive, Oozie and Hue might require configuration update if they are
installed. Finally some more remote commands are executed on the namenodes and
the Zookeeper Failover Controllers are installed and started and the
SECONDARY_NAMENODE process is deleted. The plugin will return and the cluster
will be set back in the 'Active' state.

Alternatives
------------

Manual setup through Ambari web interface

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

Developers of subsequent versions of the HDP plugin should take into account
this option and the added functionality. The procedure is not likely to change
in newer versions of HDP as it uses Ambari's API which stays intact with newer
versions.

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

A checkbox 'Enable HDFS HA' in the 'Launch Cluster' wizard when the user
chooses HDP 2.0.6 plugin.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  abbass-marouni

Work Items
----------

* Add a new attribute to cluster-level configs to indicate whether HA is enabled or not.
* Add new service classes to HDP 2.0.6 for Journal nodes and Zookeeper Failover
  Controllers
* Add new remote methods to hdp hadoopserver.py for remote commands
* Add new methods to generate new configurations according to cluster
  configuration

Dependencies
============

None

Testing
=======

Unit Test service classes
Unit Test new cluster specs
Integration Test cluster creation with HA
Integration Test cluster creation without HA

Documentation Impact
====================

Update documentation to reflect new changes and to explain new options.

References
==========

None
