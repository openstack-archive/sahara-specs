..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
CDH HDFS HA Support
===================

https://blueprints.launchpad.net/sahara/+spec/cdh-ha-support

This blueprint aims to implement HDFS High-Availability (HA) for Cloudra
plugin.

Problem description
===================

Currently Cloudera plugin does not support HA for services. We plan to
implement HDFS HA as the first step. HA for Yarn and other services will be
the later steps.


Proposed change
===============

The implementation of the HDFS HA will be done via CM API enable_nn_ha(). This
API will help us enable HDFS HA by giving several info augments.

CDH 5 only supports Quorum-based Storage as the only HA implementation, so we
will only implement this. To achieve this, we need to add a Standby NameNode,
and several JournalNodes. The JournalNode number should be odd and at least 3.
When HDFS HA is enabled, SecondaryNameNode will not be used. So we can reuse
the node for SecondaryNameNode for StandbyNameNode.

HDFS HA has several hardware constraints (see the reference link). However,
for all resources are virtual in Openstack, we will only require NameNode and
StandbyNameNode are on different physical hosts.

Overall, we will implement HDFS as below:

* Add a role JournalNode.
* If JournalNode was selected by user (cluster admin), then HA will be enabled.
* If HA is enabled, we will validate whether JournalNode number meet
  requirements.
* JournalNode roles will not be really created during cluster creation. In fact
  they will be used as parameters of CM API enable_nn_ha.
* If HA is enabled, we will use SecondaryNameNode as the StandbyNameNode.
* If HA is enabled, we will set Anti-affinity to make sure NameNode and
  SecondaryNameNode will not be on the same physical host.
* If HA is enabled, Zookeeper service is required in the cluster.
* After the cluster was started, we will call enable_nn_ha to enable HDFS HA.
* If HA is enabled, in Oozie workflow xml file, we will give nameservice name
  instead of the NameNode name in method get_name_node_uri. So that the cluster
  can determine by itself which NameNode is active.

Alternatives
------------

None.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None.

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
  Ken Chen

Work Items
----------

Changes will be only in sahara/plugins/cdh directory. We will only do this
based on CDH 5.4.0 at this stage. CDH 5.0.0 and CDH 5.3.0 plugins will not be
supported. Changes were described in the Proposed change section.


Dependencies
============

None.


Testing
=======

We will only do primitive checks: create a Cloudera cluster with HDFS HA, and
see whether it is active.

Documentation Impact
====================

The documentation needs to be updated with information about enabling CDH HDFS
HA.


References
==========

* `NameNode HA with QJM <http://www.edureka.co/blog/namenode-high-availability-with-quorum-journal-manager-qjm/>`
* `Introduction to HDFS HA <http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_hag_hdfs_ha_intro.html>`
* `Enable HDFS HA Using Cloudera Manager <http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_hag_hdfs_ha_enabling.html#cmug_topic_5_12_unique_1>`
* `Configuring Hardware for HDFS HA <http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_hag_hdfs_ha_hardware_config.html>`

