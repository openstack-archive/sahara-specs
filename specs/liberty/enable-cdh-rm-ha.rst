..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================
CDH YARN ResourceManager HA Support
===================================

https://blueprints.launchpad.net/sahara/+spec/cdh-ha-support

This blueprint aims to implement YARN ResourceManager (RM) High-Availability
(HA) for Cloudra plugin.

Problem description
===================

Currently Cloudera plugin does not support HA for YARN ResourceManager.
Therefore we plan to implement RM HA.


Proposed change
===============

The implementation of the RM HA will be done via CM API enable_rm_ha(). This
API will help us enable ResourceManager HA by giving several info augments.

To achieve RM HA, we need to add one Standby NodeManager in the node
of the cluster templates (Cloudera Plugin only support one Standby NodeManager,
while the other implementations might allow >1 Standby NodeManager).
When RM HA is enabled, we will start both the Primary and Standby NodeManagers,
let the Primary NodeManager be active, and leave the Standby NodManager
standby.

CM API enable_rm_ha accepts a new ResourceManager Host new_rm_host_id as
parameter. A Standby ResourceManager will be started on new_rm_host_id node.

ResourceManager HA requires Primary ResourceManager and Standby ResourceManager
roles deployed on different physical hosts. Zookeeper service is required too.

When ResourceManager HA is enabled, Oozie or applications depended on RM should
be able to check all available RMs and get the active RM by themselves. There
is no way to automatically switch between RMs smoothly.

Overall, we will implement RM HA as below:

* Add a role YARN_STANDBYRM.
* If YARN_STANDBYRM was selected by user (cluster admin), then YARN RM HA will
  be enabled.
* If RM HA is enabled, we will check Anti-affinity to make sure YARN_STANDBYRM
  and YARN_RESOURCEMANAGER will not be on the same physical host.
* If RM HA is enabled, Zookeeper service is required in the cluster.
* If RM HA is enabled, we will create cluster with ResourceManager on node
  where YARN_RESOURCEMANAGER role is assigned.
* If RM HA is enabled, after the cluster is started, we will call enable_rm_ha
  to enabled RM HA by using the YARN_STANDBYRM node as parameter.

It should be noted that, if HA is enabled, in Oozie workflow xml file, we need
to detect the active ResourceManager in method get_resource_manager_uri and
pass it to Oozie each time when we use it. I plan to include this part of codes
in later patches.

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

We will only do primitive checks: create a Cloudera cluster with RM HA, and
see whether it is active.

Documentation Impact
====================

The documentation needs to be updated with information about enabling CDH YARN
ResourceManager HA.


References
==========

* `Configuring High Availability for ResourceManager (MRv2/YARN) <http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_hag_rm_ha_config.html/>`
