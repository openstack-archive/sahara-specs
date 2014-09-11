..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Support Nova availability zones
===============================

https://blueprints.launchpad.net/sahara/+spec/support-nova-availability-zones

Extend API to support specifying Nova availability zones where to spawn
instances.


Problem description
===================

It can be desirable to assign Nova availability zones to node groups, in order
to have fine-grained clusters topologies.

Use cases:

* As a end user I want namenode instances to be spawned in the regular ``nova``
  AZ and datanode instances in my high-performance ``nova-highperf`` AZ.

* As a end user I want instances from node-group A to be all together in the
  ``nova-1`` AZ, separated from instances from node-group B in ``nova-2``.


Proposed change
===============

The proposed change is already implemented at [1].

It adds an ``availability_zone`` property in NodeGroup and NodeGroupTemplate
objects.  When set, it modifies the direct and Heat engines to force spawning
of instances into the right AZ.

Alternatives
------------

None

Data model impact
-----------------

This change will add ``availability_zone`` columns in the sahara database
(``node_groups``, ``node_group_templates`` and ``templates_relations`` tables).

A database migration will accompany this change.

REST API impact
---------------

Each API method which deals with node groups and node groups templates will
have an additional (and optional) ``availability_zone`` parameter.

Other end user impact
---------------------

python-saharaclient should be modified to integrate this new feature.

Deployer impact
---------------

Needs to migrate DB version using:

  ``sahara-db-manage --config-file /etc/sahara/sahara.conf upgrade head``

Developer impact
----------------

None

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

sahara-dashboard should be modified to integrate this new feature.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  adrien-verge

Work Items
----------

The proposed change is already implemented at [1].

* Add an ``availability_zone`` property in NodeGroup and NodeGroupTemplate
  objects.

* When a user specifies the (optional) availability zone, check its existence.

* In the direct engine, include the ``availability_zone`` argument in the call
  to nova.client().servers.create().

* In the Heat engine, add the ``availability_zone`` property in
  sahara/resources/instance.heat.


Dependencies
============

None


Testing
=======

* Test cluster creation without availability zone specified.

* Test cluster creation with availability zone specified.

* Test cluster creation with wrong availability zone specified.


Documentation Impact
====================

Documentation will need to be updated, at sections related to node group
template creation and cluster creation.


References
==========

* [1] https://review.openstack.org/#/c/120096
