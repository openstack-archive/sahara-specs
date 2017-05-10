..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Two step scaling with Heat engine
=================================

https://blueprints.launchpad.net/sahara/+spec/heat-two-step-scaling

In case of failure during cluster scaling Heat will rollback deletion of
all resources. After that Sahara will ask Heat to delete them anyway.
"Rollback deletion and delete after that" step looks unnecessary.

Problem description
===================

The following scenario happens when Sahara with Heat engine failed to scale
cluster:

1. User requested cluster scaling
2. Sahara runs hadoop nodes decommissioning
3. Sahara runs heat stack update with both added and removed nodes and
   rollback_on_failure=True
4. If 3 failed Heat returns all deleted nodes back
5. Sahara runs heat stack update with removed nodes only
6. Heat removes nodes one more time

So, at step 4 Heat restores nodes that will be deleted later anyway.

Proposed change
===============

The described problem could be avoided by scaling in two steps. So, resulting
flow will look like this:

1. User requested cluster scaling
2. Sahara runs hadoop nodes decommissioning
3. Sahara runs heat stack update with removed resources only and
   rollback_on_failure=False
4. Sahara runs heat stack update with new resources and
   rollback_on_failure=True

In this case if step 4 failed Heat will not try to restore deleted resources.
It will rollback to the state where resources are already deleted.

If step 3 fails there is nothing Sahara can do. Cluster will be moved to
'Error' state.

Alternatives
------------

Do nothing since issue appears only in rare scenario of failed scaling down.

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
  alazarev (Andrew Lazarev)

Other contributors:
  None

Work Items
----------

* Perform changes
* Test that rollback on cluster scale failure works as expected

Dependencies
============

None.

Testing
=======

Manually.

Documentation Impact
====================

None.

References
==========

None.