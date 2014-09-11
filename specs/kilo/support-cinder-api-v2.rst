..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Support Cinder API version 2
============================

https://blueprints.launchpad.net/sahara/+spec/support-cinder-api-v2

This specification proposes to add support for the second version of the Cinder
API, which brings useful improvements and will soon replace version one.


Problem description
===================

Currently Sahara uses only version 1 of Cinder API to create volumes.  Version
two, however, brings useful features such as scheduler hints, more consistent
responses, caching, filtering, etc.

Also, Cinder is deprecating version 1 in favor of 2, so supporting both would
make switching easier for users.

Use cases:

* As a developer I want to be able to pass scheduler hints to Cinder when
  creating clusters, in order to choose volumes more precisely and achieve
  improved performance.

* As a developer I want filtering into my requests to Cinder to make queries
  lighter.

* As a deployer I want to be able to choose between legacy Cinder API v1 and
  newer v2 API.


Proposed change
===============

The implementation will add a configuration option, cinder_api_version, that
will be defaulted to:

  ``cinder_api_version=1``

but can be changed to

  ``cinder_api_version=2``

by modifying sahara.conf.

The client() method in sahara/utils/openstack/cinder.py will either return
clientv1() or clientv2() depending on the configuration.

Alternatives
------------

Wait for Cinder API v1 to be deprecated and switch abruptly to v2.

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

* If the deployer wants to keep using Cinder API version 1, nothing has to be
  done.

* If the deployer wants to upgrade to version 2, the cinder_api_version option
  in sahara.conf should be overwritten.

Developer impact
----------------

Developers can read CONF.cinder_api_version to know what API version is being
used.

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
  adrien-verge

Work Items
----------

* Add a configuration option: cinder_api_version.

* Put some magic in sahara/utils/openstack/cinder.py to pick the correct Cinder
  client depending on the configuration.


Dependencies
============

None


Testing
=======

Same as for v1 API.


Documentation Impact
====================

None


References
==========

* https://wiki.openstack.org/wiki/CinderAPIv2
