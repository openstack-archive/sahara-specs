..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================================
Clean up clusters that are in non-final state for a long time
=============================================================

https://blueprints.launchpad.net/sahara/+spec/periodic-cleanup

This spec is to introduce periodic task to clean up old clusters in
non-final state.

Problem description
===================

For now it is possible that sahara cluster becomes stuck because of different
reasons (e.g. if sahara service was restarted during provisioning or neutron
failed to assign floating IP). This could lead to clusters holding resources
for a long time. This could happen in different tenants and it is hard to
check such conditions manually.

Related bug: https://bugs.launchpad.net/sahara/+bug/1185909

Proposed change
===============

Add "cleanup_time_for_nonfinal_clusters" parameter in "periodic" section of
configuration.

Based on this configuration periodic task will search clusters that are in
non-final state and weren't updated for a given time.

Term "non-final" includes all cluster states except "Active" and "Error".

"cleanup_time_for_nonfinal_clusters" parameter will be in hours. Non-positive
value will indicate that clean up option is disabled.

Default value will be 0 to keep backward compatibility (users don't expect
that after upgrade all their non-final cluster will be deleted).

'updated_at' column of 'clusters' column will be used to determine last
change. This is not 100% accurate, but good enough. This field is changed
each time cluster status is changed.

Alternatives
------------

Add such functionality to external service (e.g. Blazar).

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

* Implement feature
* Document feature

Dependencies
============

None.

Testing
=======

Manually.

Documentation Impact
====================

Need to be documented.

References
==========

* https://bugs.launchpad.net/sahara/+bug/1185909