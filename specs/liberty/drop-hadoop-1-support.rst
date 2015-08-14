..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================
Drop Hadoop v1 support in provisioning plugins
==============================================

https://blueprints.launchpad.net/sahara/+spec/drop-hadoop-1

This specification proposes to drop old Hadoop versions

Problem description
===================

Support for Hadoop 1 in provisioning plugins has become an unused feature. The
Hadoop development for v1 is almost frozen and Hadoop vendors are also dropping
its support.

As the sahara-plugins and sahara main repository split is going to happen it
will be a lot easier move less plugins to the new repo. Also the number of
unit and scenario tests will reduce.

The list of versions to be dropped is:

* Vanilla 1.2.1
* HDP 1.3.2

This spec does not suppose to drop the deprecated versions of plugins. That
should be a regular part of the release cycle.

Proposed change
===============

Drop the plugins code for Hadoop v1 along with:

* xml/json resources
* unit and scenario tests
* sample files
* image elements


Alternatives
------------

TBD

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

Elements for building Hadoop v1 images should be dropped as well

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  nkonovalov

Work Items
----------

* Disable Hadoop v1 tests in sahara-ci
* Drop related code and resource from sahara main repo
* Drop image elements

Dependencies
============

None

Testing
=======

Sahara-ci should not test Hadoop v1 anymore

Documentation Impact
====================

Add a note that Hadoop v1 is not supported in new releases.

References
==========

None
