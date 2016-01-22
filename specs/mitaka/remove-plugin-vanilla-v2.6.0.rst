..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Remove plugin Vanilla V2.6.0
==========================================

https://blueprints.launchpad.net/sahara/+spec/deprecate-plugin-vanilla2.6.0

Problem description
===================

As far as Vanilla v2.6.0 plugin is deprecated, we should remove all the code
completely.

Proposed change
===============

(1) Remove all plugin v2_6_0 code under plugins
(2) Remove all tests including unit and integration tests.
(3) Update Vanilla plugin documentation.

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
  luhuichun

Other contributors:
  None

Work Items
----------

* Remove Vanilla v2.6.0
* Remove all tests for Vanilla v2.6.0
* Update documentation

Dependencies
============

Dependend on change Iff2faf759ac21d5bd15372bae97a858a3d036ccb

Testing
=======

None

Documentation Impact
====================

Documention needs to be updated:
* doc/source/userdoc/vanilla_plugin.rst

References
==========

None
