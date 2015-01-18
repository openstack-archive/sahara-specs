..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================
Refactor MapR plugin code
=========================

https://blueprints.launchpad.net/sahara/+spec/mapr-refactor

MapR plugin's code should be refactored to support easy addition of new
services and releases

Problem description
===================

Current plugin implementation has several weaknesses:
* Declarative nature of Service
* Code complexity caused by usage of plugin-spec.json
* Almost all actions are implemented as sequence of utility functions calls
* Not clear separation of behaviour to modules and classes
* Some code is redundant

Proposed change
===============

* Extract Service entity to separate class with customizable behaviour
* Move provisioning logic from utility modules to specialized classes
* Remove redundant code

MapR plugin implementation delegates all operations to it's counterparts in
VersionHandler interface. VersionHandler interface mimics plugin SPI with
additional methods get_context(cluster) and get_services(). ClusterContext
object returned by get_context wraps cluster object passed as argument and
provides additional information about cluster as well as utility methods
related to wrapped cluster.

Service definitions resides in 'sahara.plugins.mapr.services' package instead
of plugin-spec.json which is completely removed now. Each service definition
represents particular version of service.

Alternatives
------------

Leave code "as-is"

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

Developers of subsequent versions of the MapR plugin should take into account
this changes.

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
  aosadchiy

Other contributors:
  ssvinarchuk

Work Items
----------

1) Extract Service entity to separate class with customizable behaviour
2) Move provisioning logic from utility modules to specialized classes
3) Remove redundant code


Dependencies
============

None

Testing
=======

Existing integration tests are sufficient

Documentation Impact
====================

None

References
==========

None
