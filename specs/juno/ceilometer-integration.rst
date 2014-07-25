..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================================
Specification for integration Sahara with Ceilometer
====================================================

https://blueprints.launchpad.net/sahara/+spec/ceilometer-integration

It is impossible to send notifications from Sahara to Ceilometer now. Sahara
should have an ability to send notifications about cluster modifications,
for example about creating/updating/destoroying cluster.

Problem description
===================

New feature will provide the following ability:

* Sending notifications to Ceilometer about cluster modifications, which
  can help for user to achive some information about clusters which he have:
  number of active clusters in each moment and etc.


Proposed change
===============

Change will consist of following modifications:

* Adding to Sahara an ability to send notifications.

* Adding to Sahara sending notificitions in such places, where cluster is
  modified

* Adding to Ceilometer an ability to pull notifications from Sahara exchange
  and to parse it.


Alternatives
------------

None


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

None

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
vgridnev

Other contributors:
slukjanov

Work Items
----------

* Add notification sender in Sahara

* Add Ceilometer parser

* Add unit tests

Dependencies
============

Depends on OpenStack requirements


Testing
=======

There will be:

 * unit tests in Sahara

 * unit tests in Ceilometer


Documentation Impact
====================

Need modifications in Ceilometer documentation here:

* [1] http://docs.openstack.org/developer/ceilometer/measurements.html
* [2] http://docs.openstack.org/developer/ceilometer/install/manual.html


References
==========

* [1] https://review.openstack.org/#/c/108982/
