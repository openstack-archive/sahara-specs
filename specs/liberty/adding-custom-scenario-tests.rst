..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================================
Adding custom scenario to scenario tests
========================================

https://blueprints.launchpad.net/sahara/+spec/custom-checks

This specification proposes to add custom tests to scenario tests for more
exhaustive testing of Sahara.

Problem description
===================

Now, scenario tests testing of basic functionality and user can not add
custom personal tests for check of other functionality in Sahara.

Extra tests should be added:

* checks for mount and available cinder volumes;
* checks for started services on cluster;
* checks for other processes that now not testing

Proposed change
===============

Custom test need add to sahara/tests/scenario/custom_checks and need implement
support of this scenarios in scenario tests.

For implementation this spec, need change field parameters for field "scenario"
in scenario tests. Now is type "enum", need change to "string" for adding
ability set custom tests.

Additionally, should be rewrite sahara/tests/scenario/testcase.py.mako
template. Custom tests will be called from module with name in format
`check_{name of check}` with method `check()` inside.

All auxiliary methods for current custom check will be written in module with
this tests. Methods, for global using in several custom scenario can be
implemented in sahara/tests/scenario/base.py.

Alternatives
------------

Tests can be added manually to scenario tests in Base class.

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
  esikachev

Work Items
----------

* Adding ability to run custom scenario tests;
* Move scripts from old integration tests to scenario tests as custom checks;
* Adding new custom checks.

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

None

References
==========

None
