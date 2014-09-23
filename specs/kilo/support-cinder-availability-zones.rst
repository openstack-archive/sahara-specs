..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Support Cinder availability zones
=================================

https://blueprints.launchpad.net/sahara/+spec/support-cinder-availability-zones

Extend API to support specifying Cinder availability zones where to create
volumes.


Problem description
===================

It can be desirable to assign Cinder availability zones to node groups, in
order to have fine-grained cluster volumes topologies.

Use case:

* As a end user I want namenode volumes to be spawned in the regular AZ and
  datanode volumes in a high-performance ``ssd-high-io`` AZ.


Proposed change
===============

It adds a new ``volumes_availability_zone`` property in NodeGroup and
NodeGroupTemplate objects.  When set, it modifies the direct and Heat engines
to force creating of volumes into the right AZ.

Alternatives
------------

None

Data model impact
-----------------

This change will add ``volumes_availability_zone`` columns in sahara database,
next to ``volumes_per_node`` and ``volumes_size``.  Impacted tables are
``node_groups``, ``node_group_templates`` and ``templates_relations``.

A database migration will accompany this change.

REST API impact
---------------

Each API method which deals with node groups and node groups templates will
have an additional (and optional) ``volumes_availability_zone`` parameter,
which will be taken into account if ``volumes_per_node`` is set and non-zero.

Example::

  {
    "name": "cluster1",
    "node_groups": [
      {
        "name": "master",
        "count": 1,
      },
      {
        "name": "worker",
        "count": 3,
        "volumes_per_node": 1,
        "volumes_size": 100,
        "volumes_availability_zone": "ssd-high-io"
      }
    ]
  }

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

* Add a ``volumes_availability_zone`` property in NodeGroup and
  NodeGroupTemplate objects.

* When a user specifies the (optional) volumes_availability zone, check its
  existence.

* In the direct engine, include the ``volumes_availability_zone`` argument in
  the call to cinder.client().volumes.create().

* In the Heat engine, add the ``availability_zone`` property in
  sahara/resources/volume.heat.


Dependencies
============

None


Testing
=======

* Test cluster creation with ``volumes_availability_zone`` specified.

* Test cluster creation with wrong ``volumes_availability_zone`` specified.


Documentation Impact
====================

Documentation will need to be updated, at sections related to node group
template creation and cluster creation.


References
==========

None
