..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================
Improve error handling for provisioning operations
==================================================

https://blueprints.launchpad.net/sahara/+spec/error-handling-in-provisioning

Currently provisioning error handling is sprayed across the whole provisioning
code. This spec is to unify error handling and localize it in one place.


Problem description
===================

Currently we have two problems connected with error handling in provisioning
part:

1) The code incorrectly handles situations when cluster was deleted by user
   during provisioning. In that case an arbitrary error might be raised in
   many places.
2) The code performs rollback only in certain places, while it could be done
   for any provisioning/scaling phase.

The following CR:
https://review.openstack.org/#/c/98556
mostly fixes issue #1, but it is full of duplicate code.


Proposed change
===============

The following solution is proposed instead which requires architectural
changes, but rather reliably fixes both problems:

1) For both cluster creation and scaling move error handling logic to the very
   top functions inside ops.py file. Once exception is caught properly
   process it:

   a) if cluster object does not exists in DB, that means that user deleted
      the cluster during provisioning; handle it and return
   b) if cluster object exists, log it and perform rollback
2) Do not do any checks if cluster exists outside of ops.py, except places
   where processing might hang indefinitely without the check.

We can employ the following rollback strategy:

For cluster creation: if anything went wrong, kill all VMs and move cluster
to the Error state.

For cluster scaling: that will be long. Cluster scaling has the following
stages:

1) decommission unneeded nodes (by plugin)
2) terminate unneeded nodes and create a new ones if needed (by engine). Note
   that both scaling up and down could be run simultaneously but in
   different node groups.
3) Configure and start nodes (by plugin)

My suggestion what to do if an exception occurred in the respective stage:

1) move cluster to Error state
2) kill unneeded nodes (finish scale down). Also kill new nodes, if they were
   created for scale up.
3) move cluster to Error state

In cases #1 and #3 it is dangerous to delete not decommissioned or not
configured nodes as this can lead to data loss.

Alternatives
------------

Keep supporting current code. It is not elegant but works good enough.

Data model impact
-----------------

None

REST API impact
---------------

None

Other end user impact
---------------------

Data provisioning logic will be changed a lot. This could lead to behavior
change in case of errors on different stages.

Deployer impact
---------------

None

Developer impact
----------------

Provisioning engine API will be extended with "rollback_cluster" method.

Sahara-image-elements impact
----------------------------

None

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  alazarev

Other contributors:
  dmitrymex

Work Items
----------

1) Implement change
2) Test that cluster provisioning and rollback works on all feature matrix we
   have.

Dependencies
============

None

Testing
=======

Provisioning could be tested manually and by CI.
It is much harder to test rollback. Even current code is not tested well (e.g.
https://bugs.launchpad.net/sahara/+bug/1337006).

Documentation Impact
====================

None

References
==========

None