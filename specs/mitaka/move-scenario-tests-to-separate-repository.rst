..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Move scenario tests to a separate repository
============================================

https://blueprints.launchpad.net/sahara/+spec/move-scenario-tests-to-separate-repo

Scenario tests represent an independent framework for testing Sahara. Moving
scenario tests to a separate repository will make testing Sahara stable
releases more comfortable. The migration should be done in a way that there
will be no need to pre-configure the CI or any other external tool, just
install the scenario framework and start the tests.

Problem description
===================

There are 2 issues to be solved:
1. It is difficult to package tests independently from Sahara.
2. It is impossible to release those tests independently.

Proposed change
===============

First it's proposed to implement a method for the correct tests installation.
Next a new repository should be created and all Sahara scenario tests should
be moved to that repository. After that all sahara-ci jobs should be migrated
to the new mechanism of running scenario tests.

The scenario framework itself should be improved to be able to correctly skip
tests for the features that are not available in earlier Sahara releases.
Testing stable/kilo and stable/liberty branches may require some updates to
appropriate scenarios. The CI should initially start running scenario tests for
stable/kilo and stable/liberty and keep testing 2 latest stable releases.

Having a separate repository allows using these tests as a plugin for Sahara
(like client tests for tempest, without releases). Perhaps, scenario tests
will be installed in a virtualenv.

The scenario framework and tests themselves may be packaged and released.
All requirements should then be installed to the current operating system.
This however implies that scenario framework dependencies should be packaged
and installed as well. Many of these  requirements are not packaged neither
in MOS nor in RDO.

There are 2 ways to solve this:
1. Manual update of requirements to keep them compatible with 2 previous
stable releases
2. Move all changing dependencies from requirements to test-requirements

Yaml-files defining the scenarios are a part of the test framework and they
should be moved to the new repository with scenario tests.

Next step is to add support of default templates for every plugin in each
stable release with correct flavors, node processes. For example, list of
plugins for the 2 previous stable releases:

* kilo: vanilla 2.6.0, cdh 5.3.0, mapr 4.0.2, spark 1.0.0, ambari 2.2/2.3
* liberty: vanilla 2.7.1, cdh 5.4.0, mapr 5.0.0, spark 1.3.1, ambari 2.2/2.3

Each default template for should be implemented as yaml file which
will be run on CI.

Alternatives
------------

Alternative would be to package the tests for easier installation and usage.
For example, command for running tests for vanilla 2.7.1 plugin will be:

.. sourcecode:: console

    $ sahara_tests -p vanilla -v 2.7.1
..

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

Deployers should switch CI to use the new repository with scenario tests
instead of current Sahara repository.

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

This will require following changes:

* Implement installation mechanism for tests.
* Move tests to separate repository (with scenario_unit tests).
* Update sahara-ci jobs for new scenario tests repository.
* Add default templates for each plugin and release.
* Update scenario framework for correct testing of previous releases.
* Add new jobs on sahara-ci for testing Sahara on previous releases.
* Add ability for tests packaging.

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

Need to add documentation with description "How to run Sahara scenario tests".

References
==========

None
