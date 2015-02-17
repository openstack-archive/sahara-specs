..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Cinder volume instance locality functionality
=============================================

https://blueprints.launchpad.net/sahara/+spec/volume-instance-locality

This specification proposes to add an opportunity to have an instance and an
attached volumes on the same physical host.

Problem description
===================

Currently there is no way to specify that volumes are attached to the same
physical host as the instance. It would be nice to have this opportunity.

Proposed change
===============

This feature could be done with Cinder InstanceLocalityFilter which allows to
request creation of volumes local to instance. It would increase performance
of I/O operations.

There will be several changes:

* Boolean field ``volume_local_to_instance`` will be added to every node group
  template, node group and templates relation. This field will be optional and
  ``False`` by default.

* If ``volume_local_to_instance`` is set to ``True``, Cinder volumes will be
  created on the same host.

* If ``volume_local_to_instance`` is ``True``, all instances of the node group
  should be created on hosts with  free disk space >= ``volumes_per_node`` *
  ``volumes_size``. If it cannot be done, error should be occurred.

Alternatives
------------

None

Data model impact
-----------------

``volume_local_to_instance`` field should be added to ``node_groups``,
``node_group_templates``, ``templates_relations``.

REST API impact
---------------

* API will be extended to support ``volume_local_to_instance`` option.
  ``volume_local_to_instance`` is optional argument with ``False`` value by
  default, so this change will be backward compatible.

* python client will be updated

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

None

Sahara-dashboard / Horizon impact
---------------------------------

New field to select ``volume_local_to_instance`` option during node group
template creation will be added.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  apavlov-n

Work Items
----------

* Adding ability to create instance and volumes on the same host;
* Adding ability to create instance on appropriate host;
* Updating documentation;
* Updating UI;
* Updating python client;
* Adding unit tests.

Dependencies
============

None

Testing
=======

Unit tests will be added.

Documentation Impact
====================

Documentation will be updated. Will be documented when this feature can be
used and when it cannot. Also will be noted how to enable it on Cinder side.

References
==========

* http://docs.openstack.org/developer/cinder/api/cinder.scheduler.filters.instance_locality_filter.html
