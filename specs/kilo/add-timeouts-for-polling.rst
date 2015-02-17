..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add timeouts for infinite polling for smth
==========================================

https://blueprints.launchpad.net/sahara/+spec/add-timeouts-for-polling

We have infinite polling-processes for smth in sahara code.
It would be nice to add timeouts for its execution

Problem description
===================

When creating a cluster, cluster's status may be stuck in Waiting and will
always await network if the network configuration is wrong. We can add
configurable timeouts for all possible polling processes.

Proposed change
===============

Here you can find list of all polling processes in Sahara code:

For service module:

* volumes._await_attach_volumes
* volumes._create_attach_volume
* volumes._detach_volume
* engine._wait_until_accessible
* engine._await_networks
* direct_engine._await_active
* direct_engine._await_deleted
* edp.job_manager.cancel_job

For utils module:

* openstack.heat.wait_stack_completion

For cdh plugin:

* cloudera_utils.await_agents
* plugin_utils.start_cloudera_manager

For hdp plugin:

* _wait_for_async_request for both plugin versions
* wait_for_host_registrations for both plugin versions
* decommission_cluster_instances for 2.0.6 versionhandler

For spark plugin:

* scaling.decommission_dn

For vanilla plugin:

* hadoop2.scaling._check_decommission
* hadoop2.await_datanodes
* v1_2_1.scaling.decommission_dn
* v1_2_1.versionhandler._await_datanodes

Proposed change would consists of following steps:

* Add new module polling_utils in sahara/utils where would register
  new timeouts options for polling processes from service and utils modules.
  Also it would consist with a specific general util for polling.
  As example it can be moved from
  https://github.com/openstack/sahara/blob/master/sahara/utils/general.py#L175.
* Add new section in sahara.conf.sample which would consist with all
  timeouts.
* All plugin specific options would be related only with this plugin and also
  would be configurable. Also user would have ability to configure all plugin
  specific options during cluster template creation.

Alternatives
------------

None

Data model impact
-----------------

This change doesn't require any data models modifications.

REST API impact
---------------

This change doesn't require any REST API modifications.

Other end user impact
---------------------

User would have ability to configure all timeouts separately. Some options
would be configurable via sahara.conf.sample, other would be configurable
from plugin during cluster template creation.

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

During cluster template creation user would have ability to configure
plugin specific options from UI.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vgridnev

Work Items
----------

* Add general polling util to sahara/utils/polling_utils
* Apply this changes to all plugins and sahara engines.

Dependencies
============

Depends on current Openstack Data Processing Requirements.

Testing
=======

this change would require to add unit tests. Also this change would be tested
manually.

Documentation Impact
====================

Required to document this feauture in sahara/userdoc/configuration.guide.

References
==========

[1] https://bugs.launchpad.net/sahara/+bug/1402903
[2] https://bugs.launchpad.net/sahara/+bug/1319079
