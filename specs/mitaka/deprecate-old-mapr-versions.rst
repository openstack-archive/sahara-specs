..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Remove unsupported versions of MapR plugin
==========================================

https://blueprints.launchpad.net/sahara/+spec/deprecate-old-mapr-versions

Remove MapR plugin 3.1.1 4.0.1 4.0.2 5.0.0.mrv1 and mapr-spark.

Problem description
===================

Some of supported MapR versions are old and not used and others
have their benefits contained in newer versions.

Proposed change
===============

We will not support the following MapR versions and remove
them in Mitaka release.

The following MapR version is removed due to end of support:
* MapR 5.0.0 mrv1
* MapR 4.0.2 mrv1 mrv2
* MapR 4.0.1 mrv1 mrv2
* MapR 3.1.1

mapr-spark version will also be removed because Spark will
go up with latest MapR as a service on YARN.

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

* Remove MapR v3.1.1 support
* Remove MapR v4.0.1 support
* Remove MapR v4.0.2 support
* Remove MapR v5.0.0 support
* Remove MapR mapr-spark

Sahara-dashboard / Horizon impact
---------------------------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  groghkov

Other contributors:
  None

Work Items
----------

* Remove MapR v3.1.1
* Remove all tests for v3.1.1
* Remove MapR v4.0.1
* Remove all tests for v4.0.1
* Remove MapR v4.0.2
* Remove all tests for v4.0.2
* Remove MapR v5.0.0.mrv1
* Remove all tests for v5.0.0.mrv1
* Remove MapR Spark
* Remove all tests for MapR Spark
* Remove default templates for MapR 3.1.1 4.0.1 4.0.2 5.0.0.mrv2
  in sahara/plugins/default_templates/mapr
* Update documentation

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

Documention needs to be updated:
* doc/source/userdoc/mapr_plugin.rst

References
==========

None